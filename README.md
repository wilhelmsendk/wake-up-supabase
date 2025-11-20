# Wake Up Supabase

A tiny GitHub Actions helper that prevents free Supabase projects from being automatically paused due to inactivity.

Supabase pauses inactive free-tier projects after about a week.  
If a project stays paused for 90 days, it is permanently deleted.

This workflow performs a lightweight daily ping to each project’s  
`/auth/v1/health` endpoint using anon keys (safe to use, public by design).  
This activity prevents auto-pausing and keeps your projects alive.

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

### 2. Add your Supabase projects to a GitHub Secret

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

---

### 3. Enable GitHub Actions

Go to the Actions tab → enable workflows if required.

---

### 4. (Optional) Run manually once

Actions tab → select the workflow → Run workflow.

---

## ⏱ How It Works

This action runs every day at 06:00 UTC.

For each project in your SUPABASE_PROJECTS_JSON secret, it:

1. Calls  
   https://your-project.supabase.co/auth/v1/health

2. Sends the anon key in an `apikey` header  
3. Checks whether the project responded correctly  

Supabase counts this as legitimate activity and will not pause the project.

---

## 🧪 Example Log Output

    Found 3 projects.
    Pinging https://abc123.supabase.co/auth/v1/health ...
    ✔ abc123.supabase.co is awake (HTTP 200)
    Pinging https://def456.supabase.co/auth/v1/health ...
    ✔ def456.supabase.co is awake (HTTP 200)

---

## ❓ FAQ

**Does this work for multiple Supabase accounts?**  
Yes — simply add all your project URLs and anon keys to the secret.

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
