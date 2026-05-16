# PromptVault 🗂️

A personal prompt management web app that syncs to Google Sheets. Organised by pages and headings, with image support, IMP flagging, and batch paste.

---

## 🔗 Important URLs

| What | URL |
|------|-----|
| Live App | https://hamentuck-89.github.io/PROMPT-VAULT/ |
| GitHub Repo | https://github.com/hamentuck-89/PROMPT-VAULT |
| Apps Script | https://script.google.com/macros/s/AKfycbxzcKIsfeQwuCj7HPtdn5BGq1oguUr6r0KuKf_C38FJp6oYHdOEKt5BI3dvQtTGrQzFfA/exec |
| Google Sheet ID | `1zOR8ovqNP28Ypa765OUXheBxuev8co-bvZE6wJw9WlA` |

---

## 🛠️ Tech Stack

- **Frontend:** Single HTML file (vanilla JS, no frameworks)
- **Backend:** Google Apps Script (Web App)
- **Database:** Google Sheets (Sheet1)
- **Images:** Google Drive (`PromptVaultImages` folder)
- **Hosting:** GitHub Pages

---

## ✨ Features

- Multi-page tabs
- Prompts grouped by headings
- Batch paste with auto-parse (detects IMP flags via `[x]` or `***`)
- Per-prompt image upload (stored in Google Drive)
- Copy, Edit, Delete per prompt
- 3 themes: Warm Dark, Original Light, Cool Dark
- Custom background image
- Sync status indicator
- Password gate
- Offline fallback via localStorage cache

---

## ⚙️ How It Works

- All prompts are stored in **Google Sheets** (Sheet1)
- Images are uploaded to a **Google Drive** folder called `PromptVaultImages`
- The frontend communicates with the backend via **JSONP** (for regular operations) and **fetch POST** (for image uploads)
- Pages and heading order are saved in `localStorage`
- A local cache (`pv_cache`) is kept for offline fallback

---

## 🚀 How to Update the App

### Frontend (HTML changes)
1. Edit `index.html` locally
2. Go to [github.com/hamentuck-89/PROMPT-VAULT](https://github.com/hamentuck-89/PROMPT-VAULT)
3. Click on `index.html` → edit pencil → or drag & drop the new file → commit
4. GitHub Pages auto-deploys in ~1 min

### Backend (Apps Script changes)
1. Go to [script.google.com](https://script.google.com) → open the Prompt Vault project
2. Edit `Code.gs`
3. Save → **Deploy → Manage Deployments**
4. Edit active deployment → set Version to **New version** → Deploy
5. Make sure **Who has access = Anyone**

---

## 🔧 Apps Script Setup (if rebuilding from scratch)

1. Go to [script.google.com](https://script.google.com) → New project
2. Paste the `Code.gs` content
3. Deploy as Web App:
   - Execute as: **Me**
   - Who has access: **Anyone**
4. Copy the deployment URL and update `const API = '...'` in `index.html`

---

## 🐛 Bugs Fixed

| # | Bug | Fix |
|---|-----|-----|
| 1 | `THEMES` const defined after `initApp()` call | Moved declaration before call — fixed crash on load |
| 2 | No `doPost` function in Apps Script | Added `doPost` — write operations were silently failing |
| 3 | `fetch()` blocked by CORS | Switched to JSONP via dynamic `<script>` tags for regular ops |
| 4 | Apps Script deployment set to "Only myself" | Changed to "Anyone" |
| 5 | App opened as local file | Hosted on GitHub Pages to fix fetch blocking |
| 6 | Image upload silently failing | See Fix #1 below ↓ |
| 7 | Image uploaded but not displaying | See Fix #2 below ↓ |

---

## 🖼️ Image Upload — Fix #1 (upload was failing silently)

### What was broken
Clicking "Add image" on a prompt did nothing — no error, just a timeout after a few seconds.

### Root cause
The `apiPost` function sent all data as **URL query parameters** (JSONP via `<script src="...">`). Base64 image data is 100–500KB, which makes a URL far beyond the browser/server limit of ~8KB. The request was silently truncated and never reached Apps Script.

On top of that, `doPost` in Apps Script was returning plain JSON with no JSONP callback wrapper, so even if the data arrived, the frontend callback would never fire.

### Fix — Frontend (`index.html`)
Added a dedicated `apiPostLarge()` function for image uploads that uses `fetch()` with the base64 data in the **POST body** instead of the URL:

```js
async function apiPostLarge(data) {
  fetch(API + '?callback=' + cbName, {
    method: 'POST',
    headers: { 'Content-Type': 'text/plain;charset=utf-8' },
    body: JSON.stringify(data), // ← base64 goes in body, not URL
  })
  .then(res => res.text())
  .then(text => { eval(text); }) // triggers the JSONP callback
}
```

`uploadImage()` now calls `apiPostLarge()` instead of `apiPost()`. All other operations (save, update, delete) still use the original JSONP method since their payloads are tiny.

### Fix — Apps Script (`Code.gs`)
Updated `doPost` to read the `callback` param from the URL and wrap the response in JSONP — same as `doGet` already does:

```js
function doPost(e) {
  var callback = e.parameter.callback; // ← from URL ?callback=pvcb_xxx
  ...
  if (callback) {
    return ContentService.createTextOutput(callback + '(' + json + ')')
      .setMimeType(ContentService.MimeType.JAVASCRIPT); // ← JSONP wrapper
  }
}
```

---

## 🖼️ Image Display — Fix #2 (image uploaded but not showing)

### What was broken
Image uploaded successfully (sync showed "synced", toast said "Image uploaded!") but the thumbnail area stayed blank — the image never rendered.

### Root cause
The URL generated by Apps Script was:
```
https://drive.google.com/uc?export=view&id=FILE_ID
```
Google blocks this URL when loaded inside an `<img>` tag on external sites due to cross-origin restrictions and cookie/login checks. It opens fine in a browser tab directly, but not when embedded.

### Fix — Apps Script (`Code.gs`)
One line changed in `uploadImage()` — switched to Google's CDN domain which allows cross-origin image embedding:

```js
// ❌ OLD — blocked in <img> tags on external sites:
var url = 'https://drive.google.com/uc?export=view&id=' + file.getId();

// ✅ NEW — Google CDN, loads cross-origin fine:
var url = 'https://lh3.googleusercontent.com/d/' + file.getId();
```

`lh3.googleusercontent.com` is Google's content delivery domain for Drive files — it doesn't redirect through a login check, so browsers can load it freely in `<img>` tags.

> **Note:** Images already uploaded with the old URL will still be broken (the old URL is saved in the Sheet). Re-upload those images to get the new working URL. All new uploads going forward will display correctly.

---

## 📁 Project Structure

```
PROMPT-VAULT/
├── index.html       # Entire frontend (HTML + CSS + JS in one file)
└── README.md        # This file
```

---

## 🔑 Password

```
S@dsatan89
```
