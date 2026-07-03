# 🎮 GameGuide AI — Etheria Restart

[![Status](https://img.shields.io/badge/status-live-brightgreen?style=flat-square)](https://sonderox.my.id/game-guide/)
[![Version](https://img.shields.io/badge/version-40-blue?style=flat-square)]()
[![License](https://img.shields.io/badge/license-MIT-yellow?style=flat-square)](#license)
[![Heroes](https://img.shields.io/badge/heroes-93-orange?style=flat-square)]()
[![API](https://img.shields.io/badge/API-Express%20%2B%20SQLite-009639?style=flat-square&logo=node.js&logoColor=white)]()
[![Admin](https://img.shields.io/badge/admin-panel--live-8b5cf6?style=flat-square)](https://sonderox.my.id/game-guide/admin/)
[![Vanilla JS](https://img.shields.io/badge/built_with-Vanilla_JS-f7df1e?style=flat-square&logo=javascript&logoColor=black)]()
[![Cloudflare](https://img.shields.io/badge/CDN-Cloudflare-f38020?style=flat-square&logo=cloudflare&logoColor=white)]()

> A comprehensive, single-page game guide for **Etheria Restart** — featuring 93 heroes, tier lists across 6 game modes, a team builder with synergy analysis, matrix effects, shell databases, **skill effects system** with 104 unique buffs/debuffs, and **best builds** with 390 PvE/PvP builds from Prydwen.gg. Built with vanilla JavaScript, Express API, and glassmorphism UI.

**🔗 Live:** [https://sonderox.my.id/game-guide/](https://sonderox.my.id/game-guide/)
**🛠️ Admin:** [https://sonderox.my.id/game-guide/admin/](https://sonderox.my.id/game-guide/admin/)
**📖 API Docs:** [https://sonderox.my.id/api/docs](https://sonderox.my.id/api/docs)

---

## 📸 Screenshots

<!-- TODO: Add screenshots -->
| Home | Characters | Tier List |
|:---:|:---:|:---:|
| ![Home](docs/screenshots/home.png) | ![Characters](docs/screenshots/characters.png) | ![Tier List](docs/screenshots/tier-list.png) |
| **Team Builder** | **Matrix Effects** | **Shells** |
| ![Team Builder](docs/screenshots/team-builder.png) | ![Matrix Effects](docs/screenshots/matrix-effects.png) | ![Shells](docs/screenshots/shells.png) |

> 💡 *To capture screenshots: visit the [live site](https://sonderox.my.id/game-guide/) and navigate to each page.*

---

## 📑 Table of Contents

- [Overview](#-overview)
- [Features](#-features)
- [Architecture](#-architecture)
- [Tech Stack](#-tech-stack)
- [Project Structure](#-project-structure)
- [Data Model](#-data-model)
- [API Reference](#-api-reference)
- [Admin Panel](#-admin-panel)
- [Design System](#-design-system)
- [Special Features](#-special-features)
- [Deployment](#-deployment)
- [Data Pipeline](#-data-pipeline)
- [Development](#-development)
- [Configuration](#-configuration)
- [Changelog](#-changelog)
- [Roadmap](#-roadmap)
- [Contributing](#-contributing)
- [License](#-license)

---

## 🔭 Overview

**GameGuide AI** is a full-stack game companion for the gacha/strategy game **Etheria Restart**. It provides players with real-time hero data, tier rankings, team composition tools, equipment databases, and skill effect references — all wrapped in a sleek glassmorphism dark theme.

| Property | Value |
|---|---|
| **Project Name** | GameGuide AI – Etheria Restart |
| **URL** | https://sonderox.my.id/game-guide/ |
| **Type** | SPA Frontend + Express API + Admin Panel |
| **Database** | SQLite (17 tables, WAL mode) |
| **Theme** | Glassmorphism dark (`#0a0a1a`), Inter font, indigo/violet/cyan tokens |
| **Server** | Nginx + Express on Tencent Cloud VPS (`43.134.75.13`) |
| **CDN** | Cloudflare (`sonderox.my.id`) |
| **Author** | [KangPalak](https://github.com/KangPalak) (AI Operator) for Sondero |

---

## ✨ Features

The application consists of **8 fully-featured pages** + an **admin panel**, each serving a distinct purpose for Etheria Restart players:

### 1. 🏠 Home

- Hero landing page with animated gradient background
- **Featured Heroes** — Top 6 heroes ranked by aggregate tier score across all 6 game modes
- Quick navigation links to all sections
- Responsive grid layout

### 2. 👤 Characters

- Browse all **93 heroes** with detailed profiles
- **Multi-filter system** — filter by Role, Rarity, and Element simultaneously
- **Real-time search** with debounced input (300ms per page)
- **Hover preview cards** showing tier rankings, stats, and skill tags
- Element badges overlaid on character portraits

### 3. 📊 Tier List

- **Prydwen-style grid layout** — roles × tiers across game modes
- **6 game modes** supported:
  - 🗡️ Story/PvE
  - ⚔️ PvP
  - 🌟 Threshold Aurora
  - 🎭 DokiDoki
  - 👹 Terrormaton
  - ⛏️ Nether Quarry
- Color-coded tier badges (T0–T5) for instant visual parsing
- Sortable columns

### 4. 🛡️ Team Builder

- Build **5-man team compositions** from the hero roster
- **Synergy analysis engine** — evaluates team composition based on role coverage, element diversity, and skill tag combinations
- **5 prebuilt team compositions** for quick reference
- Drag-and-drop style team assembly

### 5. 🔷 Matrix Effects

- Database of **27 matrix set effects**
- Search and filter functionality
- Effect descriptions and stat bonuses
- Clean card-based layout

### 6. 🐚 Shells

- **43 shell items** with detailed stats
- Four rarity tiers with distinct visual styling:
  - 🔴 **Mythic** — Animated red/orange glow
  - 🟡 **Epic** — Yellow border
  - 🟣 **Unique** — Purple border
  - 🔵 **Rare** — Blue border

### 7. ⚡ Skill Effects

- **104 unique skill effects** — buffs, debuffs, and unique passives
- Effect badges on hero skill cards (color-coded by type)
- Click-to-expand effect details with full descriptions
- Data sourced from Prydwen.gg scraped skill data

| Type | Count | Color | Example |
|---|---|---|---|
| 🟢 Buff | 42 | Green | ATK Up, DEF Up, CRIT Rate Up |
| 🔴 Debuff | 36 | Red | Vulnerable, DEF Down, Stun |
| 🟣 Unique | 26 | Purple | Custom mechanics, passives |

**How it works:**
```
Skill Card → effectDetails badges → Click badge → Expand to full effect description
```

### 8. 🐚 Best Builds (NEW)

- **390 recommended builds** — 204 PvE + 186 PvP across 93 heroes
- Shell recommendations with icons, descriptions, and commentary
- Module set recommendations per slot with priority rankings
- Best Stats (main stats, substats, comments)
- Best Shell Passives per slot
- Skill Priority (PvE/PvP upgrade order)
- Data sourced from Prydwen.gg Builds tab

**Build card structure:**
```
┌─ Build #1: Artisan ─────────────────────┐
│  [Shell Icon] Artisan                    │
│  "Grants 700 DEF for 2 turns..."         │
│  Commentary: "Exceptional personal..."    │
│                                          │
│  Slot 1: [icon] Swiftrush [12/12]        │
│  Slot 2: [icon] Bramble [8/8]            │
│        › [icon] Strive [8/8]             │
│  Slot 3: [icon] Colossguard [6/6]        │
└──────────────────────────────────────────┘
```

---

## 🏗️ Architecture

```
┌─────────────┐     ┌─────────────┐     ┌───────────────────────┐
│   Browser    │────▶│  Cloudflare  │────▶│  Nginx (:80/:443)     │
│              │     │     CDN      │     │  ├─ /game-guide/      │
└─────────────┘     └─────────────┘     │  │  └─ Static SPA      │
                                         │  ├─ /game-guide/admin/ │
                                         │  │  └─ Admin Panel     │
                                         │  └─ /api/ → :3001     │
                                         │     └─ Express API     │
                                         └───────────────────────┘
                                                    │
                                         ┌──────────▼──────────┐
                                         │  SQLite (WAL mode)  │
                                         │  gameguide.db       │
                                         │  17 tables          │
                                         └─────────────────────┘
```

### Data Flow

```
Frontend SPA ──GET /api/heroes──▶ Express API ──SELECT──▶ SQLite
                                   │ (node-cache 5min)
Admin Panel ──PUT /api/heroes/:id─▶ Express API ──UPDATE──▶ SQLite
                                      │ (cache invalidation)
                                   ◀── Response ──
```

### Frontend Data Strategy

```
Page Load ──▶ Static data.js (instant render, 93 heroes)
         ──▶ Fetch /api/heroes (background)
              ├─ Merge with static data (API wins on conflict)
              ├─ Cache to localStorage (10min TTL)
              └─ Re-render if data changed
```

---

## 🛠️ Tech Stack

| Layer | Technology | Details |
|---|---|---|
| **Frontend** | Vanilla JavaScript (ES6+) | No framework — pure DOM manipulation |
| **Styling** | CSS3 | Glassmorphism, `backdrop-filter`, CSS Grid/Flexbox |
| **Typography** | Inter | Google Fonts CDN |
| **API Server** | Express.js 4 | REST API on port 3001 |
| **Database** | SQLite (better-sqlite3) | WAL mode, 17 tables, 7 views |
| **Admin Panel** | Alpine.js 3 + Tailwind CDN | No build step, SortableJS for drag-drop |
| **Auth** | JWT (jsonwebtoken) | 24h access tokens, bcrypt passwords |
| **Caching** | node-cache + localStorage | API 5min TTL, frontend 10min TTL |
| **Compression** | gzip (compression level 6) | Threshold 1KB |
| **Security** | XSS sanitization, CSRF, audit logging | `xss` package, HMAC-SHA256 |
| **Monitoring** | Health checks + disk alerts | `/api/health`, 70%/80%/90% thresholds |
| **Server** | Nginx | Reverse proxy, port 80/443 |
| **CDN** | Cloudflare | Cache purging via API, SSL termination |
| **Images** | Self-hosted `.webp` | 261 character portraits + 5 element icons |
| **Hosting** | Tencent Cloud VPS | Ubuntu, IP: `43.134.75.13` |

### Why Vanilla JS?

This project deliberately avoids frameworks (React, Vue, etc.) to maintain:
- **Zero build step** — deploy by copying files
- **Minimal bundle size** — no framework overhead
- **Full control** over DOM rendering and performance
- **Simplicity** — easy to understand and modify

---

## 📁 Project Structure

```
hermes-landing/
├── game-guide/                          # Frontend SPA
│   ├── index.html                       # SPA entry point
│   ├── js/
│   │   ├── app.js                       # Main app logic & router (~3069 lines)
│   │   ├── data.js                      # 93 hero character data (with effectDetails)
│   │   ├── shells.js                    # 43 shell items
│   │   ├── matrix_effects.js            # 27 matrix set effects
│   │   ├── team-builder.js              # Team synergy engine
│   │   ├── tier_data_patch.js           # Prydwen tier data injection
│   │   └── role_data_patch.js           # Prydwen role data injection
│   ├── css/
│   │   └── styles.css                   # Complete stylesheet (~2266 lines)
│   ├── assets/
│   │   ├── characters/                  # 261 hero portrait .webp files (128×128)
│   │   └── elements/                    # 5 element icons
│   └── README.md                        # This file
│
├── game-guide-api/                      # Express API Server
│   ├── src/
│   │   ├── index.js                     # Express server (port 3001)
│   │   ├── db.js                        # SQLite connection + schema init
│   │   ├── middleware/
│   │   │   ├── auth.js                  # JWT authentication
│   │   │   └── cache.js                 # node-cache middleware (5min TTL)
│   │   └── routes/
│   │       ├── heroes.js                # GET/POST/PUT/DELETE /api/heroes
│   │       ├── shells.js                # GET/POST/PUT/DELETE /api/shells
│   │       ├── matrix.js                # GET/POST/PUT/DELETE /api/matrix
│   │       ├── tiers.js                 # GET/PUT /api/tiers
│   │       ├── auth.js                  # POST /api/auth/login
│   │       ├── admin.js                 # GET /api/admin/stats, backup, cache
│   │       └── skill-effects.js         # GET /api/skill-effects
│   ├── admin/                           # Admin Panel (Alpine.js SPA)
│   │   ├── index.html                   # Admin entry point
│   │   ├── js/app.js                    # Admin logic (CRUD, settings, tags)
│   │   ├── js/tier-editor.js            # Kanban tier editor
│   │   ├── css/admin.css                # Admin theme
│   │   └── css/tier-editor.css          # Tier editor styles
│   ├── data/
│   │   └── gameguide.db                 # SQLite database (WAL mode)
│   ├── scripts/
│   │   ├── backup.sh                    # Automated daily backup (3AM)
│   │   └── monitor.sh                   # Disk monitoring (6h interval)
│   ├── backups/                         # DB backups (.gz, 7-day retention)
│   └── package.json
│
├── game-guide-admin/                    # Legacy admin (deprecated, use api/admin/)
└── nginx config                         # /etc/nginx/sites-available/hermes-landing
```

### File Size Summary

| File | Size | Purpose |
|---|---|---|
| `js/app.js` | ~120 KB | Main SPA logic & router (3069 lines) |
| `js/data.js` | 324 KB | Hero database (93 characters + effectDetails) |
| `css/styles.css` | ~58 KB | Complete glassmorphism theme (2266 lines) |
| `js/shells.js` | 52 KB | Shell item database (43 items) |
| `js/matrix_effects.js` | 48 KB | Matrix set effects (27 sets) |
| `js/tier_data_patch.js` | 16 KB | Tier rankings patch |
| `js/role_data_patch.js` | 8 KB | Role assignments patch |
| `js/team-builder.js` | 12 KB | Team synergy engine |
| `index.html` | 16 KB | SPA entry point |

---

## 📦 Data Model

### Database Tables (17)

| Table | Rows | Description |
|---|---|---|
| `heroes` | 93 | Hero characters with JSON fields |
| `hero_tags` | 606 | Skill tags per hero |
| `hero_tiers` | 552 | Tier rankings per hero per game mode |
| `skill_effects` | 104 | Unique skill effects (buffs/debuffs/uniques) |
| `shells` | 43 | Shell equipment items |
| `matrix_sets` | 27 | Matrix set effects |
| `team_compositions` | 5 | Prebuilt team compositions |
| `hero_builds` | 390 | PvE/PvP builds per hero (204+186) |
| `hero_build_stats` | 93 | Best stats, shell passives, skill priority |
| `admin_users` | 1 | Admin accounts |
| `audit_log` | — | CRUD audit trail |
| `elements` | 5 | Element types (Reason, Odd, etc.) |
| `roles` | 5 | Hero roles (DPS, Support, etc.) |
| `rarities` | 3 | Rarity tiers (SSR, SR, R) |
| `game_modes` | 6 | Tier list game modes |
| `tiers` | 6 | Tier definitions (T0–T5) |
| `shell_rarities` | 4 | Shell rarity tiers |
| `hero_roles` | — | Hero-role mappings |
| `shell_heroes` / `matrix_heroes` | — | Equipment-hero recommendations |

### Hero Object

Each of the 93 heroes follows this schema:

```javascript
{
  id: "lily",                      // Unique string ID
  name: "Lily",                    // Display name
  rarity: "SSR",                   // "SSR" | "SR" | "R"
  element: "Reason",               // "Reason" | "Odd" | "Hollow" | "Constant" | "Disorder"
  role: "Support",                 // "DPS" | "Support" | "Sustain" | "Control" | "Debuff"
  skillTags: ["Buff", "Heal"],     // Skill tag identifiers
  image: "lily.webp",              // Portrait filename
  imageDetail: "lily_detail.webp", // Detail portrait filename
  color: "#8b5cf6",                // Theme color
  tiers: {
    "Story/PvE": "T0",            // Tier ranking per game mode
    "PvP": "T0",
    "Threshold Aurora": "T0",
    "DokiDoki": "T1",
    "Terrormaton": "T0",
    "Nether Quarry": "T0"
  },
  skills: [
    {
      name: "Rapid Fire",
      tags: ["Vulnerable", "Single DMG"],
      effectDetails: [
        {
          name: "Vulnerable",
          type: "Debuff",
          description: "Increases DMG Taken by 30%.",
          slug: "vulnerable"
        }
      ]
    }
  ]
}
```

### Skill Effects System

**104 unique effects** stored in `skill_effects` table, referenced by slug from hero skills:

| Type | Count | Color | Examples |
|---|---|---|---|
| **Buff** | 42 | 🟢 Green | ATK Up, DEF Up, CRIT Rate Up, Heal Over Time |
| **Debuff** | 36 | 🔴 Red | Vulnerable, DEF Down, Stun, Silence |
| **Unique** | 26 | 🟣 Purple | Custom mechanics, conditional passives |

**Data flow:**
```
skill_effects table (master) ← slug → effectDetails in skills_json (per-skill)
                                         ↓
Frontend: Skill card → colored badges → click → expand to full description
```

### Tier System

| Tier | Label | Color | Hex | Description |
|:---:|---|:---:|:---:|---|
| **T0** | Meta / Best | 🔴 | `#ef4444` | Dominant pick, defines the meta |
| **T1** | Excellent | 🟠 | `#f59e0b` | Top-tier, highly competitive |
| **T2** | Great | 🟣 | `#8b5cf6` | Strong pick in most situations |
| **T3** | Good | 🔵 | `#3b82f6` | Solid, situational value |
| **T4** | Average | 🟢 | `#22c55e` | Usable, better options exist |
| **T5** | Below Average | ⚪ | `#64748b` | Niche or underperforming |

### Rarity Styling

| Rarity | Visual | Description |
|---|---|---|
| **SSR** | Gold gradient border + glow effect | Highest rarity |
| **SR** | Purple gradient border | Mid-tier rarity |
| **R** | Blue gradient border | Base rarity |
| **SSR + Constant/Disorder** | Rainbow animated border | Special element combo styling |

### Shell Rarity

| Rarity | Visual | Color |
|---|---|---|
| **Mythic** | Animated red/orange glow | 🔴🟠 |
| **Epic** | Yellow border | 🟡 |
| **Unique** | Purple border | 🟣 |
| **Rare** | Blue border | 🔵 |

### Element Types

| Element | Icon | File |
|---|---|---|
| Reason | 🧠 | `assets/elements/ele_reason.webp` |
| Odd | 🌀 | `assets/elements/ele_odd.webp` |
| Hollow | 👻 | `assets/elements/ele_hollow.webp` |
| Constant | ⚡ | `assets/elements/ele_constant.webp` |
| Disorder | 🔮 | `assets/elements/ele_disorder.webp` |

---

## 📡 API Reference

Base URL: `https://sonderox.my.id/api/`

Swagger UI: `https://sonderox.my.id/api/docs`

### Public Endpoints (no auth)

| Method | Path | Description |
|---|---|---|
| `GET` | `/api/heroes` | List all heroes (paginated, `?page=1&limit=20`) |
| `GET` | `/api/heroes/:id` | Get hero by ID |
| `GET` | `/api/shells` | List all shells (paginated) |
| `GET` | `/api/shells/:id` | Get shell by ID |
| `GET` | `/api/matrix` | List all matrix sets (paginated) |
| `GET` | `/api/matrix/:id` | Get matrix set by ID |
| `GET` | `/api/tiers` | Get all tier data (grouped by mode) |
| `GET` | `/api/skill-effects` | Get all effects (grouped by type) |
| `GET` | `/api/skill-effects/:slug` | Get single effect by slug |
| `GET` | `/api/builds` | Get all builds grouped by hero |
| `GET` | `/api/builds/:slug` | Get builds for single hero |
| `GET` | `/api/health` | Health check (disk, memory, cache stats) |

### Auth Endpoints

| Method | Path | Description |
|---|---|---|
| `POST` | `/api/auth/login` | Login, returns JWT |
| `POST` | `/api/auth/change-password` | Change password (auth required) |
| `GET` | `/api/auth/me` | Get current user info |

### Admin Endpoints (auth required)

| Method | Path | Description |
|---|---|---|
| `POST` | `/api/heroes` | Create hero |
| `PUT` | `/api/heroes/:id` | Update hero |
| `DELETE` | `/api/heroes/:id` | Delete hero |
| `POST` | `/api/shells` | Create shell |
| `PUT` | `/api/shells/:id` | Update shell |
| `DELETE` | `/api/shells/:id` | Delete shell |
| `POST` | `/api/matrix` | Create matrix set |
| `PUT` | `/api/matrix/:id` | Update matrix set |
| `DELETE` | `/api/matrix/:id` | Delete matrix set |
| `PUT` | `/api/tiers` | Bulk update tiers |
| `GET` | `/api/admin/stats` | Dashboard statistics |
| `POST` | `/api/admin/backup` | Trigger DB backup |
| `POST` | `/api/admin/cache/purge` | Clear API cache |
| `POST` | `/api/admin/sync-tiers` | Sync tier data |
| `POST` | `/api/admin/upload/:entity` | Upload images |

### Response Format

```json
// List endpoint
{
  "success": true,
  "data": [...],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 93,
    "totalPages": 5
  }
}

// Detail endpoint
{
  "success": true,
  "data": { ... }
}

// Error
{
  "success": false,
  "error": "Not found"
}
```

### Caching

| Layer | TTL | Strategy |
|---|---|---|
| API (node-cache) | 5 min | X-Cache: HIT/MISS headers |
| API (tiers) | 2 min | Shorter TTL for frequently updated data |
| Frontend (localStorage) | 10 min | `gg_heroes`, `gg_shells`, `gg_matrix`, `gg_tiers` |
| Nginx (static files) | 1 hour | `Cache-Control: public, must-revalidate` |
| Cloudflare | — | Purged on deploy via API |

---

## 🛠️ Admin Panel

**URL:** [https://sonderox.my.id/game-guide/admin/](https://sonderox.my.id/game-guide/admin/)

### Features

| Feature | Status | Description |
|---|---|---|
| Dashboard | ✅ | Stats overview (heroes, shells, matrix, tiers, DB size) |
| Hero CRUD | ✅ | Create/edit/delete with full form (17 fields) |
| Shell CRUD | ✅ | Create/edit/delete with slot, stats, source |
| Matrix CRUD | ✅ | Create/edit/delete with bonuses |
| Tier Editor | ✅ | Kanban drag-drop board (T0–T5, 6 modes) |
| Skill Tag Editor | ✅ | Add/remove skill tags per hero |
| Settings | ✅ | Profile, change password, admin overview |
| Image Upload | ✅ | Multer-based upload with drag-drop zones |
| Cache Purge | ✅ | Clear API cache from admin UI |
| Audit Logging | ✅ | All CRUD operations logged |

### Tech Stack

- **Alpine.js 3** — Reactive UI without build step
- **Tailwind CSS CDN** — Utility-first styling
- **SortableJS** — Drag-drop for tier editor
- **JWT Auth** — 24h access tokens

---

## 🎨 Design System

### Color Palette

```css
:root {
  /* Backgrounds */
  --bg-primary:     #0a0a1a;                    /* Deep dark background */
  --bg-card:        rgba(255, 255, 255, 0.03);  /* Card surfaces */
  --bg-card-hover:  rgba(255, 255, 255, 0.06);  /* Card hover state */

  /* Brand */
  --primary:        #6366f1;                    /* Indigo */
  --primary-light:  #8b5cf6;                    /* Violet */
  --accent:         #22d3ee;                    /* Cyan */

  /* Text */
  --text-primary:   #f1f5f9;                    /* Main text */
  --text-secondary: #94a3b8;                    /* Muted text */

  /* Tier Colors */
  --tier-t0: #ef4444;  /* Red — Meta */
  --tier-t1: #f59e0b;  /* Orange — Excellent */
  --tier-t2: #8b5cf6;  /* Purple — Great */
  --tier-t3: #3b82f6;  /* Blue — Good */
  --tier-t4: #22c55e;  /* Green — Average */
  --tier-t5: #64748b;  /* Gray — Below Average */

  /* Skill Effect Colors */
  --effect-buff:    #22c55e;  /* Green */
  --effect-debuff:  #ef4444;  /* Red */
  --effect-unique:  #8b5cf6;  /* Purple */
}
```

### Typography

| Element | Size | Weight | Font |
|---|---|---|---|
| Hero name | `1.1rem` | Bold (700) | Inter |
| Body text | `0.85rem` | Regular (400) | Inter |
| Tags / badges | `0.7rem` | Medium (500) | Inter |
| Headings | `1.5–2.5rem` | Bold (700) | Inter |

**Font loading:** Google Fonts CDN via `<link>` tag — no FOUT issues.

### Glassmorphism Cards

```css
.glass-card {
  background: rgba(255, 255, 255, 0.03);
  backdrop-filter: blur(16px);
  -webkit-backdrop-filter: blur(16px);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 12px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
}
```

### Layout

- **Grid system:** CSS Grid with `auto-fill` and `minmax()` for responsive cards
- **Breakpoints:** Mobile-first, responsive at 768px and 1024px
- **Card sizes:** 160–200px character cards, fluid team builder panels

---

## 🌟 Special Features

### Hover Preview Card

When hovering over a hero card, a rich preview card appears with a 200ms delay:

```
┌──────────────────────────────┐
│  [Portrait]  Hero Name       │
│  64×64px     SSR · Reason · DPS │
│──────────────────────────────│
│  Story/PvE │ PvP │ Threshold │
│    [T0]    │[T1] │   [T0]   │
│──────────────────────────────│
│  DokiDoki │ Terrormaton │ Nether │
│    [T1]   │    [T0]    │  [T0]  │
└──────────────────────────────┘
```

**Behavior:**
- 200ms delay before showing (prevents flicker on quick mouse-overs)
- Auto-positions to left or right based on viewport edge proximity
- 2×3 tier grid with color-coded badges
- Hides automatically on scroll events

### Element Badges

- Displayed in the **bottom-left corner** of hero portrait cards
- Size: 16×16px
- Source: `assets/elements/ele_{element}.webp`

### Skill Effect Badges (NEW)

In hero detail page, each skill shows its associated effects as clickable badges:

```
┌─ Skill Card: Rapid Fire ──────────────────┐
│  Tags: [Vulnerable] [Single DMG]          │
│                                            │
│  Effects:                                  │
│  ┌──────────────────┐                      │
│  │ 🔴 Vulnerable    │  ← click to expand  │
│  └──────────────────┘                      │
│  ┌──────────────────────────────────────┐  │
│  │ Vulnerable (Debuff)                  │  │
│  │ Increases DMG Taken by 30%.          │  │
│  └──────────────────────────────────────┘  │
└────────────────────────────────────────────┘
```

- **Buff** → Green badge
- **Debuff** → Red badge
- **Unique** → Purple badge
- Click badge → inline panel expands with full effect description

### Featured Heroes (Home Page)

The home page showcases the **top 6 heroes** by aggregate tier score:

```javascript
// Tier scoring formula
T0 = 5 points, T1 = 4, T2 = 3, T3 = 2, T4 = 1, T5 = 0

// Aggregate across 6 game modes
// Example: Lily — T0 in all 6 modes = 30 points (max)
```

### Best Builds

Each hero has PvE and PvP build recommendations with full detail:

- **Shell recommendation** — icon, description, commentary
- **Module sets** — per-slot recommendations with priority rankings (e.g., Bramble > Strive)
- **Best Stats** — main stats per slot + substat priorities
- **Shell Passives** — recommended passives per slot
- **Skill Priority** — upgrade order for PvE and PvP

**Data source:** [Prydwen.gg](https://www.prydwen.gg/etheria-restart/) Builds tab

### Character Filters

All filters are **combinable** — you can stack Role + Rarity + Element + Search:

| Filter Type | Options |
|---|---|
| **Role** | DPS, Support, Sustain, Control, Debuff |
| **Rarity** | SSR, SR, R |
| **Element** | Reason, Odd, Hollow, Constant, Disorder |
| **Search** | Free-text name search with debounce |

### Loading Skeleton

When data is loading from API, animated skeleton placeholders show instead of empty space:
- 8–12 skeleton cards with pulse animation
- Smooth transition to real content when data arrives
- Error state with retry button if API fails

---

## 🚀 Deployment

### Infrastructure

```
┌─────────────┐     ┌─────────────┐     ┌───────────────────────┐
│   Browser    │────▶│  Cloudflare  │────▶│  Nginx (VPS)          │
│              │     │     CDN      │     │  43.134.75.13         │
└─────────────┘     └─────────────┘     │  Port 80/443          │
                                         │  ├─ /game-guide/      │
                                         │  └─ /api/ → :3001     │
                                         └───────────────────────┘
```

| Component | Details |
|---|---|
| **VPS** | Tencent Cloud, Ubuntu, `43.134.75.13` |
| **Frontend** | `/var/www/hermes-landing/game-guide/` |
| **API Server** | `/var/www/hermes-landing/game-guide-api/` (PM2: `gameguide-api`) |
| **Web Server** | Nginx (static + reverse proxy) |
| **CDN** | Cloudflare — `sonderox.my.id` |
| **SSL** | Cloudflare Origin Certificate + Full (Strict) |
| **Caching** | Versioned URLs (`?v=39`), Cloudflare purge on updates |

### Cache Strategy

All static assets use **version query strings** for cache busting:

```html
<link rel="stylesheet" href="css/styles.css?v=39">
<script src="js/app.js?v=39"></script>
<script src="js/data.js?v=39"></script>
```

### Auto-Deploy Script

```bash
~/.hermes/scripts/deploy_gameguide.sh

# What it does:
# 1. Bumps cache version in index.html
# 2. Purges Cloudflare cache (if credentials configured)
# 3. Restarts PM2 process if needed
```

### Deploying Updates

```bash
# 1. SSH into the VPS
ssh ubuntu@43.134.75.13

# 2. Navigate to project directory
cd /var/www/hermes-landing/game-guide/

# 3. Update files (rsync, git pull, or manual copy)

# 4. Run deploy script (auto-bumps cache + purges CF)
~/.hermes/scripts/deploy_gameguide.sh

# 5. Verify
curl -s https://sonderox.my.id/game-guide/ | head -20
```

### Backup Strategy

- **Automated daily backup** at 3:00 AM (via cron)
- SQLite `VACUUM INTO` + gzip compression
- 7-day retention in `/var/www/hermes-landing/game-guide-api/backups/`
- Manual backup: `POST /api/admin/backup` (admin auth required)

---

## 🔄 Data Pipeline

The hero data is sourced from [Prydwen.gg](https://www.prydwen.gg/etheria-restart/) and processed through a multi-stage pipeline:

```
┌──────────────┐    ┌──────────────────┐    ┌──────────────┐    ┌──────────┐
│  Prydwen.gg  │───▶│  scrape_prydwen  │───▶│  Processing  │───▶│  data.js │
│  (Source)     │    │  .py             │    │  Scripts     │    │  (Hero)  │
└──────────────┘    └──────────────────┘    └──────────────┘    └──────────┘
                                                                      │
                                            ┌─────────────────────────┤
                                            ▼                         ▼
                                    ┌───────────────┐    ┌───────────────────┐
                                    │ tier_data_    │    │ role_data_        │
                                    │ patch.js      │    │ patch.js          │
                                    └───────────────┘    └───────────────────┘
                                            │                    │
                                            ▼                    ▼
                                    ┌─────────────────────────────────────┐
                                    │  Runtime injection via <script>     │
                                    │  tags — patches applied at load     │
                                    └─────────────────────────────────────┘
```

### Skill Effects Pipeline

```
/tmp/etheria_characters.json (scraped from Prydwen.gg)
    │
    ├─ Extract unique effects by slug (104 unique)
    │
    ├─ Import into SQLite `skill_effects` table
    │
    └─ Update `skills_json` in `heroes` table with effectDetails
         │
         └─ Update `data.js` with effectDetails per skill
```

### Character Portraits

- **Format:** `.webp` (optimized for web)
- **Size:** 128×128 pixels
- **Count:** 261 total (includes alternate/detail portraits)
- **Location:** `assets/characters/`

---

## 💻 Development

### Prerequisites

- Node.js 18+ (for API server)
- Python 3.x (for data scraping scripts)
- Nginx (for reverse proxy)

### Local Development

```bash
# Frontend (static SPA)
cd /var/www/hermes-landing/game-guide/
python3 -m http.server 8080

# API Server
cd /var/www/hermes-landing/game-guide-api/
npm install
node src/index.js  # Runs on port 3001

# Or with PM2
pm2 start ecosystem.config.js
pm2 logs gameguide-api
```

### Making Changes

```bash
# Edit frontend styles
vim css/styles.css

# Edit main app logic
vim js/app.js

# Edit hero data
vim js/data.js

# Edit API routes
vim game-guide-api/src/routes/heroes.js

# Update cache version after changes
~/.hermes/scripts/deploy_gameguide.sh
```

### Key Architecture Notes

| Concept | Implementation |
|---|---|
| **Routing** | Hash-based (`#/home`, `#/characters`, etc.) — handled in `app.js` |
| **Rendering** | `innerHTML` with template literals — no virtual DOM |
| **State** | Module-level variables in `app.js` — no state management library |
| **Data access** | API-first with static fallback + localStorage cache |
| **Search** | Debounced `input` event listeners (300ms per page) |
| **Hover cards** | Absolute-positioned `div` with pointer events disabled during scroll |
| **Auth** | JWT with 24h expiry, `admin` role maps to `super-admin` |
| **Caching** | 3-layer: node-cache (5min) → localStorage (10min) → Cloudflare |

---

## ⚙️ Configuration

### Nginx Configuration

```nginx
server {
    listen 80;
    server_name sonderox.my.id;

    root /var/www/hermes-landing;
    index index.html;

    # API proxy
    location /api/ {
        proxy_pass http://127.0.0.1:3001;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Game Guide SPA
    location /game-guide/ {
        try_files $uri $uri/ /game-guide/index.html;

        # Cache static assets
        location ~* \.(webp|png|jpg|css|js)$ {
            expires 1h;
            add_header Cache-Control "public, must-revalidate";
        }

        # Gzip
        gzip on;
        gzip_types text/css application/javascript;
    }

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
}
```

### PM2 Ecosystem

```javascript
// ecosystem.config.js
module.exports = {
  apps: [{
    name: 'gameguide-api',
    script: 'src/index.js',
    cwd: '/var/www/hermes-landing/game-guide-api',
    instances: 1,
    autorestart: true,
    max_memory_restart: '256M',
    env: {
      NODE_ENV: 'production',
      PORT: 3001
    }
  }]
};
```

### Environment Variables

```bash
# API Server (.env in game-guide-api/)
JWT_SECRET=your-jwt-secret
PORT=3001

# Cloudflare (in ~/.hermes/.env)
CLOUDFLARE_ZONE_ID=your-zone-id
CLOUDFLARE_API_TOKEN=your-api-token
```

---

## 📝 Changelog

### v40 (Current) — Best Builds System

- 🐚 **Builds:** 390 PvE/PvP builds for 93 heroes from Prydwen.gg
- 🐚 **Build Cards:** Shell icons, descriptions, commentary, module set recommendations
- 📊 **Best Stats:** Main stats, substats, comments per hero
- 🛡️ **Shell Passives:** Slot-based passive recommendations
- ⬆️ **Skill Priority:** PvE/PvP upgrade order
- 📡 **API:** `GET /api/builds` + `GET /api/builds/:slug`
- 📦 **Database:** `hero_builds` (390) + `hero_build_stats` (93)

### v39 — Skill Effects System

- ⚡ **Skill Effects:** 104 unique effects (42 Buff, 36 Debuff, 26 Unique)
- ⚡ **Effect Badges:** Color-coded badges on hero skill cards
- ⚡ **Expandable Detail:** Click badge → inline effect description
- 📡 **API:** `GET /api/skill-effects` + `GET /api/skill-effects/:slug`
- 📦 **Database:** `skill_effects` table with slug-based lookup
- 🔄 **Data:** Updated 90 heroes with `effectDetails` in `skills_json`
- 📄 **Static:** Updated `data.js` with effectDetails for 214 skills

### v38 — Phase 4 Complete + Admin Form Fields

- 🔒 **Security:** CSRF (HMAC-SHA256), XSS sanitization, audit logging
- ⚡ **Performance:** gzip compression, API caching (5min TTL), query optimization
- 📊 **Monitoring:** Health check (`/api/health`), disk alerts, PM2 logrotate
- 📝 **Logging:** Morgan `combined` format, request logging
- 🛡️ **Headers:** HSTS, X-Frame-Options, X-Content-Type-Options
- 📖 **Docs:** Swagger UI at `/api/docs`
- 📄 **Pagination:** `?page=1&limit=20` for list endpoints
- ✅ **Graceful Shutdown:** SIGTERM/SIGINT handlers

### v37 — Phase 3 Complete + Skill Effects Foundation

- 🔄 **API-First:** Frontend fetches from `/api/heroes`, `/api/shells`, `/api/matrix`
- 💾 **localStorage Cache:** 10min TTL with background refresh
- 💀 **Loading Skeleton:** Animated placeholders during data fetch
- 🛡️ **Error Handling:** Try-catch + `.filter(Boolean)` in transform functions

### v36 — Phase 2 Complete

- 🛠️ **Admin Panel:** Full CRUD for heroes, shells, matrix
- 📊 **Tier Editor:** Kanban drag-drop board (T0–T5, 6 modes)
- 🏷️ **Tag Editor:** Add/remove skill tags per hero
- ⚙️ **Settings:** Profile, change password, admin overview
- 📤 **Image Upload:** Multer-based with drag-drop zones
- 🧹 **Cache Purge:** Clear API cache from admin UI

### v35 — Phase 1 Complete

- 📡 **API Server:** Express.js + SQLite (better-sqlite3)
- 🔐 **Auth:** JWT login + admin user management
- 📦 **Database:** 17 tables, WAL mode, automated backups
- 🔄 **Nginx:** Reverse proxy `/api/` → `:3001`

### v28–v34 — Frontend SPA

<details>
<summary>View early changelog</summary>

#### v28
- 🔒 Fixed XSS vulnerability in hero ID rendering
- 🔄 Unified all asset cache versions
- 🧹 Removed 13 stale files (~760KB saved)
- ⚡ Separated search timeouts per page

#### v27
- Initial glassmorphism theme implementation
- Added matrix effects page
- Shell database expansion (43 items)

#### v26
- Team builder with synergy analysis
- 5 prebuilt team compositions
- Role data patch system

#### v25
- Prydwen-style tier list grid
- 6 game mode support
- Tier data patch system

#### v24
- Character browser with filters
- Hover preview cards
- Element badges

#### v23
- Initial SPA release
- Home page with featured heroes
- Core routing and navigation

</details>

---

## 🗺️ Roadmap

| Priority | Feature | Status | Description |
|:---:|---|:---:|---|
| 1 | **Skill Effects** | ✅ Done | 104 effects with expandable badges |
| 2 | **API Server** | ✅ Done | Express + SQLite + JWT auth |
| 3 | **Admin Panel** | ✅ Done | Full CRUD + tier editor + settings |
| 4 | **Phase 4 Polish** | ✅ Done | Security, caching, monitoring, docs |
| 5 | **is_published Flag** | 🔲 Sprint 1 | Draft/publish control per hero |
| 6 | **sort_order Field** | 🔲 Sprint 1 | Custom ordering in lists |
| 7 | **Modular Code** | 🔲 Sprint 1 | Split 120KB app.js into modules |
| 8 | **Builds Section** | ✅ Done | 390 PvE/PvP builds from Prydwen.gg |
| 9 | **Bulk Import/Export** | 🔲 Sprint 2 | JSON import/export for heroes/shells |
| 10 | **PWA Support** | 🔲 Future | Offline access, home screen install |

---

## 🤝 Contributing

Contributions are welcome! Whether it's bug fixes, new features, or data updates — all help is appreciated.

### How to Contribute

1. **Fork** the repository
2. **Create** a feature branch
   ```bash
   git checkout -b feature/awesome-feature
   ```
3. **Make** your changes
4. **Test** thoroughly across all pages
5. **Commit** with clear messages
   ```bash
   git commit -m "feat: add new hero filter by skill tags"
   ```
6. **Push** to your fork
   ```bash
   git push origin feature/awesome-feature
   ```
7. **Open** a Pull Request with a clear description

### Contribution Guidelines

- **Code style:** Follow existing patterns — vanilla JS, no frameworks
- **CSS:** Use existing design tokens (CSS variables), maintain glassmorphism theme
- **Data:** When adding heroes, follow the existing data schema exactly
- **Testing:** Verify all pages render correctly after changes
- **Commits:** Use [Conventional Commits](https://www.conventionalcommits.org/) format:
  - `feat:` — new features
  - `fix:` — bug fixes
  - `data:` — hero/shell/matrix data updates
  - `style:` — CSS/design changes
  - `docs:` — documentation updates

### Reporting Issues

Found a bug? [Open an issue](../../issues) with:
- Steps to reproduce
- Expected vs actual behavior
- Browser and OS information
- Screenshots if applicable

---

## 📄 License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.

```
MIT License

Copyright (c) 2024-2026 Sondero / KangPalak

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

---

## 🙏 Acknowledgments

- **[Prydwen.gg](https://www.prydwen.gg/etheria-restart/)** — Primary data source for hero information, tier rankings, role classifications, and skill effects
- **[Etheria Restart](https://etheria.com)** — The game this guide is built for
- **[Inter](https://rsms.me/inter/)** — Beautiful typeface by Rasmus Andersson
- **Cloudflare** — CDN and SSL services
- **The Etheria Restart community** — For feedback and feature requests

---

<div align="center">

**Built with ❤️ by [KangPalak](https://github.com/KangPalak) for the Etheria Restart community**

[⬆ Back to top](#-gameguide-ai--etheria-restart)

</div>
