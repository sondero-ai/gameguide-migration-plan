# Etheria Restart GameGuide — Admin Panel Design Document

> **Version:** 1.0  
> **Date:** 2026-07-02  
> **Status:** Design / Planning  
> **Target URL:** `https://sonderox.my.id/game-guide/admin/`  
> **Parent App:** `https://sonderox.my.id/game-guide/`

---

## Table of Contents

1. [Overview](#1-overview)
2. [Tech Stack](#2-tech-stack)
3. [Database Schema](#3-database-schema)
4. [Authentication & Security](#4-authentication--security)
5. [Page Designs & Wireframes](#5-page-designs--wireframes)
   - 5.1 [Login Page](#51-login-page)
   - 5.2 [Dashboard](#52-dashboard)
   - 5.3 [Heroes Management](#53-heroes-management)
   - 5.4 [Shells Management](#54-shells-management)
   - 5.5 [Matrix Management](#55-matrix-management)
   - 5.6 [Team Compositions](#56-team-compositions)
   - 5.7 [Tier List Editor](#57-tier-list-editor)
   - 5.8 [Settings](#58-settings)
6. [API Endpoints](#6-api-endpoints)
7. [UI/UX System](#7-uiux-system)
8. [Component Library](#8-component-library)
9. [Implementation Priorities](#9-implementation-priorities)
10. [File Structure](#10-file-structure)

---

## 1. Overview

The Admin Panel is a separate SPA that provides CRUD management for all game data displayed on the public GameGuide site. It shares the same Express API backend and SQLite database, but has its own authentication layer and UI optimized for data entry and management.

**Goals:**
- Manage all game data (heroes, shells, matrix, teams, tiers) through a visual interface
- Bulk import/export data (JSON) for rapid updates
- Image upload and management for character portraits and gear icons
- Role-based access control (super-admin, editor, viewer)
- Match the existing glassmorphism dark theme of the main site

**Non-goals (v1):**
- Public-facing user accounts
- Real-time collaboration
- CMS for blog/guide content

---

## 2. Tech Stack

| Layer | Technology | Rationale |
|-------|-----------|-----------|
| **Frontend** | Alpine.js 3 + Tailwind CSS 3 | Minimal bundle (~45KB), reactive without build step, pairs with existing Tailwind |
| **Routing** | Alpine.js + History API | Consistent with main SPA approach |
| **State** | Alpine.js stores | Simple global state for auth, data cache |
| **Backend** | Express.js 4 | Same API server as main site |
| **Database** | SQLite (better-sqlite3) | Zero-config, single-file, fast reads |
| **Auth** | JWT (access + refresh tokens) | Stateless, secure with httpOnly cookies |
| **Image Processing** | Sharp | Resize/convert uploads to WebP |
| **File Upload** | Multer | Multipart form handling |
| **Deployment** | Nginx reverse proxy | Same server, `/api/admin/*` routes |

### Why Alpine.js over Vanilla JS

The main site uses vanilla JS, but the admin panel needs:
- Reactive data binding for forms and tables
- Component-like patterns for modals, toasts, dropdowns
- Less boilerplate for CRUD operations

Alpine.js provides this at **15KB gzipped** with zero build step — drop in a `<script>` tag and go. It's the sweet spot between vanilla JS verbosity and React/Vue overhead.

### Bundle Budget

| Asset | Size (gzipped) |
|-------|---------------|
| Alpine.js 3 | ~15 KB |
| Tailwind CSS (CDN play) | ~30 KB |
| Admin app.js | ~25 KB (estimate) |
| Admin styles.css | ~5 KB (custom) |
| **Total** | **~75 KB** |

---

## 3. Database Schema

### Tables

```sql
-- Admin users
CREATE TABLE admin_users (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  username TEXT UNIQUE NOT NULL,
  password_hash TEXT NOT NULL,
  role TEXT NOT NULL DEFAULT 'viewer',  -- super-admin | editor | viewer
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  last_login DATETIME
);

-- Heroes (characters)
CREATE TABLE heroes (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  slug TEXT UNIQUE NOT NULL,
  name TEXT NOT NULL,
  rarity TEXT NOT NULL,           -- SSR | SR | R
  element TEXT NOT NULL,          -- Reason | Odd | Hollow | Constant | Disorder
  role TEXT NOT NULL,             -- DPS | Support | Buffer | Healer | Tank
  tags TEXT DEFAULT '[]',         -- JSON array of tags
  avatar_card TEXT,               -- path to 128x128 WebP
  avatar_detail TEXT,             -- path to 256x256 WebP
  skills TEXT DEFAULT '[]',       -- JSON array
  stats TEXT DEFAULT '{}',        -- JSON object
  description TEXT,
  is_published BOOLEAN DEFAULT 1,
  sort_order INTEGER DEFAULT 0,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Hero tier assignments (per game mode)
CREATE TABLE hero_tiers (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  hero_id INTEGER NOT NULL,
  game_mode TEXT NOT NULL,        -- pvp | pve | boss_raid | story
  tier TEXT NOT NULL,             -- SS | S | A | B | C | D
  notes TEXT,
  FOREIGN KEY (hero_id) REFERENCES heroes(id) ON DELETE CASCADE,
  UNIQUE(hero_id, game_mode)
);

-- Shells (equipment)
CREATE TABLE shells (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  slug TEXT UNIQUE NOT NULL,
  name TEXT NOT NULL,
  rarity TEXT NOT NULL,           -- Mythic | Unique | Epic | Rare
  slot TEXT NOT NULL,             -- weapon | armor | accessory
  set_name TEXT,
  icon TEXT,                      -- path to WebP
  bonuses TEXT DEFAULT '[]',      -- JSON array
  sub_stats TEXT DEFAULT '[]',    -- JSON array
  description TEXT,
  recommended_heroes TEXT DEFAULT '[]',  -- JSON array of hero IDs
  is_published BOOLEAN DEFAULT 1,
  sort_order INTEGER DEFAULT 0,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Matrix sets (equipment sets)
CREATE TABLE matrix_sets (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  slug TEXT UNIQUE NOT NULL,
  name TEXT NOT NULL,
  type TEXT NOT NULL,             -- ATK | DEF | HP | SPD | Special
  source TEXT,                    -- where to obtain
  icon TEXT,                      -- path to WebP
  pieces TEXT DEFAULT '[]',       -- JSON array of piece names
  bonuses TEXT DEFAULT '[]',      -- JSON array of set bonuses
  effect TEXT,
  effect_awakened TEXT,
  skill_tags TEXT DEFAULT '[]',   -- JSON array
  recommended_heroes TEXT DEFAULT '[]',  -- JSON array of hero IDs
  is_published BOOLEAN DEFAULT 1,
  sort_order INTEGER DEFAULT 0,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Team compositions
CREATE TABLE teams (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  name TEXT NOT NULL,
  description TEXT,
  game_mode TEXT,                 -- pvp | pve | boss_raid | story
  hero_ids TEXT NOT NULL,         -- JSON array [1, 5, 12, 33]
  synergy_notes TEXT,
  is_published BOOLEAN DEFAULT 1,
  sort_order INTEGER DEFAULT 0,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Changelog / audit log
CREATE TABLE changelog (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  admin_user_id INTEGER,
  action TEXT NOT NULL,           -- create | update | delete | import
  resource_type TEXT NOT NULL,    -- hero | shell | matrix | team | tier
  resource_id INTEGER,
  details TEXT,                   -- JSON diff or description
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (admin_user_id) REFERENCES admin_users(id)
);

-- Settings (key-value store)
CREATE TABLE settings (
  key TEXT PRIMARY KEY,
  value TEXT,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

### Seed Data

The existing 93 heroes, 43 shells, and 27 matrix sets from the current JS data files will be imported via the bulk import endpoint on first setup.

---

## 4. Authentication & Security

### Auth Flow

```
┌─────────┐    POST /api/admin/login     ┌──────────┐
│  Admin   │ ──────────────────────────▶  │  Express  │
│  Browser │                              │  Server   │
│          │  ◀──────────────────────────  │           │
│          │  Set-Cookie: access_token    │           │
│          │  Set-Cookie: refresh_token   │           │
└─────────┘                               └──────────┘
     │                                         │
     │  GET /api/admin/heroes                  │
     │  Cookie: access_token=eyJ...            │
     │ ─────────────────────────────────────▶  │
     │                                         │
     │  401 Unauthorized (token expired)       │
     │ ◀─────────────────────────────────────  │
     │                                         │
     │  POST /api/admin/refresh                │
     │  Cookie: refresh_token=eyJ...           │
     │ ─────────────────────────────────────▶  │
     │                                         │
     │  200 OK + new access_token              │
     │ ◀─────────────────────────────────────  │
```

### Token Configuration

| Token | Lifetime | Storage | Purpose |
|-------|----------|---------|---------|
| Access | 15 minutes | httpOnly cookie, Secure, SameSite=Strict | API authentication |
| Refresh | 7 days | httpOnly cookie, Secure, SameSite=Strict | Renew access token |

### Security Measures

1. **Password Hashing:** bcrypt with 12 rounds
2. **Rate Limiting:** 5 login attempts per 15 minutes per IP (express-rate-limit)
3. **CSRF Protection:** Double-submit cookie pattern for state-changing requests
4. **Input Sanitization:** express-validator on all inputs, parameterized SQL queries
5. **Role-Based Access Control:**
   - `super-admin`: Full access, manage users, settings, backup/restore
   - `editor`: CRUD on all game data, no user/settings management
   - `viewer`: Read-only access to admin panel

### Middleware Stack

```
Request → Rate Limiter → Cookie Parser → JWT Verify → Role Check → Route Handler → Response
```

---

## 5. Page Designs & Wireframes

### Design Tokens (shared with main site)

```css
:root {
  --bg-primary: #0a0a1a;
  --bg-card: rgba(15, 15, 35, 0.8);
  --bg-card-hover: rgba(25, 25, 55, 0.9);
  --glass: rgba(255, 255, 255, 0.05);
  --glass-border: rgba(255, 255, 255, 0.1);
  --text-primary: #e2e8f0;
  --text-secondary: #94a3b8;
  --text-muted: #64748b;
  --accent: #818cf8;       /* indigo-400 */
  --accent-bright: #a5b4fc; /* indigo-300 */
  --success: #34d399;
  --warning: #fbbf24;
  --danger: #f87171;
  --info: #60a5fa;
}
```

---

### 5.1 Login Page

**URL:** `/game-guide/admin/login`

**Layout:**
```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│                    ┌──────────────────┐                     │
│                    │   🎮 GameGuide   │                     │
│                    │    Admin Panel   │                     │
│                    │                  │                     │
│                    │  ┌────────────┐  │                     │
│                    │  │ Username   │  │                     │
│                    │  └────────────┘  │                     │
│                    │  ┌────────────┐  │                     │
│                    │  │ Password   │  │                     │
│                    │  └────────────┘  │                     │
│                    │                  │                     │
│                    │  [ Login    ]    │                     │
│                    │                  │                     │
│                    │  ⚠ Error msg    │                     │
│                    └──────────────────┘                     │
│                                                             │
│              Glassmorphism card, centered vertically         │
│              Background: same as main site                   │
└─────────────────────────────────────────────────────────────┘
```

**Components:**
- Centered glassmorphism card (max-width: 400px)
- Username input with icon
- Password input with show/hide toggle
- Login button with loading spinner
- Error toast for failed attempts
- Rate limit warning after 3 failed attempts

**Behavior:**
- Enter key submits form
- Auto-focus username field on load
- Redirect to `/admin/` on success
- Show remaining attempts on rate limit

---

### 5.2 Dashboard

**URL:** `/game-guide/admin/` (default after login)

**Layout:**
```
┌─────────────────────────────────────────────────────────────────┐
│ ┌──────────┐                                                    │
│ │ SIDEBAR  │  Dashboard                                    [👤] │
│ │          │────────────────────────────────────────────────────│
│ │ 📊 Dash  │                                                    │
│ │ 🦸 Heroes│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────┐ │
│ │ 🛡️ Shells│  │  93      │ │  43      │ │  27      │ │  12  │ │
│ │ 🔷 Matrix│  │  Heroes  │ │  Shells  │ │  Matrix  │ │Teams │ │
│ │ 👥 Teams │  └──────────┘ └──────────┘ └──────────┘ └──────┘ │
│ │ ⭐ Tiers │                                                    │
│ │ ⚙️ Sett. │  Quick Actions                                     │
│ │          │  ┌──────────┐ ┌──────────┐ ┌──────────────────┐   │
│ │          │  │+ Add Hero│ │+ Add Shell│ │ 📥 Import JSON   │   │
│ │          │  └──────────┘ └──────────┘ └──────────────────┘   │
│ │          │                                                    │
│ │          │  Recent Changes                                    │
│ │          │  ┌──────────────────────────────────────────────┐  │
│ │          │  │ 2min ago  │ hero  │ update  │ Blade         │  │
│ │          │  │ 1hr ago   │ shell │ create  │ Ace Batter    │  │
│ │          │  │ 3hr ago   │ team  │ delete  │ PvP Rush      │  │
│ │          │  │ Yesterday │ hero  │ import  │ 15 heroes     │  │
│ │          │  └──────────────────────────────────────────────┘  │
│ └──────────┘                                                    │
└─────────────────────────────────────────────────────────────────┘
```

**Components:**
- **Stats Cards (4):** Glassmorphism cards with icon, count, label, and +X this week indicator
- **Quick Actions (3 buttons):** Add Hero, Add Shell, Import JSON — each opens the respective modal/page
- **Recent Changes Table:** Last 20 entries from `changelog` table, with timestamp, resource type badge, action badge, and description
- **Activity Chart (optional v2):** Simple bar chart of changes per day (last 7 days)

**Stats Card Design:**
```
┌─────────────────────────┐
│  🦸  │  93              │
│      │  Heroes          │
│      │  +3 this week    │
└─────────────────────────┘
- Background: var(--bg-card)
- Border: 1px solid var(--glass-border)
- Border-radius: 12px
- Hover: slight glow effect
```

**Changelog Badge Colors:**
- `create` → green (#34d399)
- `update` → blue (#60a5fa)
- `delete` → red (#f87171)
- `import` → purple (#a78bfa)

---

### 5.3 Heroes Management

**URL:** `/game-guide/admin/heroes`

**Layout:**
```
┌─────────────────────────────────────────────────────────────────┐
│ SIDEBAR │  Heroes Management (93 total)                   [👤]  │
│         │──────────────────────────────────────────────────────│
│         │                                                      │
│         │  [🔍 Search heroes...]  [Rarity ▾] [Role ▾] [Elem ▾]│
│         │  [Element ▾]  [Tag ▾]  [Published ▾]                 │
│         │                                                      │
│         │  [+ Add Hero]  [📥 Import]  [📤 Export]  [🗑 Bulk Del]│
│         │                                                      │
│         │  ┌─┬──────┬──────┬──────┬──────┬──────┬──────┬─────┐│
│         │  │☐│ Icon │ Name │Rarity│ Role │Elem  │Tier  │ Act ││
│         │  ├─┼──────┼──────┼──────┼──────┼──────┼──────┼─────┤│
│         │  │☐│ [🖼] │Blade │ SSR  │ DPS  │Const.│ SS   │✏️🗑 ││
│         │  │☐│ [🖼] │Celin │ SSR  │ Supp │Odd   │ S    │✏️🗑 ││
│         │  │☐│ [🖼] │Feydis│ SSR  │ Buff │Const.│ SS   │✏️🗑 ││
│         │  │☐│ [🖼] │...   │ ...  │ ...  │ ...  │ ...  │✏️🗑 ││
│         │  └─┴──────┴──────┴──────┴──────┴──────┴──────┴─────┘│
│         │                                                      │
│         │  ◀ 1 2 3 4 5 ... 10 ▶        Showing 1-20 of 93    │
│         │                                                      │
└─────────────────────────────────────────────────────────────────┘
```

**Table Columns:**
| Column | Width | Sortable | Description |
|--------|-------|----------|-------------|
| ☐ (checkbox) | 40px | No | Bulk select |
| Icon | 48px | No | Avatar thumbnail (32x32) |
| Name | 150px | Yes | Character name |
| Rarity | 80px | Yes | SSR/SR/R badge |
| Role | 100px | Yes | DPS/Support/Buffer/Healer/Tank |
| Element | 100px | Yes | Element with color icon |
| Tier (PvP) | 80px | Yes | SS/S/A/B/C/D |
| Published | 80px | Yes | Toggle switch |
| Actions | 100px | No | Edit / Delete buttons |

**Filters:**
- **Search:** Fuzzy match on name, slug, tags
- **Rarity:** Dropdown (All, SSR, SR, R)
- **Role:** Dropdown (All, DPS, Support, Buffer, Healer, Tank)
- **Element:** Dropdown (All, Reason, Odd, Hollow, Constant, Disorder)
- **Published:** Dropdown (All, Published, Draft)

**Toolbar Actions:**
- **+ Add Hero:** Opens empty edit modal
- **📥 Import:** Opens JSON file picker, validates, shows preview, confirms
- **📤 Export:** Downloads all heroes as JSON
- **🗑 Bulk Delete:** Deletes all selected heroes (with confirmation)

#### Hero Edit Modal

```
┌─────────────────────────────────────────────────────────────────┐
│  Edit Hero: Blade                                         [✕]  │
│─────────────────────────────────────────────────────────────────│
│                                                                 │
│  ┌─────────────┐   Name:    [Blade                    ]        │
│  │             │   Slug:    [blade                     ]        │
│  │   AVATAR    │   Rarity:  [SSR ▾]                            │
│  │   PREVIEW   │   Element: [Constant ▾]                        │
│  │             │   Role:    [DPS ▾]                              │
│  │  [Upload]   │   Tags:    [dps, melee, burst    ] [+]        │
│  └─────────────┘                                                │
│                                                                 │
│  ── Tiers ──────────────────────────────────────────────────   │
│  PvP:       [SS ▾]  PvE:       [S  ▾]                          │
│  Boss Raid: [SS ▾]  Story:     [A  ▾]                          │
│                                                                 │
│  ── Description ────────────────────────────────────────────   │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ Blade is a powerful DPS character with high burst...     │  │
│  │                                                          │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  Published: [●────] (toggle)                                    │
│                                                                 │
│  ── Advanced (collapsible) ─────────────────────────────────   │
│  ▶ Skills (JSON editor)                                         │
│  ▶ Stats (JSON editor)                                          │
│  ▶ Raw JSON                                                     │
│                                                                 │
│                                    [Cancel]  [Save Hero]       │
└─────────────────────────────────────────────────────────────────┘
```

**Modal Features:**
- **Image Upload:** Drag-and-drop zone + click to upload. Shows preview. Auto-resizes to 128x128 (card) and 256x256 (detail) WebP on upload.
- **Tag Input:** Type-ahead with existing tags, press Enter/comma to add, click × to remove
- **Tier Dropdowns:** Per game mode, with color-coded badges (SS=gold, S=purple, A=blue, B=green, C=gray, D=red)
- **JSON Editors:** Collapsible sections for skills/stats with syntax-highlighted textarea and validation
- **Validation:** Real-time validation (name required, slug uniqueness, etc.)
- **Dirty State:** Warns on close if unsaved changes

**Keyboard Shortcuts (in modal):**
- `Ctrl+Enter` → Save
- `Escape` → Cancel
- `Ctrl+Shift+J` → Toggle raw JSON view

---

### 5.4 Shells Management

**URL:** `/game-guide/admin/shells`

**Layout:**
```
┌─────────────────────────────────────────────────────────────────┐
│ SIDEBAR │  Shells Management (43 total)                   [👤]  │
│         │──────────────────────────────────────────────────────│
│         │                                                      │
│         │  [🔍 Search shells...]  [Rarity ▾] [Slot ▾] [Set ▾] │
│         │                                                      │
│         │  [+ Add Shell]  [📥 Import]  [📤 Export]              │
│         │                                                      │
│         │  ┌─┬──────┬────────┬──────┬──────┬──────┬──────┬───┐ │
│         │  │☐│ Icon │ Name   │Rarity│ Slot │ Set  │Heroes│Act│ │
│         │  ├─┼──────┼────────┼──────┼──────┼──────┼──────┼───┤ │
│         │  │☐│ [🖼] │Ace Batt│Mythic│ Weap │Swift │ 3    │✏️🗑│ │
│         │  │☐│ [🖼] │Guardian│Uniq  │ Armor│Guard │ 5    │✏️🗑│ │
│         │  │☐│ [🖼] │...     │ ...  │ ...  │ ...  │ ...  │✏️🗑│ │
│         │  └─┴──────┴────────┴──────┴──────┴──────┴──────┴───┘ │
│         │                                                      │
│         │  ◀ 1 2 3 ▶                      Showing 1-20 of 43  │
│         │                                                      │
└─────────────────────────────────────────────────────────────────┘
```

**Filters:**
- **Rarity:** All, Mythic, Unique, Epic, Rare
- **Slot:** All, Weapon, Armor, Accessory
- **Set:** Dropdown of all set names

#### Shell Edit Modal

```
┌─────────────────────────────────────────────────────────────────┐
│  Edit Shell: Ace Batter                                   [✕]  │
│─────────────────────────────────────────────────────────────────│
│                                                                 │
│  ┌─────────────┐   Name:    [Ace Batter                 ]      │
│  │             │   Slug:    [ace-batter                  ]      │
│  │   ICON      │   Rarity:  [Mythic ▾]                          │
│  │   PREVIEW   │   Slot:    [Weapon ▾]                          │
│  │             │   Set:     [Swift Smite ▾]                      │
│  │  [Upload]   │                                                │
│  └─────────────┘                                                │
│                                                                 │
│  ── Bonuses ───────────────────────────────────────────────    │
│  [+ Add Bonus]                                                  │
│  1. [ATK +15%          ]  [✕]                                   │
│  2. [Crit Rate +10%    ]  [✕]                                   │
│                                                                 │
│  ── Sub Stats ─────────────────────────────────────────────    │
│  [+ Add Sub Stat]                                               │
│  1. [HP +500           ]  [✕]                                   │
│  2. [DEF +200          ]  [✕]                                   │
│                                                                 │
│  ── Recommended Heroes ────────────────────────────────────    │
│  [🔍 Search heroes to add...]                                   │
│  ┌────────┐ ┌────────┐ ┌────────┐                              │
│  │ Blade ✕│ │ Celin ✕│ │ + Add  │                              │
│  └────────┘ └────────┘ └────────┘                              │
│                                                                 │
│  Description:                                                   │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ A powerful weapon favored by...                          │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  Published: [●────]                                             │
│                                                                 │
│                                    [Cancel]  [Save Shell]       │
└─────────────────────────────────────────────────────────────────┘
```

---

### 5.5 Matrix Management

**URL:** `/game-guide/admin/matrix`

**Layout:**
```
┌─────────────────────────────────────────────────────────────────┐
│ SIDEBAR │  Matrix Sets (27 total)                         [👤]  │
│         │──────────────────────────────────────────────────────│
│         │                                                      │
│         │  [🔍 Search matrix...]  [Type ▾] [Source ▾]          │
│         │                                                      │
│         │  [+ Add Matrix]  [📥 Import]  [📤 Export]             │
│         │                                                      │
│         │  ┌─┬──────┬────────┬──────┬────────┬──────┬──────┐   │
│         │  │☐│ Icon │ Name   │ Type │ Source │Pieces│Heroes│   │
│         │  ├─┼──────┼────────┼──────┼────────┼──────┼──────┤   │
│         │  │☐│ [🖼] │Swift Sm│ ATK  │ Boss   │ 4    │ 5    │   │
│         │  │☐│ [🖼] │Guardian│ DEF  │ Story  │ 4    │ 3    │   │
│         │  │☐│ [🖼] │...     │ ...  │ ...    │ ...  │ ...  │   │
│         │  └─┴──────┴────────┴──────┴────────┴──────┴──────┘   │
│         │                                                      │
└─────────────────────────────────────────────────────────────────┘
```

**Filters:**
- **Type:** All, ATK, DEF, HP, SPD, Special
- **Source:** All, Boss, Story, Event, Shop

#### Matrix Edit Modal

```
┌─────────────────────────────────────────────────────────────────┐
│  Edit Matrix: Swift Smite                                 [✕]  │
│─────────────────────────────────────────────────────────────────│
│                                                                 │
│  ┌─────────────┐   Name:    [Swift Smite                 ]     │
│  │             │   Slug:    [swift-smite                  ]     │
│  │   ICON      │   Type:    [ATK ▾]  (color-coded badge)       │
│  │   PREVIEW   │   Source:  [Boss Raid ▾]                       │
│  │             │                                                │
│  │  [Upload]   │                                                │
│  └─────────────┘                                                │
│                                                                 │
│  ── Pieces ────────────────────────────────────────────────    │
│  [+ Add Piece]                                                  │
│  1. [Swift Blade      ]  [✕]                                   │
│  2. [Swift Armor      ]  [✕]                                   │
│  3. [Swift Ring       ]  [✕]                                   │
│  4. [Swift Amulet     ]  [✕]                                   │
│                                                                 │
│  ── Set Bonuses ───────────────────────────────────────────    │
│  [+ Add Bonus]                                                  │
│  2-piece: [ATK +10%                                       ]     │
│  4-piece: [After crit, gain SPD +20% for 2 turns         ]     │
│                                                                 │
│  ── Effects ───────────────────────────────────────────────    │
│  Effect:                                                        │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ Increases ATK by 10%...                                  │  │
│  └──────────────────────────────────────────────────────────┘  │
│  Awakened Effect:                                               │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ Increases ATK by 15%, and additionally...                │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ── Skill Tags ────────────────────────────────────────────    │
│  [damage] [speed] [critical] [+ Add]                            │
│                                                                 │
│  ── Recommended Heroes ────────────────────────────────────    │
│  [🔍 Search heroes...]                                          │
│  ┌────────┐ ┌────────┐ ┌────────┐                              │
│  │ Blade ✕│ │ Feydis │ │ + Add  │                              │
│  └────────┘ └────────┘ └────────┘                              │
│                                                                 │
│  Published: [●────]                                             │
│                                    [Cancel]  [Save Matrix]      │
└─────────────────────────────────────────────────────────────────┘
```

---

### 5.6 Team Compositions

**URL:** `/game-guide/admin/teams`

**Layout:**
```
┌─────────────────────────────────────────────────────────────────┐
│ SIDEBAR │  Team Compositions (12 total)                  [👤]   │
│         │──────────────────────────────────────────────────────│
│         │                                                      │
│         │  [🔍 Search teams...]  [Mode ▾]                       │
│         │                                                      │
│         │  [+ Add Team]  [📥 Import]  [📤 Export]               │
│         │                                                      │
│         │  ┌────────────────────────────────────────────────┐  │
│         │  │  PvP Burst Rush                    [PvP] [✏️🗑] │  │
│         │  │  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐          │  │
│         │  │  │ [🖼] │ │ [🖼] │ │ [🖼] │ │ [🖼] │          │  │
│         │  │  │Blade │ │Celin │ │Feydis│ │Tian  │          │  │
│         │  │  └──────┘ └──────┘ └──────┘ └──────┘          │  │
│         │  │  High burst damage team for PvP arena...       │  │
│         │  └────────────────────────────────────────────────┘  │
│         │                                                      │
│         │  ┌────────────────────────────────────────────────┐  │
│         │  │  Boss Raid Sustain                  [Boss] [✏️🗑]│  │
│         │  │  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐          │  │
│         │  │  │ [🖼] │ │ [🖼] │ │ [🖼] │ │ [🖼] │          │  │
│         │  │  │Healer│ │Tank  │ │Buffer│ │ DPS  │          │  │
│         │  │  └──────┘ └──────┘ └──────┘ └──────┘          │  │
│         │  │  Sustain-focused team for boss encounters...   │  │
│         │  └────────────────────────────────────────────────┘  │
│         │                                                      │
└─────────────────────────────────────────────────────────────────┘
```

#### Team Edit Modal

```
┌─────────────────────────────────────────────────────────────────┐
│  Edit Team: PvP Burst Rush                                [✕]  │
│─────────────────────────────────────────────────────────────────│
│                                                                 │
│  Team Name: [PvP Burst Rush                              ]     │
│  Game Mode: [PvP ▾]                                             │
│                                                                 │
│  ── Team Slots (drag to reorder) ──────────────────────────    │
│                                                                 │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐          │
│  │  Slot 1  │ │  Slot 2  │ │  Slot 3  │ │  Slot 4  │          │
│  │          │ │          │ │          │ │          │          │
│  │  [🖼]    │ │  [🖼]    │ │  [🖼]    │ │  [🖼]    │          │
│  │  Blade   │ │  Celin   │ │  Feydis  │ │  [Empty] │          │
│  │  DPS     │ │  Support │ │  Buffer  │ │          │          │
│  │    [✕]   │ │    [✕]   │ │    [✕]   │ │          │          │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘          │
│                                                                 │
│  ── Add Hero ──────────────────────────────────────────────    │
│  [🔍 Search and add hero...]                                    │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ [🖼] Tian        DPS    Disorder   [Add to Slot 4]       │  │
│  │ [🖼] Epsilon     DPS    Constant   [Add to Slot 4]       │  │
│  │ [🖼] Verde       Buffer Odd        [Add to Slot 4]       │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ── Synergy Preview ───────────────────────────────────────    │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ Roles: 2 DPS, 1 Support, 1 Buffer                       │  │
│  │ Elements: 2 Constant, 1 Odd                              │  │
│  │ Synergy Score: 85/100                                    │  │
│  │ ⚡ No healer — may struggle in long fights               │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  Synergy Notes:                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ High burst combo: Blade + Celin synergy for first-turn   │  │
│  │ kill potential...                                        │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  Published: [●────]                                             │
│                                    [Cancel]  [Save Team]        │
└─────────────────────────────────────────────────────────────────┘
```

**Drag-and-Drop:** Use HTML5 Drag and Drop API or a lightweight library like SortableJS (~10KB) for reordering team slots.

**Synergy Preview:** Real-time calculation based on:
- Role distribution (DPS/Support/Buffer/Healer/Tank balance)
- Element diversity
- Known synergies (from hero data)
- Warnings (e.g., no healer, all same element)

---

### 5.7 Tier List Editor

**URL:** `/game-guide/admin/tiers`

**Layout:**
```
┌─────────────────────────────────────────────────────────────────┐
│ SIDEBAR │  Tier List Editor                               [👤]  │
│         │──────────────────────────────────────────────────────│
│         │                                                      │
│         │  Game Mode: [PvP ▾]  [PvE ▾]  [Boss Raid ▾]  [All]  │
│         │                                                      │
│         │  [📥 Import Tiers]  [📤 Export Tiers]  [Bulk Edit]    │
│         │                                                      │
│         │  ┌──────────────────────────────────────────────────┐│
│         │  │  SS Tier                                        ││
│         │  │  ┌────┐ ┌────┐ ┌────┐                           ││
│         │  │  │[🖼]│ │[🖼]│ │[🖼]│                           ││
│         │  │  │Blade│ │Fey.│ │Tian│                           ││
│         │  │  └────┘ └────┘ └────┘                           ││
│         │  ├──────────────────────────────────────────────────┤│
│         │  │  S Tier                                         ││
│         │  │  ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐            ││
│         │  │  │[🖼]│ │[🖼]│ │[🖼]│ │[🖼]│ │[🖼]│            ││
│         │  │  │Cel.│ │Eps.│ │Ver.│ │Aur.│ │Lyn.│            ││
│         │  │  └────┘ └────┘ └────┘ └────┘ └────┘            ││
│         │  ├──────────────────────────────────────────────────┤│
│         │  │  A Tier                                         ││
│         │  │  ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐     ││
│         │  │  │[🖼]│ │[🖼]│ │[🖼]│ │[🖼]│ │[🖼]│ │[🖼]│     ││
│         │  │  │... │ │... │ │... │ │... │ │... │ │... │     ││
│         │  │  └────┘ └────┘ └────┘ └────┘ └────┘ └────┘     ││
│         │  ├──────────────────────────────────────────────────┤│
│         │  │  B Tier ...                                     ││
│         │  ├──────────────────────────────────────────────────┤│
│         │  │  C Tier ...                                     ││
│         │  └──────────────────────────────────────────────────┘│
│         │                                                      │
└─────────────────────────────────────────────────────────────────┘
```

**Features:**
- **Grid View:** Heroes displayed as cards grouped by tier (SS, S, A, B, C, D)
- **Drag-and-Drop:** Drag hero cards between tier rows to reassign
- **Mode Tabs:** Switch between PvP, PvE, Boss Raid, Story modes
- **Bulk Edit:** Select multiple heroes, assign tier from dropdown
- **Quick Assign:** Right-click hero → context menu with tier options
- **Visual Indicators:**
  - Tier row headers with color coding (SS=gold gradient, S=purple, A=blue, B=green, C=gray, D=red)
  - Hero cards show name + small portrait
  - Recently changed heroes get a subtle pulse animation

**Tier Color Map:**
| Tier | Background | Text |
|------|-----------|------|
| SS | linear-gradient(135deg, #fbbf24, #f59e0b) | #000 |
| S | linear-gradient(135deg, #a78bfa, #7c3aed) | #fff |
| A | linear-gradient(135deg, #60a5fa, #3b82f6) | #fff |
| B | linear-gradient(135deg, #34d399, #10b981) | #000 |
| C | linear-gradient(135deg, #94a3b8, #64748b) | #fff |
| D | linear-gradient(135deg, #f87171, #ef4444) | #fff |

---

### 5.8 Settings

**URL:** `/game-guide/admin/settings`

**Layout:**
```
┌─────────────────────────────────────────────────────────────────┐
│ SIDEBAR │  Settings                                         [👤]│
│         │──────────────────────────────────────────────────────│
│         │                                                      │
│         │  ┌─ Account ──────────────────────────────────────┐  │
│         │  │  Change Password                                │  │
│         │  │  Current: [••••••••]                            │  │
│         │  │  New:     [••••••••]                            │  │
│         │  │  Confirm: [••••••••]                            │  │
│         │  │                          [Update Password]      │  │
│         │  └────────────────────────────────────────────────┘  │
│         │                                                      │
│         │  ┌─ Admin Users (super-admin only) ───────────────┐  │
│         │  │  [+ Add User]                                   │  │
│         │  │  ┌──────────┬──────┬──────────┬──────┐          │  │
│         │  │  │ Username │ Role │ Last Login│ Act  │          │  │
│         │  │  ├──────────┼──────┼──────────┼──────┤          │  │
│         │  │  │ admin    │ super│ 2hr ago  │  ✏️🗑│          │  │
│         │  │  │ editor1  │ edit │ Yesterday│  ✏️🗑│          │  │
│         │  │  │ viewer1  │ view │ Never    │  ✏️🗑│          │  │
│         │  │  └──────────┴──────┴──────────┴──────┘          │  │
│         │  └────────────────────────────────────────────────┘  │
│         │                                                      │
│         │  ┌─ Cache Control ────────────────────────────────┐  │
│         │  │  Cloudflare Cache:  [Purge All Cache]          │  │
│         │  │  API Response Cache: [Clear]                    │  │
│         │  │  Last purged: 2 hours ago                       │  │
│         │  └────────────────────────────────────────────────┘  │
│         │                                                      │
│         │  ┌─ Database ─────────────────────────────────────┐  │
│         │  │  [📥 Backup Database]  [📤 Restore Database]    │  │
│         │  │  Last backup: 2026-07-01 15:30                  │  │
│         │  │  Size: 2.4 MB                                   │  │
│         │  │                                                 │  │
│         │  │  ⚠️ Restore will overwrite current data         │  │
│         │  └────────────────────────────────────────────────┘  │
│         │                                                      │
│         │  ┌─ API Keys ─────────────────────────────────────┐  │
│         │  │  Public API Key: [sk_pub_****...****] [Copy]    │  │
│         │  │  [Regenerate Key]                               │  │
│         │  └────────────────────────────────────────────────┘  │
│         │                                                      │
│         │  ┌─ System Info ──────────────────────────────────┐  │
│         │  │  Version: 1.0.0                                 │  │
│         │  │  Heroes: 93  │  Shells: 43  │  Matrix: 27      │  │
│         │  │  Database: 2.4 MB                               │  │
│         │  │  Node.js: v20.x  │  SQLite: 3.x               │  │
│         │  └────────────────────────────────────────────────┘  │
│         │                                                      │
└─────────────────────────────────────────────────────────────────┘
```

---

## 6. API Endpoints

### Base URL: `/api/admin`

All endpoints require JWT authentication (except login). Responses use standard JSON format:

```json
{
  "success": true,
  "data": { ... },
  "meta": { "total": 93, "page": 1, "limit": 20 }
}
```

### Authentication

| Method | Endpoint | Description | Body |
|--------|----------|-------------|------|
| POST | `/login` | Authenticate admin | `{ username, password }` |
| POST | `/refresh` | Refresh access token | (cookie) |
| POST | `/logout` | Clear tokens | (cookie) |

### Heroes

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | `/heroes` | List all heroes (paginated, filterable) | viewer+ |
| GET | `/heroes/:id` | Get single hero | viewer+ |
| POST | `/heroes` | Create hero | editor+ |
| PUT | `/heroes/:id` | Update hero | editor+ |
| DELETE | `/heroes/:id` | Delete hero | editor+ |
| POST | `/heroes/bulk` | Bulk import heroes (JSON array) | editor+ |
| GET | `/heroes/export` | Export all heroes as JSON | viewer+ |
| POST | `/heroes/:id/image` | Upload hero image | editor+ |

**Query Parameters for GET /heroes:**
- `page` (default: 1)
- `limit` (default: 20, max: 100)
- `search` (fuzzy match on name/slug/tags)
- `rarity` (SSR|SR|R)
- `role` (DPS|Support|Buffer|Healer|Tank)
- `element` (Reason|Odd|Hollow|Constant|Disorder)
- `published` (true|false)
- `sort` (name|rarity|role|element|updated_at)
- `order` (asc|desc)

### Shells

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | `/shells` | List all shells | viewer+ |
| GET | `/shells/:id` | Get single shell | viewer+ |
| POST | `/shells` | Create shell | editor+ |
| PUT | `/shells/:id` | Update shell | editor+ |
| DELETE | `/shells/:id` | Delete shell | editor+ |
| POST | `/shells/bulk` | Bulk import | editor+ |
| GET | `/shells/export` | Export all | viewer+ |
| POST | `/shells/:id/image` | Upload icon | editor+ |

### Matrix Sets

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | `/matrix` | List all matrix sets | viewer+ |
| GET | `/matrix/:id` | Get single set | viewer+ |
| POST | `/matrix` | Create set | editor+ |
| PUT | `/matrix/:id` | Update set | editor+ |
| DELETE | `/matrix/:id` | Delete set | editor+ |
| POST | `/matrix/bulk` | Bulk import | editor+ |
| GET | `/matrix/export` | Export all | viewer+ |
| POST | `/matrix/:id/image` | Upload icon | editor+ |

### Teams

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | `/teams` | List all teams | viewer+ |
| GET | `/teams/:id` | Get single team | viewer+ |
| POST | `/teams` | Create team | editor+ |
| PUT | `/teams/:id` | Update team | editor+ |
| DELETE | `/teams/:id` | Delete team | editor+ |

### Tiers

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | `/tiers` | List all tier assignments | viewer+ |
| GET | `/tiers/:gameMode` | Get tiers for a game mode | viewer+ |
| PUT | `/tiers` | Bulk update tier assignments | editor+ |
| POST | `/tiers/import` | Import tier data (JSON) | editor+ |
| GET | `/tiers/export` | Export tier data | viewer+ |

### Settings & Admin

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | `/settings` | Get all settings | viewer+ |
| PUT | `/settings` | Update settings | super-admin |
| GET | `/users` | List admin users | super-admin |
| POST | `/users` | Create admin user | super-admin |
| PUT | `/users/:id` | Update user | super-admin |
| DELETE | `/users/:id` | Delete user | super-admin |
| POST | `/change-password` | Change own password | any |
| POST | `/cache/purge` | Purge Cloudflare cache | super-admin |
| POST | `/backup` | Create database backup | super-admin |
| POST | `/restore` | Restore database from backup | super-admin |

### Dashboard

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | `/dashboard/stats` | Get counts for all resources | viewer+ |
| GET | `/dashboard/changelog` | Get recent changes | viewer+ |

---

## 7. UI/UX System

### Toast Notifications

```
Position: top-right, stacked vertically
Duration: 3s (success), 5s (error), manual dismiss (warning)

┌─────────────────────────────────┐
│ ✅ Hero "Blade" updated         │  (success - green left border)
└─────────────────────────────────┘

┌─────────────────────────────────┐
│ ❌ Failed to save: name exists  │  (error - red left border)
└─────────────────────────────────┘

┌─────────────────────────────────┐
│ ⚠️ Are you sure? This can't be  │  (warning - yellow left border)
│    undone.  [Confirm] [Cancel]  │
└─────────────────────────────────┘
```

### Confirmation Dialogs

For destructive operations (delete, bulk delete, restore):

```
┌─────────────────────────────────────┐
│         ⚠️ Delete Hero?             │
│                                     │
│  This will permanently delete       │
│  "Blade" and all associated         │
│  tier assignments.                  │
│                                     │
│  This action cannot be undone.      │
│                                     │
│       [Cancel]  [Delete]            │
│                  (red button)        │
└─────────────────────────────────────┘
```

### Loading States

**Skeleton Screens:**
```
┌──────────────────────────────────┐
│ ████████  ████████  ████  ████  │  (shimmer animation)
│ ████████  ████████  ████  ████  │
│ ████████  ████████  ████  ████  │
└──────────────────────────────────┘
```

**Button Loading:**
```
[⟳ Saving...]  (spinner icon + disabled state)
```

### Keyboard Shortcuts

| Shortcut | Action | Context |
|----------|--------|---------|
| `Ctrl+K` | Open search/command palette | Global |
| `Ctrl+N` | New item (context-dependent) | Global |
| `Ctrl+Enter` | Save/Submit | Modals |
| `Escape` | Close modal/cancel | Modals |
| `Ctrl+Shift+J` | Toggle JSON view | Edit modals |
| `j/k` | Navigate table rows | Tables |
| `e` | Edit selected | Tables |
| `d` | Delete selected (with confirm) | Tables |
| `g h` | Go to Dashboard | Global |
| `g c` | Go to Heroes | Global |
| `g s` | Go to Shells | Global |
| `g m` | Go to Matrix | Global |
| `g t` | Go to Teams | Global |

### Sidebar Navigation

```
┌────────────────────┐
│  🎮 GameGuide      │
│     Admin Panel    │
│                    │
│  ── Content ──     │
│  📊 Dashboard      │  ← active (indigo highlight)
│  🦸 Heroes (93)    │
│  🛡️ Shells (43)    │
│  🔷 Matrix (27)    │
│  👥 Teams (12)     │
│  ⭐ Tier Lists     │
│                    │
│  ── System ──      │
│  ⚙️ Settings        │
│                    │
│                    │
│                    │
│  ── User ──        │
│  👤 admin          │
│  🚪 Logout         │
└────────────────────┘

- Fixed left, 240px wide
- Collapsible to 64px (icon-only) on mobile
- Active item: indigo left border + bg highlight
- Resource counts shown in muted text
- Glassmorphism background matching main site
```

---

## 8. Component Library

### Reusable Components

| Component | Description | Usage |
|-----------|-------------|-------|
| `<admin-table>` | Sortable, paginated data table | Heroes, Shells, Matrix lists |
| `<admin-modal>` | Glassmorphism modal with backdrop | Edit forms, confirmations |
| `<admin-toast>` | Stacked notification system | All actions |
| `<admin-search>` | Search input with debounce | All list pages |
| `<admin-filter>` | Dropdown filter group | All list pages |
| `<admin-upload>` | Drag-and-drop image upload | Hero/Shell/Matrix images |
| `<admin-tag-input>` | Tag input with autocomplete | Hero tags, skill tags |
| `<admin-json-editor>` | Syntax-highlighted JSON textarea | Skills, stats, raw data |
| `<admin-toggle>` | Publish/unpublish toggle | All resources |
| `<admin-badge>` | Colored badge (rarity, tier, role) | Tables, cards |
| `<admin-pagination>` | Page navigation | All list pages |
| `<admin-confirm>` | Confirmation dialog | Destructive actions |
| `<admin-hero-picker>` | Hero search + select modal | Teams, recommendations |
| `<admin-tier-badge>` | Tier badge with color coding | Heroes table, tier editor |
| `<admin-sidebar>` | Navigation sidebar | All pages |

### Alpine.js Component Pattern

```html
<!-- Example: Admin Table Component -->
<div x-data="adminTable({
  endpoint: '/api/admin/heroes',
  columns: ['icon', 'name', 'rarity', 'role', 'element', 'tier'],
  sort: 'name',
  filters: { rarity: 'all', role: 'all' }
})">
  <!-- Search & Filters -->
  <div class="flex gap-4 mb-4">
    <input x-model="search" @input.debounce.300ms="fetch()" 
           placeholder="Search heroes..." class="admin-input">
    <select x-model="filters.rarity" @change="fetch()" class="admin-select">
      <option value="all">All Rarities</option>
      <option value="SSR">SSR</option>
      <option value="SR">SR</option>
      <option value="R">R</option>
    </select>
  </div>
  
  <!-- Table -->
  <table class="admin-table">
    <thead>
      <tr>
        <template x-for="col in columns">
          <th @click="sort(col)" x-text="col.label" :class="{ 'sorted': sortCol === col.key }"></th>
        </template>
      </tr>
    </thead>
    <tbody>
      <template x-for="row in rows">
        <tr @click="edit(row)">
          <td><img :src="row.avatar_card" class="w-8 h-8 rounded"></td>
          <td x-text="row.name"></td>
          <td><span class="badge" :class="'badge-' + row.rarity" x-text="row.rarity"></span></td>
          <!-- ... -->
        </tr>
      </template>
    </tbody>
  </table>
  
  <!-- Pagination -->
  <admin-pagination :page="page" :total="total" @navigate="page = $event; fetch()"></admin-pagination>
</div>
```

---

## 9. Implementation Priorities

### Phase 1: Foundation (Week 1) — **CRITICAL**

| Task | Priority | Effort | Description |
|------|----------|--------|-------------|
| 1.1 | P0 | 4h | Express server setup with SQLite, JWT auth middleware |
| 1.2 | P0 | 3h | Admin login page + auth flow |
| 1.3 | P0 | 2h | Admin SPA shell (sidebar, routing, layout) |
| 1.4 | P0 | 3h | Database schema creation + seed from existing JS data |
| 1.5 | P1 | 2h | Dashboard page with stats |

**Deliverable:** Working admin panel with login, sidebar navigation, dashboard showing real counts.

### Phase 2: Core CRUD (Week 2) — **HIGH**

| Task | Priority | Effort | Description |
|------|----------|--------|-------------|
| 2.1 | P0 | 6h | Heroes management (table + edit modal + CRUD API) |
| 2.2 | P0 | 4h | Shells management (table + edit modal + CRUD API) |
| 2.3 | P0 | 4h | Matrix management (table + edit modal + CRUD API) |
| 2.4 | P1 | 3h | Image upload with Sharp processing (WebP conversion) |
| 2.5 | P1 | 2h | Bulk import/export JSON for all resources |

**Deliverable:** Full CRUD for heroes, shells, matrix with image upload and bulk operations.

### Phase 3: Teams & Tiers (Week 3) — **MEDIUM**

| Task | Priority | Effort | Description |
|------|----------|--------|-------------|
| 3.1 | P1 | 5h | Team compositions (CRUD + hero picker) |
| 3.2 | P1 | 6h | Tier list editor (grid view + drag-and-drop) |
| 3.3 | P2 | 3h | Synergy preview calculation |
| 3.4 | P2 | 2h | Drag-and-drop for team builder (SortableJS) |

**Deliverable:** Team builder with drag-and-drop, tier list editor with grid view.

### Phase 4: Polish & Security (Week 4) — **MEDIUM**

| Task | Priority | Effort | Description |
|------|----------|--------|-------------|
| 4.1 | P1 | 3h | Settings page (account, users, cache) |
| 4.2 | P1 | 2h | Database backup/restore |
| 4.3 | P1 | 2h | Rate limiting, CSRF protection, input validation |
| 4.4 | P2 | 3h | Keyboard shortcuts system |
| 4.5 | P2 | 2h | Toast notifications, confirmation dialogs |
| 4.6 | P2 | 2h | Loading states, skeleton screens |
| 4.7 | P2 | 2h | Changelog/audit log tracking |

**Deliverable:** Production-ready admin panel with full security, polish, and keyboard shortcuts.

### Phase 5: Future Enhancements

| Task | Priority | Effort | Description |
|------|----------|--------|-------------|
| 5.1 | P3 | 4h | Prydwen import (scrape + map to schema) |
| 5.2 | P3 | 3h | Builds section (equipment recommendations per hero) |
| 5.3 | P3 | 3h | Activity chart (changes per day) |
| 5.4 | P3 | 2h | Dark/light theme toggle |
| 5.5 | P3 | 4h | Multi-language support |

---

## 10. File Structure

```
game-guide/
├── admin/
│   ├── index.html              # Admin SPA entry point
│   ├── login.html              # Standalone login page
│   ├── css/
│   │   ├── admin.css           # Admin-specific styles
│   │   └── tailwind.min.css    # Tailwind (same as main)
│   ├── js/
│   │   ├── app.js              # Main admin app (Alpine.js stores)
│   │   ├── router.js           # Client-side routing
│   │   ├── auth.js             # Auth logic (login, refresh, logout)
│   │   ├── api.js              # API client with auto-refresh
│   │   ├── components/
│   │   │   ├── table.js        # Admin table component
│   │   │   ├── modal.js        # Modal system
│   │   │   ├── toast.js        # Toast notifications
│   │   │   ├── upload.js       # Image upload component
│   │   │   ├── tag-input.js    # Tag input component
│   │   │   ├── json-editor.js  # JSON editor component
│   │   │   └── hero-picker.js  # Hero search/select
│   │   └── pages/
│   │       ├── dashboard.js    # Dashboard page
│   │       ├── heroes.js       # Heroes management
│   │       ├── shells.js       # Shells management
│   │       ├── matrix.js       # Matrix management
│   │       ├── teams.js        # Team compositions
│   │       ├── tiers.js        # Tier list editor
│   │       └── settings.js     # Settings page
│   └── assets/
│       └── icons/              # Admin UI icons
├── server/
│   ├── index.js                # Express server entry
│   ├── config.js               # Configuration
│   ├── db/
│   │   ├── schema.sql          # Database schema
│   │   ├── seed.js             # Seed data from existing JS
│   │   └── connection.js       # SQLite connection
│   ├── middleware/
│   │   ├── auth.js             # JWT verification
│   │   ├── role.js             # Role-based access
│   │   ├── rateLimit.js        # Rate limiting
│   │   └── validate.js         # Input validation
│   ├── routes/
│   │   ├── auth.js             # Login, refresh, logout
│   │   ├── heroes.js           # Heroes CRUD
│   │   ├── shells.js           # Shells CRUD
│   │   ├── matrix.js           # Matrix CRUD
│   │   ├── teams.js            # Teams CRUD
│   │   ├── tiers.js            # Tiers management
│   │   ├── dashboard.js        # Dashboard stats
│   │   ├── settings.js         # Settings management
│   │   └── upload.js           # Image upload
│   └── utils/
│       ├── imageProcessor.js   # Sharp image processing
│       ├── backup.js           # Database backup/restore
│       └── changelog.js        # Audit logging
├── data/
│   ├── gameguide.db            # SQLite database file
│   └── backups/                # Database backups
└── package.json
```

---

## Appendix: Glassmorphism Admin Theme

The admin panel reuses the main site's glassmorphism theme with admin-specific additions:

```css
/* Admin-specific overrides */
.admin-card {
  background: rgba(15, 15, 35, 0.8);
  backdrop-filter: blur(12px);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 12px;
}

.admin-input {
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 8px;
  color: #e2e8f0;
  padding: 8px 12px;
  transition: border-color 0.2s;
}

.admin-input:focus {
  border-color: #818cf8;
  outline: none;
  box-shadow: 0 0 0 2px rgba(129, 140, 248, 0.2);
}

.admin-table {
  width: 100%;
  border-collapse: separate;
  border-spacing: 0;
}

.admin-table th {
  background: rgba(255, 255, 255, 0.03);
  padding: 12px 16px;
  text-align: left;
  font-weight: 600;
  font-size: 0.75rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  color: #94a3b8;
  border-bottom: 1px solid rgba(255, 255, 255, 0.06);
}

.admin-table td {
  padding: 12px 16px;
  border-bottom: 1px solid rgba(255, 255, 255, 0.04);
}

.admin-table tr:hover td {
  background: rgba(255, 255, 255, 0.02);
}

.admin-btn-primary {
  background: linear-gradient(135deg, #818cf8, #6366f1);
  color: white;
  padding: 8px 16px;
  border-radius: 8px;
  font-weight: 500;
  transition: all 0.2s;
}

.admin-btn-primary:hover {
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(99, 102, 241, 0.4);
}

.admin-btn-danger {
  background: linear-gradient(135deg, #f87171, #ef4444);
  color: white;
  padding: 8px 16px;
  border-radius: 8px;
  font-weight: 500;
}
```

---

*End of Admin Panel Design Document*
