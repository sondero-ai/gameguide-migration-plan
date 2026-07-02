# 🎮 GameGuide Migration Plan: Static → SQLite + Admin Panel

> Comprehensive technical documentation for migrating the Etheria Restart GameGuide SPA from static JSON files to a SQLite database with a full admin panel.

## 📋 Table of Contents

| Document | Description | Size |
|----------|-------------|------|
| [ARCHITECTURE.md](./ARCHITECTURE.md) | System architecture, API design, tech stack, deployment | 25KB |
| [DATABASE-SCHEMA.md](./DATABASE-SCHEMA.md) | Complete SQLite schema with 16 tables, views, triggers | 40KB |
| [ADMIN-PANEL.md](./ADMIN-PANEL.md) | Admin panel design: pages, wireframes, security, components | 75KB |

---

## 🎯 Project Overview

**Current State (2026-07-02):**
- ✅ SQLite database with 16 tables, 7 views, 5 triggers
- ✅ Express.js REST API on port 3001 (PM2 managed)
- ✅ Admin panel at `sonderox.my.id/game-guide/admin/`
- ✅ 93 heroes, 43 shells, 27 matrix sets, 5 team compositions
- ✅ Tier data served from API (real-time, no cache issues)
- ✅ JWT authentication with role-based access control
- ✅ Frontend SPA fetches ALL data from API (heroes, shells, matrix, tiers)
- ✅ localStorage cache with 10-minute TTL
- ✅ Loading skeleton for better UX
- ✅ Static JS files kept as fallback

**Target State:**
- SQLite database backend ✅
- Express.js REST API ✅
- Admin panel for CRUD operations ✅
- Frontend fetches from API ✅
- Automated data management ✅

---

## 🏗️ Architecture (Implemented)

```
┌─────────────┐     ┌─────────────┐     ┌─────────────────────┐
│   Browser   │────▶│ Cloudflare  │────▶│       Nginx         │
└─────────────┘     └─────────────┘     └──────────┬──────────┘
                                                   │
                                    ┌──────────────┴──────────────┐
                                    ▼                             ▼
                            ┌───────────────┐           ┌─────────────────┐
                            │ Static Files  │           │  Express API    │
                            │ (game-guide/) │           │  (port 3001)    │
                            └───────────────┘           └────────┬────────┘
                                                                 │
                                                                 ▼
                                                        ┌─────────────────┐
                                                        │     SQLite      │
                                                        │ (better-sqlite3)│
                                                        └─────────────────┘
```

**Data Flow (Live):**
```
Browser → GET /api/heroes → Express → SQLite → JSON response
        → GET /api/shells → Express → SQLite → JSON response
        → GET /api/matrix → Express → SQLite → JSON response
        → GET /api/tiers  → Express → SQLite → JSON response

Admin Panel → PUT /api/heroes/:id → SQLite DB
            → PUT /api/shells/:id → SQLite DB
            → PUT /api/matrix/:id → SQLite DB
            → PUT /api/tiers/bulk → SQLite DB
```

**Frontend Data Strategy:**
```
1. Static JS files load first (instant render)
2. localStorage cache checked (if valid, use cached data)
3. API fetch in background (updates cache + re-renders)
4. If API fails, static data remains as fallback
```

---

## 📊 Database Schema Summary

**16 Tables:**
- 6 Reference tables (elements, rarities, roles, game_modes, tiers, shell_rarities)
- 4 Core entities (heroes, shells, matrix_sets, team_compositions)
- 4 Junction tables (hero_tiers, hero_tags, shell_heroes, matrix_heroes)
- 2 Auth/audit tables (admin_users, changelog)

**7 Views** for common queries (denormalized hero list, tier matrix, equipment recs)

**5 Triggers** for auto-updating `updated_at` timestamps

---

## 🖥️ Admin Panel (Implemented)

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

### Phase 4: Polish & Security ⬜ NEXT
- ⬜ Security hardening (CSRF, input sanitization, audit logging)
- ⬜ Performance optimization (API caching, query optimization)
- ⬜ Documentation
- ⬜ Monitoring setup
- ⬜ Backup automation

---

## 📈 Progress

| Phase | Status | Completion |
|-------|--------|------------|
| Phase 1: API + Database | ✅ DONE | 100% |
| Phase 2: Admin Panel | ✅ DONE | 100% |
| Phase 3: Frontend Switch | ✅ DONE | 100% |
| Phase 4: Polish & Security | ⬜ NEXT | 0% |

**Overall: 75% complete (3/4 phases)**

---

## 📈 Performance

| Metric | Static (Before) | API (After) | Status |
|--------|-----------------|-------------|--------|
| Hero list load | ~50ms (static) | <5ms (API) | ✅ |
| Filter/search | Client-side | <10ms (indexed) | ✅ |
| Data updates | Manual JS edit | Admin panel | ✅ |
| New hero add | Edit data.js + rebuild | Admin panel | ✅ |
| Tier changes | Edit patch.js + rebuild | Admin panel | ✅ |
| Cache strategy | None | localStorage 10min TTL | ✅ |
| Loading UX | None | Skeleton animation | ✅ |

