# Wake Up Supabase

[English](./README.md) | **简体中文**

一个极简的 GitHub Actions 小工具，用于防止免费版 Supabase 项目因不活跃而被自动暂停。

Supabase 会在大约一周不活跃后暂停免费版项目。
如果项目持续暂停 90 天，将被永久删除。

Supabase 的 7 天不活跃计时器只统计**数据库活动** —— 而非 API 请求、
访问 dashboard 或认证健康检查。因此本 workflow 每天通过 PostgREST 数据接口
（`POST /rest/v1/wake_up_supabase`）向每个项目里的一张专用表**写入一行**。
`INSERT` 是一次无歧义的数据库事务 —— 不会被任何缓存代答，正是计时器统计的那种活动。

> 之前这个 workflow 用的是每天一次 `SELECT`。实测发现某些项目仅靠读取并不能
> 稳定重置计时器，因此改为 `INSERT`。一天一行毫无负担 —— 每个项目一年约 365 行。

---

## 🚀 特性

- 可同时保活任意数量的 Supabase 项目
- 即使项目分布在多个 Supabase 账号下也能工作
- 密钥只存放在 GitHub Secrets 中（绝不写入仓库）
- 全自动 —— 每天运行一次
- 完全免费

---

## 🛡 安全性

- 只使用 anon key（绝不使用 service_role key）
- 密钥安全地存放在 GitHub Actions Secrets 中
- workflow 不会打印任何密钥
- 仓库本身不含任何敏感信息

---

## 🧩 配置步骤

### 1. Fork 本仓库
点击 GitHub 页面右上角的 Fork 按钮。

---

### 2. 在每个 Supabase 项目里建好心跳表

对你要保活的**每一个** Supabase 项目，打开
**Supabase Dashboard → SQL Editor**，把下面这段 SQL 跑一次：

```sql
create table if not exists public.wake_up_supabase (
  id bigserial primary key,
  value text not null
);

alter table public.wake_up_supabase enable row level security;

create policy "anon insert wake_up_supabase"
  on public.wake_up_supabase
  for insert
  to anon
  with check (true);
```

这段 SQL 会创建一张很小的表，并授予 `anon` 角色写入权限。**没有**授予 `select`
策略，因此 anon 无法读回这些行 —— 这张表对公开 API 来说是只写的。

---

### 3. 把你的 Supabase 项目添加到 GitHub Secret

进入：

Settings → Secrets and variables → Actions → New repository secret

创建一个名为如下的 secret：

    SUPABASE_PROJECTS_JSON

按以下格式粘贴你的项目：

    [
      {
        "url": "https://your-project-1.supabase.co",
        "anon_key": "YOUR_PROJECT_1_ANON_PUBLIC_KEY"
      },
      {
        "url": "https://your-project-2.supabase.co",
        "anon_key": "YOUR_PROJECT_2_ANON_PUBLIC_KEY"
      }
    ]

可以添加任意数量的项目。

💡 只使用 anon key —— 绝不要用 service_role key。
anon key 可在以下位置找到：Supabase Dashboard → Project Settings → API

#### `table` —— 可选

如果不填 `table`，workflow 会写入 `wake_up_supabase`（即步骤 2 里建的表）。
只有当你想指向自定义表时才需要填 `table`：

    {
      "url": "https://your-project.supabase.co",
      "anon_key": "YOUR_PROJECT_ANON_PUBLIC_KEY",
      "table": "your_custom_table"
    }

自定义表必须满足：

- 已存在并通过 API 暴露，
- 有一条允许 `anon` 角色 `INSERT` 的 RLS 策略，
- 能接收 JSON body `{"value": "<iso-timestamp>"}`（即有一个 `value text` 列，
  或者能容纳字符串的列）。

---

### 4. 启用 GitHub Actions

进入 Actions 标签页 → 如有需要，启用 workflows。

---

### 5.（可选）手动运行一次

Actions 标签页 → 选择该 workflow → Run workflow。

---

## ⏱ 工作原理

本 action 每天 06:00 UTC 运行一次。

对于 SUPABASE_PROJECTS_JSON secret 中的每个项目，它会：

1. 构造数据接口 URL
   `https://your-project.supabase.co/rest/v1/wake_up_supabase`

2. 在 `apikey` 和 `Authorization: Bearer` 两个请求头中携带 anon key
3. `POST` 一段 JSON `{"value": "<当前 UTC 时间>"}`，并校验返回 HTTP 201

这条 `INSERT` 会真正在 Postgres 上执行 —— 而这正是 Supabase 所统计的活动，
因此不活跃计时器会被重置，项目不会被暂停。

某个项目失败不会中断整次运行：缺字段、curl 出错或返回非 2xx 状态都会被记录，
随后循环继续处理下一个项目。

> ⚠️ 本 workflow 用于**预防**暂停 —— 它无法唤醒一个已被暂停的项目。
> 如果项目已经被暂停，请先在 Supabase dashboard 中手动 Restore 恢复一次，
> 之后每天的写入就能持续让它保持活跃。

---

## 🧪 日志输出示例

    📦 Found 2 Supabase project(s).
    🕒 Heartbeat value: 2026-05-26T06:00:00Z
    🌐 Inserting heartbeat into: https://abc123.supabase.co/rest/v1/wake_up_supabase
    ✅ https://abc123.supabase.co responded with HTTP 201 — row inserted, timer reset.
    🌐 Inserting heartbeat into: https://def456.supabase.co/rest/v1/wake_up_supabase
    ✅ https://def456.supabase.co responded with HTTP 201 — row inserted, timer reset.

---

## ❓ 常见问题

**为什么是写一行，而不是 `SELECT`？**
Supabase 的不活跃计时器只统计**数据库活动**。`SELECT` 可能被缓存代答，或被
RLS 拒绝（401/403 并不算活动）。实测发现仅靠每天 `SELECT` 部分项目仍然会被
暂停。`INSERT` 是一次无歧义的写事务 —— Postgres 必须真实执行，Supabase 会
明确地把它算作活动。

**表会不会一直变大？**
一天一行 = 每个项目每年约 365 行，多年下来也完全无负担。要清理的话，随时在
SQL Editor 里跑 `truncate public.wake_up_supabase` 即可。

**为什么要查表，而不是 ping `/auth/v1/health`？**
Supabase 的不活跃计时器只统计**数据库活动**。认证健康检查接口完全不会触及
Postgres，所以 ping 它不会重置计时器。

**这能用于多个 Supabase 账号吗？**
可以 —— 只需把你所有项目都加入 secret 即可。

**这个仓库可以公开吗？**
可以 —— 所有敏感数据都保存在 GitHub Secrets 中。

**这会违反 Supabase 的规则吗？**
不会。你只是在按设计意图正常使用公开的 anon 接口。

---

## ❤️ 贡献

本项目刻意保持精简。
欢迎提交 issue 或 PR 来添加诸如以下的功能：

- Slack / Discord 提醒
- 错误通知
- JSON schema 校验
- 多区域健康检查
- 可选的日志看板

---

祝使用愉快！
@wilhelmsendk
