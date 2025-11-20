# Wake Up Supabase

A tiny GitHub Actions helper that prevents **free Supabase projects**
from being automatically paused due to inactivity.

Supabase pauses inactive free-tier projects after about a week.  
If a project stays paused for **90 days**, it is permanently deleted.

This workflow simply performs a lightweight daily ping to each project’s  
`/auth/v1/health` endpoint using **anon keys** (safe to use, public by design).  
This counts as “activity” and keeps the project alive.

---

## 🚀 Features

- Keeps **any number of Supabase projects** awake
- Works even if projects are owned by **different Supabase accounts**
- Uses **GitHub Secrets** only (no keys in repo!)
- Fully automated, runs daily
- Completely free

---

## 🛡 Security

- Only **anon keys** are used — these are safe to expose in public frontends.
- All keys are stored only in **GitHub Actions Secrets**, never committed.
- The workflow never prints keys to logs.
- Repo contains **zero** project information.

---

## 🧩 Setup

### 1. Fork this repository  
Click the **Fork** button at the top right of GitHub.

---

### 2. Add your Supabase projects to a GitHub Secret  

Go to:

**Settings → Secrets and variables → Actions → New repository secret**

Create a secret named:

SUPABASE_PROJECTS_JSON

kotlin
Kopier kode

Paste your projects in this format:

```json
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
You can find anon keys here:
Supabase Dashboard → Project Settings → API

3. Enable GitHub Actions
Go to the Actions tab → click Enable workflows if prompted.

4. (Optional) Run once manually
In the Actions tab → select the workflow → click Run workflow to test immediately.

⏱ How It Works
The action runs every day at 06:00 UTC.

It loads your SUPABASE_PROJECTS_JSON secret, parses it, and for each project:

Calls
https://your-project.supabase.co/auth/v1/health

Sends the anon key using the apikey header

Logs whether the project responded correctly

Supabase registers this as legitimate activity, preventing auto-pausing.

🧪 Example Log Output
bash
Kopier kode
Found 3 projects.
Pinging https://abc123.supabase.co/auth/v1/health ...
✔ abc123.supabase.co is awake (HTTP 200)
Pinging https://def456.supabase.co/auth/v1/health ...
✔ def456.supabase.co is awake (HTTP 200)
❓ FAQ
Does this work for multiple Supabase accounts?
Yes — just add all projects to the JSON list in the secret.

Can I make the repo public?
Yes — all sensitive data stays in your GitHub Secrets.

Does this violate any Supabase rules?
No. You're only hitting public anon endpoints.

❤️ Contribute
This project is intentionally tiny.
Feel free to open issues or PRs to add features like:

Slack / Discord notifications

Error reporting

JSON schema validation

Multi-region pinging

Enjoy :-) @wilhelmsendk