---

## 🛠️ Tech Stack

| Component | Technology | Why |
|-----------|------------|-----|
| Runtime | Node.js 20+ | Already on VPS |
| Framework | Express.js | Lightweight, fast |
| Database | SQLite (better-sqlite3) | Zero-config, fast, sync |
| Auth | JWT + bcrypt | Stateless, secure |
| Validation | Zod | Type-safe, fast |
| Process | PM2 | Production-ready |
| Frontend | Alpine.js + Tailwind | Zero build, small bundle |
| File Upload | Multer | Multipart handling |
| Cache | localStorage | Client-side, 10min TTL |

---

## 📁 Project Structure (Implemented)

```
/var/www/hermes-landing/
├── game-guide/                    # Frontend SPA
│   ├── index.html                 # SPA entry (?v=36)
│   ├── js/
│   │   ├── app.js                 # Main app (~120KB, API-first + fallback)
│   │   ├── data.js                # Hero data (323KB, FALLBACK)
│   │   ├── shells.js              # Shell data (51KB, FALLBACK)
│   │   ├── matrix_effects.js      # Matrix data (46KB, FALLBACK)
│   │   ├── tier_data_patch.js     # Tier fallback (12KB)
│   │   ├── role_data_patch.js     # Role data (7KB)
│   │   └── team-builder.js        # Team builder (12KB)
│   ├── css/
│   │   └── styles.css             # All styles + skeleton animations
│   └── assets/
│       ├── characters/            # Hero images (93 webp)
│       └── gear/
│           ├── shells/            # Shell icons (43 webp)
│           └── matrix/            # Matrix icons (27 webp)
│
└── game-guide-api/                # API server
    ├── src/
    │   ├── db/
    │   │   ├── schema.sql         # 16 tables, 7 views, 5 triggers
    │   │   ├── seed.sql
    │   │   └── connection.js      # SQLite + WAL mode
    │   ├── routes/
    │   │   ├── heroes.js          # CRUD + tags + tiers
    │   │   ├── shells.js          # CRUD
    │   │   ├── matrix.js          # CRUD
    │   │   ├── teams.js           # CRUD
    │   │   ├── tiers.js           # GET + bulk update
    │   │   ├── auth.js            # login/refresh/logout/change-password
    │   │   └── admin.js           # stats/changelog/backup/purge-cache/upload
    │   ├── middleware/
    │   │   ├── auth.js            # JWT + role-based access
    │   │   ├── validate.js        # Zod validation
    │   │   └── errorHandler.js
    │   └── index.js               # Express server (port 3001)
    ├── admin/                     # Admin panel SPA
    │   ├── index.html             # Alpine.js + Tailwind CDN
    │   ├── js/
    │   │   ├── app.js             # Auth, dashboard, CRUD, upload
    │   │   └── tier-editor.js     # Kanban board (SortableJS)
    │   └── css/
    │       ├── admin.css          # Dark gaming theme + modal + upload
    │       └── tier-editor.css    # Kanban styles
    ├── data/
    │   └── gameguide.db           # SQLite database (WAL mode)
    ├── logs/
    ├── uploads/                   # Temp upload directory
    ├── scripts/
    │   └── sync_tiers.sh          # DB → static file sync
    ├── package.json
    └── ecosystem.config.js        # PM2 config
```

---

## 📝 Next Steps

1. ✅ ~~Phase 1: API + Database~~
2. ✅ ~~Phase 2: Admin Panel~~
3. ✅ ~~Phase 3: Frontend Switch~~
4. **Phase 4: Polish & Security**
   - Security hardening (CSRF, input sanitization, audit logging)
   - Performance optimization (API response caching, query optimization)
   - Documentation
   - Monitoring setup
   - Backup automation

---

## 🔐 API Endpoints

### Public (No Auth)
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/heroes` | List all heroes |
| GET | `/api/heroes/:id` | Get hero with tags |
| GET | `/api/shells` | List all shells |
| GET | `/api/matrix` | List all matrix sets |
| GET | `/api/tiers` | Get tier data (grouped by mode) |

### Auth Required
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/auth/login` | Login, get JWT |
| POST | `/api/auth/refresh` | Refresh token |
| GET | `/api/auth/me` | Get current user |
| POST | `/api/auth/change-password` | Change password |

### Admin (Auth + Role)
| Method | Endpoint | Description |
|--------|----------|-------------|
| PUT | `/api/heroes/:id` | Update hero |
| POST | `/api/heroes/:id/tags` | Add tag |
| DELETE | `/api/heroes/:id/tags/:tag` | Remove tag |
| PUT | `/api/shells/:id` | Update shell |
| PUT | `/api/matrix/:id` | Update matrix |
| PUT | `/api/tiers/bulk` | Bulk update tiers |
| GET | `/api/admin/stats` | Dashboard stats |
| POST | `/api/admin/purge-cache` | Purge CF cache |
| POST | `/api/admin/upload` | Upload image |
| POST | `/api/admin/backup` | Backup database |

---

*Documentation updated by KangPalak — 2026-07-02*
*Phase 1, 2, & 3 complete. Phase 4 next.*
