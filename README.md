# 🎨 Auto Art Gallery

> An autonomous AI-powered art gallery that generates a brand-new artwork every **2 hours** and publishes it to a beautiful GitHub Pages website — automatically.

[![Generate Artwork](https://github.com/kychugo/Auto-art-gallery/actions/workflows/generate.yml/badge.svg)](https://github.com/kychugo/Auto-art-gallery/actions/workflows/generate.yml)
[![Deploy to GitHub Pages](https://github.com/kychugo/Auto-art-gallery/actions/workflows/deploy.yml/badge.svg)](https://github.com/kychugo/Auto-art-gallery/actions/workflows/deploy.yml)

---

## ✨ Features

| Feature | Detail |
|---|---|
| ⏰ Auto-generation | New artwork every **2 hours** via GitHub Actions |
| 🎲 Always unique | Random topic, art style, mood, lighting & seed every time |
| 🤖 Multi-model AI | Randomly picks from **6 text models** + **8 image models** with auto-fallback |
| 🌍 Diverse topics | 30+ topics: AI, Hong Kong, global news, climate, health, culture … |
| 🖼️ Beautiful gallery | Responsive masonry grid, lightbox, search, filter, download |
| 🔁 Manual trigger | Run generation on-demand from the Actions tab |

---

## 🚀 Setup Guide

Follow these steps **once** after forking / cloning the repository.

---

### Step 1 — Add the API Key Secret

The generator uses [Pollinations AI](https://pollinations.ai). You need to store the API key as a **GitHub Actions Repository Secret** so it is never exposed in source code.

1. Go to your repository on GitHub.
2. Click **Settings** (top menu).
3. In the left sidebar, expand **Secrets and variables** → click **Actions**.
4. Under **Repository secrets**, click **New repository secret**.
5. Fill in the form:

   | Field | Value |
   |---|---|
   | **Name** | `POLLINATIONS_API_KEY` |
   | **Secret** | `sk_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx` ← paste your key here |

6. Click **Add secret**.

> ⚠️ **Never** commit the API key directly into any file. The workflow reads it from the secret automatically via `${{ secrets.POLLINATIONS_API_KEY }}`.

---

### Step 2 — Enable GitHub Pages

1. Go to **Settings** → **Pages** (left sidebar).
2. Under **Source**, select **GitHub Actions**.
3. Click **Save**.

That's it. The Pages deploy workflow will trigger automatically whenever new artwork is committed to `main`.

---

### Step 3 — Trigger Your First Artwork (Manual)

You don't have to wait 2 hours for the first image!

1. Go to the **Actions** tab in your repository.
2. Click **Generate Artwork** in the left sidebar.
3. Click the **Run workflow** button (top-right of the runs list).
4. Select branch `main` → click **Run workflow**.

The workflow will:
- Pick a random topic, art style, mood, lighting, and color palette
- Call a random text model to craft a rich image prompt
- Call a random image model to generate the artwork
- Commit the image + update `gallery.json`
- Automatically trigger the Pages deploy

Your gallery will be live at:
```
https://<your-github-username>.github.io/Auto-art-gallery/
```

---

## 🔄 How It Works

```
Every 2 hours (or on-demand)
        │
        ▼
┌───────────────────────────┐
│   Generate Artwork CI     │
│                           │
│  1. Pick random topic     │
│  2. Pick random text      │
│     model (6 options)     │
│  3. Generate image prompt │
│     via Pollinations AI   │
│  4. Pick random image     │
│     model (8 options)     │
│  5. Generate image        │
│  6. Save to images/       │
│  7. Update gallery.json   │
│  8. Commit & push → main  │
└───────────┬───────────────┘
            │ push to main triggers
            ▼
┌───────────────────────────┐
│   Deploy to Pages CI      │
│                           │
│  Deploys entire repo to   │
│  GitHub Pages             │
└───────────────────────────┘
```

---

## 🤖 AI Models Used

### Text Models (prompt generation)
One is chosen randomly each run; the next is tried automatically if one fails.

| Model ID | Provider |
|---|---|
| `gemini-fast` | Google Gemini 2.5 Flash Lite |
| `gemini-search` | Google Gemini 2.5 Flash Lite (search) |
| `openai` | OpenAI GPT-5 Mini |
| `perplexity-fast` | Perplexity Sonar |
| `minimax` | MiniMax M2.5 |
| `deepseek` | DeepSeek V3.2 |

### Image Models (artwork generation)
One is chosen randomly each run; the next is tried automatically if one fails.

| Model ID | Provider |
|---|---|
| `flux` | Flux Schnell |
| `zimage` | Z-Image Turbo |
| `flux-2-dev` | FLUX.2 Dev (api.airforce) |
| `imagen-4` | Imagen 4 (api.airforce) |
| `grok-imagine` | Grok Imagine (api.airforce) |
| `klein` | FLUX.2 Klein 4B |
| `klein-large` | FLUX.2 Klein 9B |
| `gptimage` | GPT Image 1 Mini |

---

## 📁 Project Structure

```
Auto-art-gallery/
├── .github/
│   └── workflows/
│       ├── generate.yml    # Scheduled + manual artwork generation
│       └── deploy.yml      # Auto-deploys to GitHub Pages on push to main
├── images/                 # Generated artwork images (auto-populated)
├── index.html              # Gallery web page (GitHub Pages)
├── gallery.json            # Artwork metadata (auto-updated)
├── generate_artwork.py     # Core generation script
├── APIDOCS.md              # Pollinations AI API reference
└── README.md               # This file
```

---

## 🎨 Customisation

### Add more topics
Edit the `TOPICS` list in `generate_artwork.py`:
```python
TOPICS = [
    "your custom topic here",
    ...
]
```

### Change the generation schedule
Edit `.github/workflows/generate.yml`:
```yaml
schedule:
  - cron: '0 */2 * * *'   # currently every 2 hours — adjust to taste
```
Use [crontab.guru](https://crontab.guru) to build a cron expression.

### Change image dimensions
In `generate_artwork.py`, find the `generate_image` function and update:
```python
f"?model={model}&width=1024&height=1024&seed={seed}&nologo=true"
#                 ^^^^^^^^^^   ^^^^^^^^^^^
```

---

## 🔑 Environment Variables Reference

| Variable | Where to set | Description |
|---|---|---|
| `POLLINATIONS_API_KEY` | GitHub → Settings → Secrets and variables → Actions → Repository secrets | Your Pollinations AI secret key |

> **Only repository secrets are used.** No `.env` files are needed — everything runs inside GitHub Actions.

---

## 🔐 Admin Panel

The gallery includes a built-in admin panel that lets you **permanently delete artworks** directly from the browser without touching GitHub manually.

### How to access

1. Open your gallery website.
2. Scroll to the very bottom of the page.
3. Click the small, subtle **`admin`** link in the footer (bottom-right area).
4. An **Admin Login** modal will appear asking for two things:
   - **Password** — the `ADMIN_PASSWORD` value configured in `index.html`.
   - **GitHub Personal Access Token** — a token with write access to repository contents (see below).

### Getting a GitHub Personal Access Token

The admin panel uses the GitHub REST API to delete image files and update `gallery.json`.  
It only needs permission to **read and write repository contents** — nothing else.

#### Step-by-step: Create a fine-grained token

1. Go to **GitHub → Settings → Developer settings → Personal access tokens → Fine-grained tokens**.
2. Click **Generate new token**.
3. Fill in the form:

   | Field | Recommended value |
   |---|---|
   | **Token name** | `Auto-art-gallery admin` (or any name you like) |
   | **Expiration** | 30 or 90 days (set an expiry — GitHub recommends it) |
   | **Resource owner** | Your GitHub account |
   | **Repository access** | **Only select repositories** → pick `Auto-art-gallery` |

4. Under **Permissions → Repository permissions**, set **only** the following:

   | Permission | Access level |
   |---|---|
   | **Contents** | **Read and write** |
   | Metadata | Read-only *(always required — cannot be changed)* |

   > ⚠️ **All other permissions** (Actions, Deployments, Pages, Workflows, etc.) can be left at **No access**. The admin panel does not use them.

5. Click **Generate token** and **copy the token immediately** — GitHub only shows it once.

### Logging in

Paste the token into the **GitHub Personal Access Token** field in the admin login modal, enter the password, and click **Login**.  
A red **🔐 Admin Mode** badge will appear in the top-right corner confirming you are logged in.

### Deleting an artwork

Once in admin mode, each artwork card shows a red **Delete** button.  
Clicking it will:
1. Ask for confirmation.
2. Delete the image file from the repository via the GitHub API.
3. Remove the artwork entry from `gallery.json` and commit the update.
4. Remove the card from the gallery view immediately.

> ℹ️ Deletion is **permanent** and directly modifies the repository. The artwork cannot be recovered unless you restore it via `git`.

### Exiting admin mode

Click the **admin** footer link again (or reload the page). Your GitHub token is held in memory only — it is never stored in `localStorage` or cookies.

---

## 📄 License

MIT — do whatever you like with it.
