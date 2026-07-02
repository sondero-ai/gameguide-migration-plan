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
- Frontend SPA still reads from static JS files (Phase 3 pending)

**Target State:**
- SQLite database backend ✅
- Express.js REST API ✅
- Admin panel for CRUD operations ✅
- Frontend fetches from API (Phase 3)
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

**Tier Data Flow (Live):**
```
Admin Panel → PUT /api/tiers/bulk → SQLite DB
                                         │
Browser → GET /api/tiers ←──────────────┘
         (fetch on page load)
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
3. ✅ Heroes Management (list, search, filter, edit form, tag editor)
4. ✅ Shells Management (list, search, edit form)
5. ✅ Matrix Management (list, search, edit form)
6. ✅ Tier Editor (Kanban board: T0-T5 × 6 modes, drag-drop with SortableJS)
7. ✅ Teams Management (list)
8. ✅ Settings (profile, change password, admin overview)
9. ⬜ Image Upload (not yet implemented)

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
- ✅ CRUD operations for heroes, shells, matrix
- ✅ Tier editor with Kanban drag-drop (T0-T5, 6 game modes)
- ✅ JWT authentication with login/logout/refresh
- ✅ Role-based access (super-admin, editor, admin)
- ✅ Settings page (profile, change password)
- ✅ Hero tag editor (add/remove skill tags)
- ✅ Purge cache button (graceful CF missing handling)
- ⬜ Image upload support (deferred to Phase 4)

### Phase 3: Frontend Switch ⬜ NEXT
- ⬜ Modify frontend to fetch hero/shell/matrix data from API
- ⬜ Add fallback to static data (if API unavailable)
- ⬜ Remove dependency on static JS files (data.js, shells.js, matrix_effects.js)
- ⬜ Performance testing
- ⬜ Deploy to production

### Phase 4: Polish & Security
- ⬜ Security hardening (CSRF, input sanitization, audit logging)
- ⬜ Performance optimization (API caching, query optimization)
- ⬜ Image upload support
- ⬜ Documentation
- ⬜ Monitoring setup
- ⬜ Backup automation

---

## 📈 Progress

| Phase | Status | Completion |
|-------|--------|------------|
| Phase 1: API + Database | ✅ DONE | 100% |
| Phase 2: Admin Panel | ✅ DONE | 100% |
| Phase 3: Frontend Switch | ⬜ NEXT | 0% |
| Phase 4: Polish & Security | ⬜ FUTURE | 0% |

**Overall: 50% complete (2/4 phases)**

---

## 📈 Expected Performance

| Metric | Current (Static) | Target (API) | Status |
|--------|-----------------|--------------|--------|
| Hero list load | ~50ms | <5ms | ⬜ Phase 3 |
| Filter/search | Client-side | <10ms (indexed) | ⬜ Phase 3 |
| Data updates | Manual JS edit | Admin panel | ✅ DONE |
| New hero add | Edit data.js + rebuild | Admin panel | ✅ DONE |
| Tier changes | Edit patch.js + rebuild | Admin panel | ✅ DONE |

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

---

## 📁 Project Structure (Implemented)

```
/var/www/hermes-landing/
├── game-guide/                    # Existing SPA (static fallback)
│   ├── index.html
│   ├── js/
│   │   ├── app.js                 # Main app (116KB)
│   │   ├── data.js                # Hero data (323KB)
│   │   ├── shells.js              # Shell data (51KB)
│   │   ├── matrix_effects.js      # Matrix data (46KB)
│   │   ├── tier_data_patch.js     # Tier fallback (12KB)
│   │   ├── role_data_patch.js     # Role data (7KB)
│   │   └── team-builder.js        # Team builder (12KB)
│   ├── css/
│   └── assets/
│
└── game-guide-api/                # API server
    ├── src/
    │   ├── db/
    │   │   ├── schema.sql         # 16 tables, 7 views, 5 triggers
    │   │   ├── seed.sql
    │   │   ├── connection.js      # SQLite + WAL mode
    │   │   └── migrate.js
    │   ├── routes/
    │   │   ├── heroes.js          # CRUD + tags + tiers
    │   │   ├── shells.js          # CRUD
    │   │   ├── matrix.js          # CRUD
    │   │   ├── teams.js           # CRUD
    │   │   ├── tiers.js           # GET + bulk update
    │   │   ├── auth.js            # login/refresh/logout/change-password
    │   │   └── admin.js           # stats/changelog/backup/purge-cache
    │   ├── middleware/
    │   │   ├── auth.js            # JWT + role-based access
    │   │   ├── validate.js        # Zod validation
    │   │   └── errorHandler.js
    │   └── index.js               # Express server (port 3001)
    ├── admin/                     # Admin panel SPA
    │   ├── index.html             # Alpine.js + Tailwind CDN
    │   ├── js/
    │   │   ├── app.js             # Auth, dashboard, CRUD
    │   │   └── tier-editor.js     # Kanban board (SortableJS)
    │   └── css/
    │       ├── admin.css          # Dark gaming theme
    │       └── tier-editor.css    # Kanban styles
    ├── data/
    │   └── gameguide.db           # SQLite database (WAL mode)
    ├── logs/
    ├── scripts/
    │   └── sync_tiers.sh          # DB → static file sync
    ├── package.json
    └── ecosystem.config.js        # PM2 config
```

---

## 📝 Next Steps

1. ✅ ~~Review all 3 documents~~
2. ✅ ~~Approve architecture decisions~~
3. ✅ ~~Start Phase 1 implementation~~
4. ✅ ~~Phase 2: Admin Panel~~
5. **Phase 3: Frontend Switch to API**
   - Modify `app.js` to fetch heroes/shells/matrix from API
   - Keep static files as fallback
   - Test performance
   - Deploy

---

*Documentation updated by KangPalak — 2026-07-02*
*Phase 1 & 2 complete. Phase 3 next.*
