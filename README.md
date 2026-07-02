# 🎮 GameGuide Migration Plan: Static → SQLite + Admin Panel

> Comprehensive technical documentation for migrating the Etheria Restart GameGuide SPA from static JSON files to a SQLite database with a full admin panel.

## 📋 Table of Contents

| Document | Description | Size |
|----------|-------------|------|
| [ARCHITECTURE.md](./ARCHITECTURE.md) | System architecture, API design, tech stack, deployment | 25KB |
| [DATABASE-SCHEMA.md](./DATABASE-SCHEMA.md) | Complete SQLite schema with 17 tables, views, triggers | 40KB |
| [ADMIN-PANEL.md](./ADMIN-PANEL.md) | Admin panel design: pages, wireframes, security, components | 75KB |

---

## 🎯 Project Overview

**Current State (2026-07-02): All 4 Phases Complete + Enhancements**

| Component | Status | Details |
|---|---|---|
| SQLite Database | ✅ | 17 tables, 7 views, 5 triggers, WAL mode |
| Express API | ✅ | PM2 `gameguide-api` on port 3001 |
| Admin Panel | ✅ | `sonderox.my.id/game-guide/admin/` |
| Frontend SPA | ✅ | API-first + static fallback + localStorage cache |
| Skill Effects | ✅ | 104 unique effects (42 Buff, 36 Debuff, 26 Unique) |
| Security | ✅ | CSRF, XSS sanitization, audit logging |
| Performance | ✅ | gzip, API caching, query optimization |
| Monitoring | ✅ | Health check, disk alerts, PM2 logrotate |
| Documentation | ✅ | Swagger UI at `/api/docs` |

**Data:**
- ✅ 93 heroes (with skills, effectDetails, tags, tiers)
- ✅ 43 shells (with stats, slots, sources)
- ✅ 27 matrix sets (with bonuses, pieces)
- ✅ 5 team compositions
- ✅ 104 skill effects (expandable badges in hero detail)
- ✅ 606 hero tags
- ✅ 552 tier entries (6 modes × 92 heroes)

---

## 🏗️ Architecture (Implemented)

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

**Data Flow (Live):**
```
Browser → GET /api/heroes → Express → SQLite → JSON response
        → GET /api/shells → Express → SQLite → JSON response
        → GET /api/matrix → Express → SQLite → JSON response
        → GET /api/tiers  → Express → SQLite → JSON response
        → GET /api/skill-effects → Express → SQLite → JSON response

Admin Panel → PUT /api/heroes/:id → SQLite DB
            → PUT /api/shells/:id → SQLite DB
            → PUT /api/matrix/:id → SQLite DB
            → PUT /api/tiers/bulk → SQLite DB
```

**Frontend Data Strategy:**
```
1. Static JS files load first (instant render, 93 heroes)
2. localStorage cache checked (10min TTL)
3. API fetch in background (merge with static, update cache)
4. If API fails, static data remains as fallback
```

**3-Layer Caching:**
```
Layer 1: node-cache (5min TTL) — API response cache, X-Cache headers
Layer 2: localStorage (10min TTL) — Frontend data cache
Layer 3: Cloudflare CDN — Static asset cache, purged on deploy
```

---

## 📊 Database Schema Summary

**17 Tables:**

| Table | Rows | Description |
|---|---|---|
| `heroes` | 93 | Hero characters with JSON fields |
| `hero_tags` | 606 | Skill tags per hero |
| `hero_tiers` | 552 | Tier rankings per hero per game mode |
| `skill_effects` | 104 | Unique skill effects (buffs/debuffs/uniques) |
| `shells` | 43 | Shell equipment items |
| `matrix_sets` | 27 | Matrix set effects |
| `team_compositions` | 5 | Prebuilt team compositions |
| `admin_users` | 1 | Admin accounts |
| `audit_log` | — | CRUD audit trail |
| `elements` | 5 | Element types |
| `roles` | 5 | Hero roles |
| `rarities` | 3 | Rarity tiers |
| `game_modes` | 6 | Tier list game modes |
| `tiers` | 6 | Tier definitions (T0–T5) |
| `shell_rarities` | 4 | Shell rarity tiers |
| `hero_roles` | — | Hero-role mappings |
| `shell_heroes` / `matrix_heroes` | — | Equipment-hero recommendations |

