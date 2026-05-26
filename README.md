# Wake Up Supabase

**English** | [简体中文](./README.zh-CN.md)

A tiny GitHub Actions helper that prevents free Supabase projects from being automatically paused due to inactivity.

Supabase pauses inactive free-tier projects after about a week.  
If a project stays paused for 90 days, it is permanently deleted.

Supabase's 7-day inactivity timer is tracked against **database activity** —
not API requests, dashboard visits, or auth health checks. So this workflow
**writes** one row per day into a dedicated table in each project via the
PostgREST data API (`POST /rest/v1/wake_up_supabase`). An `INSERT` is an
unambiguous DB transaction — it cannot be served from any cache, and it is
exactly the kind of activity the timer tracks.

> Previously this workflow used a daily `SELECT`. In practice, reads were not
> reliably resetting the timer for some projects, so we switched to an
> `INSERT`. A row a day is harmless — ~365 rows/year per project.

---

## 🚀 Features

- Keeps any number of Supabase projects awake  
- Works even if projects span multiple Supabase accounts  
- Stores keys only in GitHub Secrets (never in the repo)  
- Fully automated — runs once per day  
- Completely free

---

## 🛡 Security

- Uses anon keys only (never the service_role key)  
- Secrets stored securely in GitHub Actions Secrets  
- Workflow never prints keys  
- Repo contains zero sensitive information

---

## 🧩 Setup

### 1. Fork this repository  
Click the Fork button at the top right of GitHub.

---

### 2. Create the heartbeat table in each Supabase project

For **every** Supabase project you want to keep awake, open
**Supabase Dashboard → SQL Editor** and run this once:

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

This creates a tiny table and grants the `anon` role permission to insert
into it. No `select` policy is granted, so anon cannot read the rows back —
the table is write-only from the public API.

---

### 3. Add your Supabase projects to a GitHub Secret

Go to:

Settings → Secrets and variables → Actions → New repository secret

Create a secret named:

    SUPABASE_PROJECTS_JSON

Paste your projects in this format:

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

Add as many projects as you want.

💡 Only use anon keys — never the service_role key.  
You can find anon keys under: Supabase Dashboard → Project Settings → API

#### `table` — optional

If you omit `table`, the workflow writes to `wake_up_supabase` (the table
created in step 2). You only need to set `table` if you want to point at a
custom table:

    {
      "url": "https://your-project.supabase.co",
      "anon_key": "YOUR_PROJECT_ANON_PUBLIC_KEY",
      "table": "your_custom_table"
    }

A custom table must:

- exist and be exposed via the API,
- have an RLS policy granting `INSERT` to the `anon` role,
- accept a JSON body of `{"value": "<iso-timestamp>"}` (so a `value text`
  column, or anything that coerces a string).

---

### 4. Enable GitHub Actions

Go to the Actions tab → enable workflows if required.

---

### 5. (Optional) Run manually once

Actions tab → select the workflow → Run workflow.

---

## ⏱ How It Works

This action runs every day at 06:00 UTC.

For each project in your SUPABASE_PROJECTS_JSON secret, it:

1. Builds the data-API URL  
   `https://your-project.supabase.co/rest/v1/wake_up_supabase`

2. Sends the anon key in both the `apikey` and `Authorization: Bearer` headers  
3. `POST`s a JSON body `{"value": "<current UTC time>"}` and checks for HTTP 201  

That `INSERT` runs against Postgres, which is the activity Supabase tracks —
so the inactivity timer is reset and the project is not paused.

A failing project never aborts the run: a missing field, a curl error, or a
non-2xx status is logged and the loop moves on to the next project.

> ⚠️ This workflow **prevents** pausing — it cannot wake an already-paused
> project. If a project is already paused, restore it once from the Supabase
> dashboard; the daily insert then keeps it awake from there on.

---

## 🧪 Example Log Output

    📦 Found 2 Supabase project(s).
    🕒 Heartbeat value: 2026-05-26T06:00:00Z
    🌐 Inserting heartbeat into: https://abc123.supabase.co/rest/v1/wake_up_supabase
    ✅ https://abc123.supabase.co responded with HTTP 201 — row inserted, timer reset.
    🌐 Inserting heartbeat into: https://def456.supabase.co/rest/v1/wake_up_supabase
    ✅ https://def456.supabase.co responded with HTTP 201 — row inserted, timer reset.

---

## ❓ FAQ

**Why insert a row instead of running a `SELECT`?**  
Supabase's inactivity timer only counts **database activity**. A `SELECT` may
be served from a cache or denied by RLS (a 401/403 isn't activity), and in
practice some projects were still getting paused with daily SELECTs. An
`INSERT` is an unambiguous write transaction — Postgres has to actually run
it, and Supabase clearly counts that as activity.

**Won't the table grow forever?**  
One row per day = ~365 rows/year per project. The table is harmless even after
many years. If you want to clean up, run `truncate public.wake_up_supabase`
in the SQL Editor anytime.

**Why query a table at all instead of pinging `/auth/v1/health`?**  
Supabase's inactivity timer only counts **database activity**. The auth health
endpoint never touches Postgres, so pinging it does not reset the timer.

**Does this work for multiple Supabase accounts?**  
Yes — simply add all your projects to the secret.

**Can this repo be public?**  
Yes — all sensitive data stays inside GitHub Secrets.

**Does this violate Supabase rules?**  
No. You're using public anon endpoints exactly as intended.

---

## ❤️ Contribute

This project is intentionally minimal.  
Feel free to open issues or PRs to add features like:

- Slack / Discord alerts  
- Error notifications  
- JSON schema validation  
- Multi-region health checks  
- Optional logging dashboard  

---

Enjoy!  
@wilhelmsendk
