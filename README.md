# 🚀 Wake Up Supabase

> Keep your free Supabase projects alive automatically using GitHub Actions.

Supabase automatically pauses inactive **Free Plan** projects after approximately **7 days**.  
If a project remains paused for **90 days**, it is permanently deleted.

This repository performs a lightweight **daily database request** against each configured project to keep the inactivity timer reset.

---

## ✨ Features

- ✅ Keeps one or many Supabase projects awake
- ✅ Supports projects from multiple Supabase accounts
- ✅ Pure Python (no dependencies)
- ✅ Runs for free using GitHub Actions
- ✅ Secrets stay inside GitHub Actions
- ✅ Zero configuration after setup

---

## 🔒 Security

- Uses **anon/publishable keys only**
- ❌ Never use your `service_role` key
- Secrets are stored in **GitHub Actions Secrets**
- Keys are never printed in logs

---

# 🛠 Installation

## 1. Fork this repository

Click **Fork** in the top-right corner of GitHub.

---

## 2. Create the GitHub Secret

Go to:

`Settings → Secrets and variables → Actions`

Create a new repository secret named:

```text
SUPABASE_PROJECTS_JSON
```

Paste your projects using this format:

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
```

You can add as many projects as you like.

> 💡 Only use **anon** or **publishable** keys.
> Never use the **service_role** key.

You can find your API keys here:

`Supabase Dashboard → Project Settings → API`

---

## 3. Enable GitHub Actions

Open the **Actions** tab and enable workflows if GitHub asks you.

---

## 4. (Optional) Run once manually

Open:

**Actions → Wake Up Supabase → Run workflow**

---

# ⚙️ How it works

Every day at **06:00 UTC** the workflow:

1. Reads `SUPABASE_PROJECTS_JSON`
2. Loops through every project
3. Sends a request to:

```text
https://your-project.supabase.co/rest/v1/
```

using your anon key.

A successful response keeps the database active and prevents auto-pausing.

---

## 🧪 Example output

```text
📦 Found 2 Supabase project(s).

🌐 Pinging Database (PostgREST) via:
https://abc123.supabase.co/rest/v1/

✅ HTTP 200 — Database is awake.

🌐 Pinging Database (PostgREST) via:
https://def456.supabase.co/rest/v1/

✅ HTTP 200 — Database is awake.
```

---

# ❓ FAQ

### Does it support multiple Supabase accounts?

Yes.

Simply add every project to `SUPABASE_PROJECTS_JSON`.

---

### Can this repository be public?

Yes.

All secrets remain inside GitHub Secrets.

---

### Why not use `/auth/v1/health`?

Supabase changed how inactivity is detected.

Calling `/rest/v1/` performs a real PostgREST request, which resets the inactivity timer.

---

# ❤️ Contributing

Pull requests are welcome.

Ideas:

- Slack notifications
- Discord notifications
- JSON schema validation
- Health dashboard
- Better logging

---

## ⭐ If this project helped you

Consider giving the repository a ⭐ on GitHub.

Happy building!

— @wilhelmsendk
