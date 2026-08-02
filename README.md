# ReconEasy

**Phone-first expense reconciliation for WildTrust — Credit Card & Petty Cash.**

Photograph a slip, let AI suggest the merchant, amount, and category, tag it with a vote number and project, and sync straight to Google Sheets and Drive. No app store, no install, no server.

*Built in-house by WILDOCEANS IT.*

---

## What it does

- 📷 **Capture** — photograph a slip or choose one from your gallery
- ✨ **AI-assisted reading** — merchant, amount, date, and category are suggested automatically, written in WildOceans' own recon phrasing style
- 🔎 **Vote number search** — searches a live, shared Google Sheet of budget/vote codes
- ✅ **Review before saving** — nothing is ever auto-committed; every AI suggestion can be edited or ignored
- ☁️ **Sync to your own Google Drive** — a Sheet and a receipts folder are created automatically each month, in *your* Drive, under *your* Google account
- 📱 **Installable** — add it to your phone's home screen and it behaves like a native app, complete with its own icon
- 📖 **Built-in guide & policy reference** — the Credit Card and Petty Cash use agreements, distilled into a quick in-app FAQ and rules screen

---

## How it's built

This is a single static HTML file — no build step, no framework, no backend server. It's paired with one small serverless proxy that keeps the AI API key private.

```
┌─────────────────────┐        ┌──────────────────────┐        ┌────────────────────┐
│   index.html         │──────▶│  Cloudflare Worker    │──────▶│   Anthropic API     │
│  (GitHub Pages)       │        │  (holds the API key)  │        │  (reads the slip)   │
└─────────────────────┘        └──────────────────────┘        └────────────────────┘
          │
          ▼
┌─────────────────────────────────────┐
│  Signed-in user's own Google account │
│  → Drive (receipts) + Sheets (recon) │
└─────────────────────────────────────┘
```

| Piece | Where it lives | Purpose |
|---|---|---|
| `index.html` | This repo, served via GitHub Pages | The entire app — UI, logic, vote database fallback |
| Worker (`index.js`) | Separate repo: `reconeasy-worker`, deployed on Cloudflare | Proxies AI slip-reading requests so the API key never touches the client |
| Vote database | A shared Google Sheet | Live source of vote/budget codes, read by every signed-in user |
| Recon entries & receipts | Each user's own Google Drive | Created automatically on first sync — nothing is centralised |

---

## Live app

**https://nxrth98.github.io/reconeasy**

Add it to your phone's home screen (Share → Add to Home Screen on iOS, or the browser menu on Android) for the closest thing to a native app experience.

---

## Access

This app is currently in Google's **OAuth Testing mode**, capped at 100 users. To get access:

1. Ask WILDOCEANS IT to add your Gmail as a test user in the Google Cloud project
2. Ask to be added as a **Viewer** on the shared vote database Sheet — without this, the app silently falls back to an older, hardcoded snapshot of vote codes instead of the live list
3. Open the link above and sign in — you'll see an "unverified app" warning from Google the first time, which is expected while in Testing mode. Click **Advanced → Go to ReconEasy (unsafe)** to continue

---

## Configuration

These values are already set inside `index.html` — listed here for reference if you're maintaining or forking this:

| Setting | Value |
|---|---|
| Google OAuth Client ID | `419380799116-k9dnh5p4i66r1kulqdjirj6uh3ojenrv.apps.googleusercontent.com` |
| OAuth redirect URI | `https://nxrth98.github.io/reconeasy` |
| Cloudflare Worker URL | `https://reconeasy-worker.nxrth98.workers.dev` |
| Vote database Sheet ID | `1GrLCv_s_WTvjksuZI50NANHtXZnnwcdxVbYMWL-qurc` |

The Anthropic API key is **not** in this repo — it's stored as an encrypted secret directly in the Cloudflare Worker's environment variables, kept deliberately separate from the app's source.

---

## Deploying your own copy

1. Fork or copy this repo, enable **GitHub Pages** on it (Settings → Pages → Deploy from branch)
2. Create a Google Cloud project, enable the **Google Drive API** and **Google Sheets API**, and create an OAuth Client ID (Web application type) with your Pages URL as both the authorised origin and redirect URI
3. Deploy the companion Cloudflare Worker (see `reconeasy-worker` repo) with your own Anthropic API key stored as an encrypted secret
4. Update the config values in `index.html` (see table above) to match your own IDs and URLs
5. Create your own vote database Google Sheet and update `VOTE_SHEET_ID` accordingly

---

## Data model, in short

- Entries and photos are stored **locally on the device** (browser storage) until manually synced — nothing is sent anywhere just by capturing a slip
- Tapping **Sync** writes entries to a Google Sheet in the signed-in user's own Drive
- Tapping **Upload** pushes receipt photos to that same user's Drive
- There is currently **no shared team view** — each person's synced data lives in their own Drive, not a central location

For the full technical breakdown — capture flow, folder structure, known fragile points, and the multi-user model — see the project documentation.

---

## Status

Actively used by a small group of WildTrust testers. Not mandated policy — an optional tool for anyone who finds it useful.
