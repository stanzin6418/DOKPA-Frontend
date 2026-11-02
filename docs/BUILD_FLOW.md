# 🏗️ Build Flow — Dokpa Expeditions Frontend

> **Last Updated:** November 2026  
> **Stack:** React 18 + Vite 6 + React Router 7 + Vanilla CSS (Design System)  
> **Repo:** https://github.com/10zinnn/DOKPA-Frontend  
> **Upstream:** https://github.com/stanzin6418/DOKPA-Frontend

---

## Project Structure

```
DOKPA-Frontend/
├── public/                   # Static assets (copied as-is to dist/)
│   ├── .htaccess             # Apache SPA routing + gzip + cache headers
│   ├── robots.txt
│   ├── logo-black.svg
│   ├── logo-white.svg
│   └── *.jpg                 # Hero, destination, activity images
│
├── src/
│   ├── components/           # Reusable UI components
│   │   ├── Navigation.jsx    # Sticky glassmorphism nav + dark mode toggle
│   │   ├── Navigation.css
│   │   ├── Activities.jsx    # Adventure slider with category tabs
│   │   ├── FeaturedTrips.jsx
│   │   ├── Footer.jsx
│   │   ├── Hero.jsx
│   │   ├── About.jsx
│   │   ├── Destinations.jsx
│   │   ├── Services.jsx
│   │   ├── Testimonials.jsx
│   │   ├── TrustBar.jsx
│   │   ├── ContactForm.jsx
│   │   └── Loading.jsx
│   │
│   ├── pages/                # Route-level page components
│   │   ├── Home.jsx          # Combines all homepage sections
│   │   ├── TripsPage.jsx     # All trips grouped by category
│   │   ├── TripDetailPage.jsx  # Individual trip + itinerary
│   │   ├── ActivitiesPage.jsx
│   │   ├── ActivityDetailPage.jsx
│   │   ├── DestinationsPage.jsx
│   │   ├── DestinationDetailPage.jsx
│   │   ├── GeneralInfoPage.jsx  # Ladakh travel guide (7 sections)
│   │   ├── BlogPage.jsx         # Blog listing with search + filter
│   │   └── BlogPostPage.jsx     # Individual blog article + sidebar
│   │
│   ├── data/                 # Static data (migrated from WordPress)
│   │   ├── tripsData.js      # 13 trips (105KB) — itineraries, inclusions, etc.
│   │   ├── activitiesData.js # Adventures: trekking, climbing, rafting, etc.
│   │   ├── destinationsData.js
│   │   ├── generalInfoData.js  # Acclimatization, permits, seasons, packing, FAQs
│   │   └── blogData.js         # Blog posts with full content blocks
│   │
│   ├── App.jsx               # React Router route definitions
│   ├── main.jsx              # ReactDOM entry point
│   └── index.css             # Global design system (CSS variables + tokens)
│
├── docs/                     # Project documentation
├── .github/
│   └── workflows/
│       └── deploy.yml        # GitHub Actions CI/CD → Hostinger FTP
├── vite.config.js            # base: './' (required for Hostinger shared hosting)
└── package.json
```

---

## Design System (index.css)

All styling uses CSS custom properties defined in `:root` and `[data-theme='dark']`.

### Color Tokens
| Token | Light Mode | Dark Mode |
|---|---|---|
| `--bg-app` | `#F7F3F0` (sand) | `#1A1F23` (charcoal) |
| `--bg-alt` | `#EFEDE9` | `#121619` |
| `--bg-card` | `#FFFFFF` | `#192025` |
| `--bg-card-subtle` | `#F3EFEA` | `#14181B` |
| `--text-main` | `#1A1F23` | `#F7F3F0` |
| `--text-body` | `#2B3339` | `#D1D8DD` |
| `--text-muted` | `#5A636A` | `#A1A9AF` |
| `--color-gold` | `#D4AF37` | `#D4AF37` |
| `--color-sage` | `#6B8E6F` | `#6B8E6F` |
| `--border-card` | `rgba(0,0,0,0.08)` | `rgba(255,255,255,0.09)` |
| `--shadow-card` | `0 8px 24px -4px rgba(0,0,0,0.06)` | `0 8px 24px -4px rgba(0,0,0,0.4)` |

