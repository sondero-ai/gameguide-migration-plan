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

**Current State:**
- Static SPA at `sonderox.my.id/game-guide/`
- 93 heroes, 43 shells, 27 matrix sets, 5 team compositions
- Data stored in JS files (data.js, shells.js, matrix_effects.js, etc.)
- Vanilla JS SPA with glassmorphism dark theme
- Nginx + Cloudflare CDN on VPS

**Target State:**
- SQLite database backend
- Express.js REST API
- Admin panel for CRUD operations
- Same frontend (minimal changes)
- Automated data management

---

## 🏗️ Architecture Summary

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

## 🖥️ Admin Panel Summary

**8 Pages:**
1. Login (JWT auth)
2. Dashboard (stats, quick actions, changelog)
3. Heroes Management (CRUD, bulk ops, image upload)
4. Shells Management (CRUD, hero recommendations)
5. Matrix Management (CRUD, set effects)
6. Team Compositions (drag-and-drop builder)
7. Tier List Editor (grid: hero × mode)
8. Settings (accounts, cache, backups)

**Tech Stack:** Alpine.js 3 + Tailwind CSS 3 (zero build step)

**Security:** JWT with refresh tokens, RBAC (super-admin/editor/viewer), rate limiting, CSRF protection

---

## 🚀 Migration Strategy

### Phase 1: API + Database (Week 1)
- Set up Express server + SQLite
- Create API endpoints
- Import existing data from JS files
- Keep static files as fallback

### Phase 2: Admin Panel (Week 2)
- Build admin SPA
- Implement CRUD operations
- Add authentication
- Image upload support

### Phase 3: Frontend Switch (Week 3)
- Modify frontend to fetch from API
- Add fallback to static data
- Performance testing
- Deploy to production

### Phase 4: Polish & Security (Week 4)
- Security hardening
- Performance optimization
- Documentation
- Monitoring setup

---

## 📈 Expected Performance

| Metric | Current | Target |
|--------|---------|--------|
| Hero list load | ~50ms (static) | <5ms (API) |
| Filter/search | Client-side | <10ms (indexed) |
| Data updates | Manual JS edit | Admin panel |
| New hero add | Edit data.js + rebuild | Admin panel |
| Tier changes | Edit patch.js + rebuild | Admin panel |

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

## 📁 Project Structure (Proposed)

```
/var/www/hermes-landing/
├── game-guide/                    # Existing SPA (keep as fallback)
│   ├── index.html
│   ├── js/
│   ├── css/
│   └── assets/
│
└── game-guide-api/                # New API server
    ├── src/
    │   ├── db/
    │   │   ├── schema.sql
    │   │   ├── seed.sql
    │   │   └── migrate.js
    │   ├── routes/
    │   │   ├── heroes.js
    │   │   ├── shells.js
    │   │   ├── matrix.js
    │   │   ├── teams.js
    │   │   ├── tiers.js
    │   │   └── admin.js
    │   ├── middleware/
    │   │   ├── auth.js
    │   │   ├── validate.js
    │   │   └── errorHandler.js
    │   └── index.js
    ├── admin/                     # Admin panel SPA
    │   ├── index.html
    │   ├── js/
    │   └── css/
    ├── data/
    │   └── gameguide.db           # SQLite database
    ├── package.json
    └── ecosystem.config.js        # PM2 config
```

---

## 📝 Next Steps

1. Review all 3 documents
2. Approve architecture decisions
3. Start Phase 1 implementation
4. Set up development environment
5. Begin database migration

---

*Documentation generated by KangPalak for Sondero — 2026-07-02*
