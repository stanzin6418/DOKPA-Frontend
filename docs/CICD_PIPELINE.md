# 🚀 CI/CD Pipeline — Dokpa Expeditions → Hostinger

> **Last Updated:** November 2026  
> **Pipeline:** GitHub Actions → FTP → Hostinger Shared Hosting  
> **Trigger:** Every push to `main` branch (or manual trigger)

---

## Pipeline Overview

```
Developer Machine
      │
      │  git push origin main
      ▼
GitHub (10zinnn/DOKPA-Frontend)
      │
      │  Triggers: .github/workflows/deploy.yml
      ▼
GitHub Actions Runner (ubuntu-latest)
      │
      ├─ 1. Checkout code
      ├─ 2. Setup Node.js 18
      ├─ 3. npm ci  (clean install from lock file)
      ├─ 4. npm run build  →  dist/
      └─ 5. FTP upload dist/ → Hostinger public_html/
                  │
                  ▼
         Live Site on Hostinger
         (dokpaexpeditions.com)
```

**Total pipeline time:** ~2–3 minutes

---

## Workflow File

**Location:** `.github/workflows/deploy.yml`

```yaml
name: Deploy to Hostinger

on:
  push:
    branches:
      - main
  workflow_dispatch:  # Manual trigger from GitHub Actions tab

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build project
        run: npm run build

      - name: Deploy to Hostinger
        uses: SamKirkland/FTP-Deploy-Action@v4.3.4
        with:
          server: ${{ secrets.FTP_SERVER }}
          username: ${{ secrets.FTP_USERNAME }}
          password: ${{ secrets.FTP_PASSWORD }}
          local-dir: ./dist/
          server-dir: ./public_html/
          dangerous-clean-slate: false
          exclude: |
            **/.git*
            **/.git*/**
            **/node_modules/**
```

---

## One-Time Setup (Required Before First Deploy)

### Step 1 — Get FTP Credentials from Hostinger

1. Log in at **hpanel.hostinger.com**
2. Go to **Hosting → Manage → Files → FTP Accounts**
3. Create or note the existing FTP account:
   - **FTP Server:** e.g., `ftp.dokpaexpeditions.com` or `srv123.hostinger.com`
   - **FTP Username:** e.g., `u123456789.dokpaexpeditions.com`
   - **FTP Password:** (set/reset in hPanel)
   - **Port:** 21

### Step 2 — Add Secrets to GitHub

Go to:  
`https://github.com/10zinnn/DOKPA-Frontend/settings/secrets/actions`

Click **"New repository secret"** and add each:

| Secret Name | Value | Where to find |
|---|---|---|
| `FTP_SERVER` | Your Hostinger FTP server | hPanel → FTP Accounts |
| `FTP_USERNAME` | Your FTP username | hPanel → FTP Accounts |
| `FTP_PASSWORD` | Your FTP password | hPanel → FTP Accounts (reset if needed) |

> **Security:** Secrets are encrypted and never appear in logs. Never hardcode credentials in the YAML file.

### Step 3 — Verify Hostinger public_html

Make sure your domain points to `public_html/` in hPanel:  
**hPanel → Hosting → Manage → File Manager → public_html**

---

## How to Deploy

### Option A — Automatic (Recommended)
Just push to `main`:
```bash
git add .
git commit -m "your change description"
git push origin main
```
GitHub Actions fires automatically. Monitor at:  
`https://github.com/10zinnn/DOKPA-Frontend/actions`

### Option B — Manual Trigger (No code change needed)
1. Go to: `https://github.com/10zinnn/DOKPA-Frontend/actions`
2. Click **"Deploy to Hostinger"** workflow
3. Click **"Run workflow"** → **"Run workflow"** button

Useful for re-deploying without making code changes.

### Option C — Manual FTP Upload (Bypass CI/CD)
When you need to deploy immediately without GitHub:
```bash
npm run build
# Then upload dist/ contents to public_html/ via:
# - Hostinger File Manager (hPanel → File Manager)
# - FileZilla (FTP client)
# - Any SFTP client
```

---

## What Gets Deployed

The `dist/` folder contents are uploaded to `public_html/`:

```
public_html/
├── assets/
│   ├── index-[hash].css     (all styles, minified)
│   └── index-[hash].js      (all JS, minified + tree-shaken)
├── .htaccess                (SPA routing + gzip + browser caching)
├── index.html               (React entry point)
├── robots.txt
├── logo-black.svg
├── logo-white.svg
└── *.jpg                    (hero, destination, activity images)
```

The `.htaccess` handles:
- **SPA routing:** All URLs redirect to `index.html` (so `/blog/1` works on refresh)
- **Gzip compression:** HTML, CSS, JS, JSON served compressed
- **Browser caching:** Images cached 1 year, CSS/JS cached 1 month

---

## Updating Content

### Add/Edit a Trip
1. Open `src/data/tripsData.js`
2. Add or modify a trip object
3. `git push origin main` → auto-deploys

### Add a Blog Post
1. Open `src/data/blogData.js`
2. Add a new post object following the existing schema
3. Push to `main`

### Add/Edit General Info content
1. Open `src/data/generalInfoData.js`
2. Edit the relevant section
3. Push to `main`

### Update images
1. Add image to `public/` folder
2. Reference it as `/image-name.jpg` in your data/component
3. Push to `main`

---

## Monitoring a Deploy

After pushing, go to:  
`https://github.com/10zinnn/DOKPA-Frontend/actions`

Each run shows:
- ✅ Green = deployed successfully
- ❌ Red = failed (click to see error log)

Common failure reasons:
| Error | Fix |
|---|---|
| `Authentication failed` | Check `FTP_PASSWORD` secret is correct |
| `Connection refused` | Check `FTP_SERVER` address in hPanel |
| `Build failed` | Fix the JS/JSX error shown in logs, then push again |
| `ENOENT dist/` | `npm run build` didn't run — check `package.json` scripts |

---

## Post-Deploy Checklist

After every significant deploy, verify:

- [ ] Homepage loads at `https://dokpaexpeditions.com`
- [ ] Navigation links work (Trips, General Info, Blog)
- [ ] Dark/Light mode toggle works
- [ ] Trip detail page loads (`/trips/some-trip-id`)
- [ ] Blog post loads (`/blog/1`)
- [ ] Page refresh on a sub-route works (tests `.htaccess`)
- [ ] Mobile menu opens/closes
- [ ] Images load (no broken images)
- [ ] Browser console shows no errors

---

## Rollback

If a deploy breaks the live site:

### Quick rollback via GitHub
```bash
git revert HEAD
git push origin main
# Triggers a new deploy with the reverted code
```

### Hard rollback to a specific commit
```bash
git log --oneline          # Find the good commit hash
git revert <bad-hash>      # Revert specific commit
git push origin main
```

### Emergency: Manual restore
1. Go to hPanel → File Manager → public_html
2. Restore from hPanel's automatic backup (hPanel → Backups)

---

## Local Build vs CI Build

| | Local (`npm run build`) | CI (GitHub Actions) |
|---|---|---|
| **Node version** | Whatever you have installed | Node 18 (pinned) |
| **Install** | `npm install` (uses cache) | `npm ci` (clean, from lock file) |
| **Output** | `dist/` on your machine | Uploaded directly to Hostinger |
| **Speed** | ~1 second | ~2–3 minutes total |
| **Use case** | Testing before push | Production deployment |

---

*See also: [BUILD_FLOW.md](./BUILD_FLOW.md) | [DEPLOYMENT.md](./DEPLOYMENT.md) | [QUICK_DEPLOY_SETUP.md](./QUICK_DEPLOY_SETUP.md)*