### Spacing Tokens
```
--s-1: 0.25rem    --s-2: 0.5rem    --s-3: 0.75rem
--s-4: 1rem       --s-5: 1.25rem   --s-6: 1.5rem
--s-8: 2rem       --s-10: 2.5rem   --s-12: 3rem
--s-16: 4rem      --s-20: 5rem     --s-24: 6rem
```

> All page sections use `clamp()` for fluid padding — no cramped layout at any viewport.

---

## Routing (App.jsx)

```
/                         → Home.jsx
/trips                    → TripsPage.jsx
/trips/:tripId            → TripDetailPage.jsx
/activities               → ActivitiesPage.jsx
/activities/:activityId   → ActivityDetailPage.jsx
/destinations             → DestinationsPage.jsx
/destinations/:destId     → DestinationDetailPage.jsx
/general-info             → GeneralInfoPage.jsx
/blog                     → BlogPage.jsx
/blog/:postId             → BlogPostPage.jsx
```

React Router 7 with `BrowserRouter`. The `public/.htaccess` rewrites all non-file requests to `index.html` so direct URL access and refresh work on Hostinger.

---

## Local Development

### Prerequisites
- Node.js 18+
- npm

### Setup (first time)
```bash
git clone https://github.com/10zinnn/DOKPA-Frontend.git
cd DOKPA-Frontend
npm install
```

### Start dev server
```bash
npm run dev
# → http://localhost:5175
```

Vite HMR (Hot Module Replacement) is active — any file save instantly reflects in browser with no full reload.

### Other commands
```bash
npm run build    # Production build → dist/
npm run preview  # Serve dist/ locally at http://localhost:4173
npm run lint     # ESLint check on src/
```

---

## Production Build

```bash
npm run build
```

**Output (`dist/`):**
```
dist/
├── assets/
│   ├── index-[hash].css    ~9KB  (gzip: 2.6KB)
│   └── index-[hash].js     ~530KB (gzip: 130KB)
├── .htaccess               Apache config (SPA + compression + caching)
├── index.html              Entry point
├── robots.txt
├── logo-black.svg
├── logo-white.svg
└── *.jpg                   All images from public/
```

### Key vite.config.js setting
```js
export default defineConfig({
  plugins: [react()],
  base: './',   // ← CRITICAL for Hostinger shared hosting (relative asset paths)
})
```

Without `base: './'`, all asset paths would be absolute (`/assets/...`) and break on Hostinger.

---

## Data Layer

All content lives in `src/data/` as plain JS objects — no database, no CMS API calls at runtime.

| File | Size | Contents |
|---|---|---|
| `tripsData.js` | 105KB | 13 trips migrated from WordPress (WP Travel Engine API) — full itineraries, inclusions, exclusions, pricing, images |
| `activitiesData.js` | 12KB | Trekking, peak climbing, cultural tours, rafting, biking, etc. |
| `destinationsData.js` | 8KB | Leh, Nubra, Zanskar, Kargil, Pangong, etc. |
| `generalInfoData.js` | 14KB | Acclimatization, permits, seasons, packing, etiquette, logistics, FAQs |
| `blogData.js` | 13KB | Blog posts with structured content blocks (paragraph, heading, quote, callout) |

> **To add new trips or content:** Edit the relevant file in `src/data/`, commit, and push to `main`. GitHub Actions will auto-build and deploy.

---

## Page Architecture Notes

### GeneralInfoPage
- Sticky sub-navigation bar (7 section anchors)
- Sections: Acclimatization → Permits → Seasons → Gear & Packing → Culture & Ethics → Logistics → FAQs
- All cards use `--bg-card` (#FFF light / #192025 dark) with `--border-card` and `--shadow-card`

### BlogPage
- Category pill filter (6 categories) + live search
- Featured banner (full grid layout) shown when no filter active
- Stories grid (`auto-fill, minmax(320px, 1fr)`)
- Newsletter signup card at bottom

### BlogPostPage
- 2-column layout: `2.2fr 1fr` (article main + sidebar)
- Content block renderer: `paragraph`, `heading`, `quote`, `callout` types
- Related posts section at bottom
- Collapses to single column on mobile

### TripDetailPage
- Day-by-day itinerary accordion
- Inclusions / Exclusions tabs
- Fixed day buttons visibility fix applied for light mode

---

*See also: [DEPLOYMENT.md](./DEPLOYMENT.md) | [GITHUB_ACTIONS_DEPLOYMENT.md](./GITHUB_ACTIONS_DEPLOYMENT.md)*
