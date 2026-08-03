# Wake Up Supabase

A tiny GitHub Actions helper that prevents free Supabase projects from being automatically paused due to inactivity.

Supabase pauses inactive free-tier projects after about a week.  
If a project stays paused for 90 days, it is permanently deleted.

This workflow performs a daily ping against your projects using your public `anon` or `publishable` key. This resets Supabase's internal inactivity timer and keeps your projects alive.

---

## 🚀 Features

- **Auth & API Pings**: Hits `/auth/v1/health` to ensure Supabase registers activity
- **Zero extra dependencies**: Built with pure Python in GitHub Actions for fast, clean execution
- **Multi-project support**: Keeps any number of Supabase projects awake (even across different accounts)
- **Support for new & legacy keys**: Works with both standard `anon` JWT keys and new `sb_publishable_` keys
- **Secure**: Keys are stored safely in GitHub Actions Secrets (never in public code or logs)
- **Completely free**

---

## 🛡 Security

- Uses `anon` / `publishable` keys only (never use your `service_role` key)
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

Paste your projects in this JSON format:

```json
[
  {
    "name": "savinghero",
    "url": "[https://your-project-1.supabase.co](https://your-project-1.supabase.co)",
    "anon_key": "YOUR_PROJECT_1_ANON_OR_PUBLISHABLE_KEY"
  },
  {
    "name": "my-test-app",
    "url": "[https://your-project-2.supabase.co](https://your-project-2.supabase.co)",
    "anon_key": "YOUR_PROJECT_2_ANON_OR_PUBLISHABLE_KEY"
  }
]
Add as many projects as you want.

💡 Note: Only use anon or publishable keys — never use the service_role key.

You can find your key under: Supabase Dashboard → Project Settings → API.

3. Enable GitHub Actions
Go to the Actions tab → enable workflows if required.

4. (Optional) Run manually once
Go to Actions tab → select Wake Up Supabase → click Run workflow.

⏱ How It Works
This action runs automatically every day at 06:00 UTC.

For each project in your SUPABASE_PROJECTS_JSON secret, it:

Calls the Supabase Auth Health endpoint:

https://your-project.supabase.co/auth/v1/health

Sends the key in both apikey and Authorization: Bearer <key> headers.

The health check resets Supabase's auto-pause timer and keeps the project active.

🧪 Example Log Output
Plaintext
📦 Found 2 Supabase project(s).

🌐 Pinging Database: jobhero ([https://jkjbsytazfatxbhrwthv.supabase.co](https://jkjbsytazfatxbhrwthv.supabase.co))
✅ jobhero ([https://jkjbsytazfatxbhrwthv.supabase.co](https://jkjbsytazfatxbhrwthv.supabase.co)) responded with HTTP 200 — database is awake!

🌐 Pinging Database: savinghero ([https://wryomwolthgnobkwergv.supabase.co](https://wryomwolthgnobkwergv.supabase.co))
✅ savinghero ([https://wryomwolthgnobkwergv.supabase.co](https://wryomwolthgnobkwergv.supabase.co)) responded with HTTP 200 — database is awake!
❓ FAQ
Does this work for multiple Supabase accounts?

Yes — simply add all your project URLs and keys to the JSON secret.

Can this repo be public?

Yes — all sensitive data stays inside GitHub Secrets.

❤️ Contribute
This project is intentionally minimal.

Feel free to open issues or PRs to add features like:

Slack / Discord alerts

Error notifications

JSON schema validation

Enjoy!

@wilhelmsendk