**7 Views** for common queries (denormalized hero list, tier matrix, equipment recs)

**5 Triggers** for auto-updating `updated_at` timestamps

---

## ⚡ Skill Effects System (NEW)

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

**API Endpoints:**
- `GET /api/skill-effects` — All effects grouped by type
- `GET /api/skill-effects/:slug` — Single effect by slug

**Frontend:**
- Effect badges on hero skill cards (color-coded by type)
- Click-to-expand inline panel with full effect description
- 214 skills across 93 heroes have effectDetails

---

## 🖥️ Admin Panel (Implemented)

**URL:** [https://sonderox.my.id/game-guide/admin/](https://sonderox.my.id/game-guide/admin/)

**Pages:**
1. ✅ Login (JWT auth with refresh tokens)
2. ✅ Dashboard (stats, quick actions, recent changes)
3. ✅ Heroes Management (list, search, filter, full edit form with all fields)
4. ✅ Shells Management (list, search, full edit form with all fields)
5. ✅ Matrix Management (list, search, full edit form with all fields)
6. ✅ Tier Editor (Kanban board: T0-T5 × 6 modes, drag-drop with SortableJS)
7. ✅ Teams Management (list)
8. ✅ Settings (profile, change password, admin overview)
9. ✅ Hero Tag Editor (add/remove skill tags)
10. ✅ Image Upload (multer endpoint + drag-drop UI)

**Hero Edit Form Fields:**
- Basic: name, title, rarity, element, role, rating, description
- Visual: image (upload + preview), image_detail (upload + preview), color (picker), avatar (letter)
- Data: faction, stats_json (8-stat grid editor), skills_json (collapsible skill cards)
- Tags: skill tags (removable chips + quick-add suggestions)

**Shell Edit Form Fields:**
- Basic: name, slug, rarity, slot, main_stat, source
- Data: sub_stats_json (editable rows), set_belongs_to_json (chips)
- Visual: icon (upload + preview), local_icon
- Effects: effect, effect_awakened, description

**Matrix Edit Form Fields:**
- Basic: name, slug, type, type_detail, source
- Data: bonuses_json (rows editor), pieces_json (rows editor)
- Visual: icon (upload + preview), local_icon
- Effects: set_2pc, set_4pc, description

**Tech Stack:** Alpine.js 3 + Tailwind CSS 3 (zero build step)

**Security:** JWT with refresh tokens, RBAC (super-admin/editor), rate limiting

---

## 🔐 API Endpoints

Base URL: `https://sonderox.my.id/api/`
Swagger UI: `https://sonderox.my.id/api/docs`

### Public (No Auth)
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/heroes` | List all heroes (paginated) |
| GET | `/api/heroes/:id` | Get hero with tags + effectDetails |
| GET | `/api/shells` | List all shells (paginated) |
| GET | `/api/shells/:id` | Get shell by ID |
| GET | `/api/matrix` | List all matrix sets (paginated) |
| GET | `/api/matrix/:id` | Get matrix set by ID |
| GET | `/api/tiers` | Get tier data (grouped by mode) |
| GET | `/api/skill-effects` | Get all effects (grouped by type) |
| GET | `/api/skill-effects/:slug` | Get single effect by slug |
| GET | `/api/health` | Health check (disk, memory, cache) |

### Auth
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/auth/login` | Login, get JWT |
| POST | `/api/auth/change-password` | Change password |
| GET | `/api/auth/me` | Get current user |

### Admin (Auth Required)
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/heroes` | Create hero |
| PUT | `/api/heroes/:id` | Update hero |
| DELETE | `/api/heroes/:id` | Delete hero |
| POST | `/api/shells` | Create shell |
| PUT | `/api/shells/:id` | Update shell |
| DELETE | `/api/shells/:id` | Delete shell |
| POST | `/api/matrix` | Create matrix set |
| PUT | `/api/matrix/:id` | Update matrix set |
| DELETE | `/api/matrix/:id` | Delete matrix set |
| PUT | `/api/tiers` | Bulk update tiers |
| GET | `/api/admin/stats` | Dashboard statistics |
| POST | `/api/admin/backup` | Trigger DB backup |
| POST | `/api/admin/cache/purge` | Clear API cache |
| POST | `/api/admin/sync-tiers` | Sync tier data |
| POST | `/api/admin/upload/:entity` | Upload images |

---

## 🚀 Migration Strategy

### Phase 1: API + Database ✅ DONE
- ✅ Express server + SQLite (better-sqlite3, WAL mode)
- ✅ REST API endpoints: heroes, shells, matrix, teams, tiers, auth, admin
- ✅ Data imported from static JS files (93 heroes, 43 shells, 27 matrix, 5 teams)
- ✅ Static files kept as fallback
- ✅ PM2 process management
- ✅ Nginx proxy `/api/` → localhost:3001

### Phase 2: Admin Panel ✅ DONE
- ✅ Admin SPA (Alpine.js + Tailwind CDN)
- ✅ CRUD operations for heroes, shells, matrix (ALL fields editable)
- ✅ Tier editor with Kanban drag-drop (T0-T5, 6 game modes)
- ✅ JWT authentication with login/logout/refresh
- ✅ Role-based access (super-admin, editor, admin)
- ✅ Settings page (profile, change password)
- ✅ Hero tag editor (add/remove skill tags)
- ✅ Purge cache button (graceful CF missing handling)
- ✅ Image upload (multer + drag-drop UI)
- ✅ Full edit forms for all entities (heroes, shells, matrix)

### Phase 3: Frontend Switch ✅ DONE
- ✅ Frontend fetches hero/shell/matrix data from API
- ✅ Transform functions convert API format to frontend format
- ✅ Static JS files kept as fallback (instant render)
- ✅ localStorage cache with 10-minute TTL
- ✅ Loading skeleton for better UX
- ✅ Error handling in transform functions (try-catch + filter)
- ✅ Background refresh (non-blocking)
- ✅ All pages verified: Home, Characters, Tier List, Shells, Matrix, Team Builder

### Phase 4: Polish & Security ✅ DONE
- ✅ CSRF protection (HMAC-SHA256)
- ✅ XSS sanitization (`xss` package)
- ✅ Audit logging (`audit_log` table)
- ✅ gzip compression (level 6, threshold 1KB)
- ✅ API response caching (node-cache, 5min TTL)
- ✅ Query optimization (SELECT specific columns)
- ✅ Health check endpoint (`/api/health`)
- ✅ Disk monitoring (70%/80%/90% thresholds)
- ✅ PM2 logrotate (10MB max, 5 retained)
- ✅ Morgan request logging (`combined` format)
- ✅ Graceful shutdown (SIGTERM/SIGINT)
- ✅ Security headers (X-Frame, X-Content-Type, X-XSS, Referrer-Policy)
- ✅ HSTS (nginx configured, CF dashboard pending)
- ✅ Swagger UI at `/api/docs`
- ✅ Response pagination (`?page=1&limit=20`)
- ✅ Automated DB backup (daily 3AM, 7-day retention)

### Post-Phase 4: Enhancements ✅ DONE
- ✅ Skill Effects System (104 unique effects)
- ✅ Effect badges on hero skill cards
- ✅ Click-to-expand effect descriptions
- ✅ `skill_effects` table + API endpoints
- ✅ Updated data.js with effectDetails for 214 skills

---

## 📈 Progress

| Phase | Status | Completion |
|-------|--------|------------|
| Phase 1: API + Database | ✅ DONE | 100% |
| Phase 2: Admin Panel | ✅ DONE | 100% |
| Phase 3: Frontend Switch | ✅ DONE | 100% |
| Phase 4: Polish & Security | ✅ DONE | 100% |
| Post-Phase 4: Skill Effects | ✅ DONE | 100% |

**Overall: 100% complete (4/4 phases + enhancements)**

---

## 📈 Performance

| Metric | Static (Before) | API (After) | Status |
|--------|-----------------|-------------|--------|
| Hero list load | ~50ms (static) | <5ms (API, cached) | ✅ |
| Filter/search | Client-side | <10ms (indexed) | ✅ |
| Data updates | Manual JS edit | Admin panel | ✅ |
| New hero add | Edit data.js + rebuild | Admin panel | ✅ |
| Tier changes | Edit patch.js + rebuild | Admin panel | ✅ |
| Cache strategy | None | 3-layer (API/localStorage/CDN) | ✅ |
| Loading UX | None | Skeleton animation | ✅ |
| Compression | None | gzip level 6 | ✅ |
| Security | None | CSRF + XSS + audit | ✅ |

---

## 🛠️ Tech Stack

| Component | Technology | Why |
|-----------|------------|-----|
| Runtime | Node.js 20+ | Already on VPS |
| Framework | Express.js | Lightweight, fast |
| Database | SQLite (better-sqlite3) | Zero-config, fast, sync |
| Auth | JWT + bcrypt | Stateless, secure |
| Caching | node-cache + localStorage | 3-layer, no Redis needed |
| Compression | gzip (compression) | Level 6, 1KB threshold |
| Security | xss + HMAC-SHA256 CSRF | Lightweight, effective |
| Monitoring | Custom health checks | Disk, memory, cache stats |
| Process | PM2 | Production-ready, logrotate |
| Frontend | Alpine.js + Tailwind | Zero build, small bundle |
| File Upload | Multer | Multipart handling |

---

## 📁 Project Structure (Implemented)

```
/var/www/hermes-landing/
├── game-guide/                    # Frontend SPA
│   ├── index.html                 # SPA entry (?v=39)
│   ├── js/
│   │   ├── app.js                 # Main app (~3069 lines, API-first + fallback)
│   │   ├── data.js                # Hero data (323KB, with effectDetails)
│   │   ├── shells.js              # Shell data (51KB, fallback)
│   │   ├── matrix_effects.js      # Matrix data (46KB, fallback)
│   │   ├── tier_data_patch.js     # Tier fallback (12KB)
│   │   ├── role_data_patch.js     # Role data (7KB)
│   │   └── team-builder.js        # Team builder (12KB)
│   ├── css/
│   │   └── styles.css             # All styles (~2266 lines)
│   └── assets/
│       ├── characters/            # Hero images (261 webp)
│       └── elements/              # Element icons (5 webp)
│
└── game-guide-api/                # API server
    ├── src/
    │   ├── index.js               # Express server (port 3001)
    │   ├── db.js                  # SQLite + WAL mode
    │   ├── routes/
    │   │   ├── heroes.js          # CRUD + pagination + cache
    │   │   ├── shells.js          # CRUD + pagination + cache
    │   │   ├── matrix.js          # CRUD + pagination + cache
    │   │   ├── tiers.js           # GET + bulk update
    │   │   ├── auth.js            # login/change-password/me
    │   │   ├── admin.js           # stats/backup/purge/upload
    │   │   └── skill-effects.js   # GET /api/skill-effects
    │   └── middleware/
    │       ├── auth.js            # JWT + role-based access
    │       └── cache.js           # node-cache (5min TTL)
    ├── admin/                     # Admin panel SPA
    │   ├── index.html             # Alpine.js + Tailwind CDN
    │   ├── js/
    │   │   ├── app.js             # Auth, dashboard, CRUD, upload
    │   │   └── tier-editor.js     # Kanban board (SortableJS)
    │   └── css/
    │       ├── admin.css          # Dark gaming theme + modal
    │       └── tier-editor.css    # Kanban styles
    ├── data/
    │   └── gameguide.db           # SQLite database (WAL mode)
    ├── scripts/
    │   ├── backup.sh              # Automated daily backup (3AM)
    │   └── monitor.sh             # Disk monitoring (6h interval)
    ├── backups/                   # DB backups (.gz, 7-day retention)
    ├── package.json
    └── ecosystem.config.js        # PM2 config
```

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
| 8 | **Builds Section** | 🔲 Sprint 2 | Equipment recommendations per character |
| 9 | **Bulk Import/Export** | 🔲 Sprint 2 | JSON import/export for heroes/shells |
| 10 | **PWA Support** | 🔲 Future | Offline access, home screen install |

---

*Documentation updated by KangPalak — 2026-07-02*
*All 4 phases complete. Skill effects system live. Sprint 1 next.*
