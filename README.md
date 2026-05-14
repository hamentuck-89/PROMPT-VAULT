# PromptVault

A personal prompt management web app that syncs to Google Sheets. Organised by pages and headings, with image support, IMP flagging, and batch paste.

---

## Live URL
```
https://hamentuck-89.github.io/PROMPT-VAULT/
```

## Password
```
S@dsatan89
```

---

## Tech Stack
- Single HTML file (vanilla JS, no frameworks)
- Backend: Google Apps Script (Web App)
- Database: Google Sheets
- Images: Google Drive
- Hosting: GitHub Pages

---

## Important URLs

| What | URL |
|------|-----|
| Live App | https://hamentuck-89.github.io/PROMPT-VAULT/ |
| GitHub Repo | https://github.com/hamentuck-89/PROMPT-VAULT |
| Apps Script | https://script.google.com/macros/s/AKfycbxzcKIsfeQwuCj7HPtdn5BGq1oguUr6r0KuKf_C38FJp6oYHdOEKt5BI3dvQtTGrQzFfA/exec |
| Google Sheet ID | 1zOR8ovqNP28Ypa765OUXheBxuev8co-bvZE6wJw9WlA |

---

## How It Works
- All prompts are stored in **Google Sheets (Sheet1)**
- Images are uploaded to a **Google Drive folder** called `PromptVaultImages`
- The frontend communicates with the backend via **JSONP** (to bypass CORS)
- Pages and heading order are saved in **localStorage**
- A local cache (`pv_cache`) is kept for offline fallback

---

## How to Update the App

### Update the HTML (frontend changes):
1. Edit `index.html` locally
2. Go to github.com/hamentuck-89/PROMPT-VAULT
3. Click **Add file → Upload files** → upload new `index.html`
4. Commit changes — GitHub Pages auto-deploys in ~1 min

### Update the Apps Script (backend changes):
1. Go to script.google.com
2. Open the Prompt Vault project
3. Edit `Code.gs`
4. Save → **Deploy → Manage deployments**
5. Edit active deployment → set Version to **New version** → Deploy
6. Make sure **Who has access = Anyone**

---

## Apps Script Setup (if rebuilding from scratch)
1. Go to script.google.com → New project
2. Paste the `Code.gs` content
3. Deploy as Web App:
   - Execute as: **Me**
   - Who has access: **Anyone**
4. Copy the deployment URL and update `const API = '...'` in `index.html`

---

## Bugs Fixed (for reference)
1. `THEMES` const was defined after `initApp()` call — caused crash on load
2. No `doPost` function in Apps Script — write operations were silently failing
3. `fetch()` blocked by CORS — switched to JSONP via dynamic script tags
4. Apps Script deployment was set to "Only myself" — changed to "Anyone"
5. App was being opened as a local file — hosted on GitHub Pages to fix fetch blocking

---

## Features
- Multi-page tabs
- Prompts grouped by headings
- Batch paste with auto-parse (detects IMP flags via `[x]` or `***`)
- Per-prompt image upload (stored in Google Drive)
- Copy, Edit, Delete per prompt
- 3 themes: Warm Dark, Light, Cool Dark
- Custom background image
- Sync status indicator
- Password gate
- Offline fallback via localStorage cache
