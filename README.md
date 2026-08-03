# Wake Up Supabase

A tiny GitHub Actions helper that prevents free Supabase projects from being automatically paused due to inactivity.

Supabase pauses inactive free-tier projects after about a week.  
If a project stays paused for 90 days, it is permanently deleted.

This workflow performs a daily ping directly against each project's **PostgREST engine** (`/rest/v1/`) using your public `anon` key. This forces a database query execution, which resets Supabase's internal inactivity timer and keeps your projects alive.

---

## 🚀 Features

- **Database-level pings**: Hits `/rest/v1/` to ensure Supabase registers real database activity
- **Zero extra dependencies**: Built with pure Python in GitHub Actions for fast, clean execution
- **Multi-project support**: Keeps any number of Supabase projects awake (even across different accounts)
- **Secure**: Keys are stored safely in GitHub Actions Secrets (never in public code or logs)
- **Completely free**

---

## 🛡 Security

- Uses `anon` keys only (never use your `service_role` key)
- Secrets stored securely in GitHub Actions Secrets
- Workflow never prints or reveals keys in execution logs
- Repo contains zero sensitive information

---

## 🧩 Setup

### 1. Fork this repository  
Click the **Fork** button at the top right of GitHub.

---

### 2. Add your Supabase projects to a GitHub Secret

Go to:  
**Settings → Secrets and variables → Actions → New repository secret**

Create a secret named:  
`SUPABASE_PROJECTS_JSON`

Paste your projects in this JSON format (you can use either legacy `anon` keys or new `sb_publishable_` keys):

```json
[
  {
    "name": "savinghero",
    "url": "https://your-project-1.supabase.co",
    "anon_key": "YOUR_PROJECT_1_ANON_OR_PUBLISHABLE_KEY"
  },
  {
    "name": "my-test-app",
    "url": "https://your-project-2.supabase.co",
    "anon_key": "YOUR_PROJECT_2_ANON_OR_PUBLISHABLE_KEY"
  }
]
---

### 3. Enable GitHub Actions

Go to the **Actions** tab → enable workflows if required.

---

### 4. (Optional) Run manually once

Actions tab → select **Wake Up Supabase** → click **Run workflow**.

---

## ⏱ How It Works

This action runs automatically every day at 06:00 UTC.

For each project in your `SUPABASE_PROJECTS_JSON` secret, it:

1. Calls  
   `https://your-project.supabase.co/rest/v1/`

2. Sends the anon key in both `apikey` and `Authorization: Bearer <anon_key>` headers.
3. PostgREST queries the database schema, which resets Supabase's auto-pause timer and keeps the project active.

---

## 🧪 Example Log Output

```text
📦 Found 2 Supabase project(s).

🌐 Pinging Database (PostgREST) via: https://abc123.supabase.co/rest/v1/
✅ https://abc123.supabase.co responded with HTTP 200 — database is awake!

🌐 Pinging Database (PostgREST) via: https://def456.supabase.co/rest/v1/
✅ https://def456.supabase.co responded with HTTP 200 — database is awake!
```

---

## ❓ FAQ

**Does this work for multiple Supabase accounts?**  
Yes — simply add all your project URLs and anon keys to the secret.

**Can this repo be public?**  
Yes — all sensitive data stays inside GitHub Secrets.

**Why was this updated from `/auth/v1/health`?**  
Supabase updated their inactivity detection so simple auth health checks no longer reset the auto-pause timer. Hitting `/rest/v1/` forces a real PostgREST query against the database engine.

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
