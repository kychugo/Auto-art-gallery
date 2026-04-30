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

## 🔍 Getting Indexed by Google

The gallery is already configured for search-engine discoverability:

- **`robots.txt`** — tells crawlers they are allowed to index all pages.
- **`sitemap.xml`** — lists the gallery URL; the `lastmod` date is refreshed automatically every time a new artwork is generated.
- **Meta tags** — `<title>`, `<meta name="description">`, Open Graph, Twitter Card, and JSON-LD structured data are all present in `index.html`.
- **Search engine ping** — after each artwork run, the script notifies Bing's ping endpoint so re-indexing happens faster.

### One-time: Submit to Google Search Console

The fastest way to appear in Google results is to register the site in [Google Search Console](https://search.google.com/search-console/about):

1. Go to <https://search.google.com/search-console/about> and click **Start now**.
2. Under **URL prefix**, enter your gallery URL (e.g. `https://kychugo.github.io/Auto-art-gallery/`) and click **Continue**.
3. Use the **HTML tag** verification method. Copy the `<meta>` tag that Google gives you (looks like `<meta name="google-site-verification" content="XXXXXXX" />`).
4. Paste it inside the `<head>` section of `index.html`, just below the `<meta charset>` line.
5. Commit and push, let GitHub Pages redeploy, then click **Verify** in Search Console.
6. After verification, go to **Sitemaps** (left sidebar) and submit:
   ```
   https://kychugo.github.io/Auto-art-gallery/sitemap.xml
   ```

Google will crawl and index the site within a few days. New artworks will be re-crawled automatically because the sitemap `lastmod` is updated on every run.

---

## 📄 License

MIT — do whatever you like with it.
