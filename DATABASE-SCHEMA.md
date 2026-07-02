# Etheria Restart GameGuide — SQLite Database Schema

> **Version:** 1.0.0
> **Engine:** SQLite 3.35+ (for `RETURNING` and generated columns)
> **Character set:** UTF-8
> **Normalization:** 3NF — no JSON blobs, all relationships via junction tables

---

## Table of Contents

1. [Reference / Lookup Tables](#1-reference--lookup-tables)
2. [Core Entity Tables](#2-core-entity-tables)
3. [Junction Tables](#3-junction-tables)
4. [Auth & Audit Tables](#4-auth--audit-tables)
5. [Views](#5-views)
6. [Indexes](#6-indexes-summary)
7. [Seed Data Strategy](#7-seed-data-strategy)
8. [Migration Notes](#8-migration-notes)
9. [Backup Strategy](#9-backup-strategy)

---

## 1. Reference / Lookup Tables

These tables enforce controlled vocabularies and make filtering/indexing efficient.

### 1.1 `elements`

Canonical list of hero elements.

```sql
CREATE TABLE elements (
    id          TEXT PRIMARY KEY,           -- 'reason', 'odd', 'hollow', 'constant', 'disorder'
    name        TEXT NOT NULL UNIQUE,       -- 'Reason', 'Odd', 'Hollow', 'Constant', 'Disorder'
    color       TEXT,                       -- hex color for UI badges, e.g. '#3b82f6'
    sort_order  INTEGER NOT NULL DEFAULT 0, -- display ordering
    created_at  TEXT NOT NULL DEFAULT (datetime('now')),
    updated_at  TEXT NOT NULL DEFAULT (datetime('now'))
);

-- Seed
INSERT INTO elements (id, name, color, sort_order) VALUES
    ('reason',    'Reason',    '#3b82f6', 1),
    ('odd',       'Odd',       '#f59e0b', 2),
    ('hollow',    'Hollow',    '#8b5cf6', 3),
    ('constant',  'Constant',  '#10b981', 4),
    ('disorder',  'Disorder',  '#ef4444', 5);
```

**Migration note:** Element names are title-cased in the existing data. Map via `element.toLowerCase()` → `elements.id`.

---

### 1.2 `rarities`

```sql
CREATE TABLE rarities (
    id          TEXT PRIMARY KEY,           -- 'ssr', 'sr', 'r'
    name        TEXT NOT NULL UNIQUE,       -- 'SSR', 'SR', 'R'
    color       TEXT,                       -- UI badge color
    sort_order  INTEGER NOT NULL DEFAULT 0,
    created_at  TEXT NOT NULL DEFAULT (datetime('now')),
    updated_at  TEXT NOT NULL DEFAULT (datetime('now'))
);

INSERT INTO rarities (id, name, color, sort_order) VALUES
    ('ssr', 'SSR', '#f59e0b', 1),
    ('sr',  'SR',  '#8b5cf6', 2),
    ('r',   'R',   '#3b82f6', 3);
```

**Migration note:** Existing data uses uppercase `'SSR'`, `'SR'`, `'R'`. Normalize to lowercase id.

---

### 1.3 `roles`

```sql
CREATE TABLE roles (
    id          TEXT PRIMARY KEY,           -- 'dps', 'support', 'sustain', 'control', 'debuff'
    name        TEXT NOT NULL UNIQUE,       -- 'DPS', 'Support', 'Sustain', 'Control', 'Debuff'
    icon        TEXT,                       -- optional icon path
    sort_order  INTEGER NOT NULL DEFAULT 0,
    created_at  TEXT NOT NULL DEFAULT (datetime('now')),
    updated_at  TEXT NOT NULL DEFAULT (datetime('now'))
);

INSERT INTO roles (id, name, sort_order) VALUES
    ('dps',     'DPS',     1),
    ('support', 'Support', 2),
    ('sustain', 'Sustain', 3),
    ('control', 'Control', 4),
    ('debuff',  'Debuff',  5);
```

**Migration note:** Existing `hero.role` field maps here. The deprecated `hero.allRoles` (comma-separated) can be ignored or used for validation.

---

### 1.4 `game_modes`

```sql
CREATE TABLE game_modes (
    id          TEXT PRIMARY KEY,           -- 'story_pve', 'pvp', 'threshold_aurora', etc.
    name        TEXT NOT NULL UNIQUE,       -- 'Story/PvE', 'PvP', 'Threshold (Aurora)', etc.
    category    TEXT,                       -- 'general', 'threshold', 'raid' (for grouping)
    sort_order  INTEGER NOT NULL DEFAULT 0,
    created_at  TEXT NOT NULL DEFAULT (datetime('now')),
    updated_at  TEXT NOT NULL DEFAULT (datetime('now'))
);

INSERT INTO game_modes (id, name, category, sort_order) VALUES
    ('story_pve',           'Story/PvE',              'general',   1),
    ('pvp',                 'PvP',                    'general',   2),
    ('threshold_aurora',    'Threshold (Aurora)',     'threshold', 3),
    ('threshold_dokidoki',  'Threshold (DokiDoki)',   'threshold', 4),
    ('threshold_terrormaton','Threshold (Terrormaton)','threshold', 5),
    ('nether_quarry',       'Nether Quarry',          'raid',      6);
```

**Migration note:** Mode keys from existing `hero.tiers` object → slugify to `game_modes.id`. E.g. `'Story/PvE'` → `'story_pve'`.

---

### 1.5 `tiers`

```sql
CREATE TABLE tiers (
    id          TEXT PRIMARY KEY,           -- 't0', 't1', 't2', 't3', 't4', 't5'
    name        TEXT NOT NULL UNIQUE,       -- 'T0', 'T1', ... 'T5'
    priority    INTEGER NOT NULL,           -- 0=highest (T0), 5=lowest (T5) — for sorting
    color       TEXT,                       -- UI badge color (T0=gold, T1=purple, etc.)
    created_at  TEXT NOT NULL DEFAULT (datetime('now')),
    updated_at  TEXT NOT NULL DEFAULT (datetime('now'))
);

INSERT INTO tiers (id, name, priority, color) VALUES
    ('t0', 'T0', 0, '#f59e0b'),
    ('t1', 'T1', 1, '#a855f7'),
    ('t2', 'T2', 2, '#3b82f6'),
    ('t3', 'T3', 3, '#6b7280'),
    ('t4', 'T4', 4, '#9ca3af'),
    ('t5', 'T5', 5, '#d1d5db');
```

---

### 1.6 `shell_rarities`

Separate from hero rarities — shells have their own rarity system.

```sql
CREATE TABLE shell_rarities (
    id          TEXT PRIMARY KEY,           -- 'mythic', 'legendary', 'epic', 'rare'
    name        TEXT NOT NULL UNIQUE,
    color       TEXT,
    sort_order  INTEGER NOT NULL DEFAULT 0,
    created_at  TEXT NOT NULL DEFAULT (datetime('now')),
    updated_at  TEXT NOT NULL DEFAULT (datetime('now'))
);

INSERT INTO shell_rarities (id, name, sort_order) VALUES
    ('mythic',    'Mythic',    1),
    ('legendary', 'Legendary', 2),
    ('epic',      'Epic',      3),
    ('rare',      'Rare',      4);
```

---

## 2. Core Entity Tables

### 2.1 `heroes`

The central entity — 93 records at launch.

```sql
CREATE TABLE heroes (
    id              TEXT PRIMARY KEY,               -- slug: 'lily', 'plume', 'valerian'
    name            TEXT NOT NULL,                   -- display name: 'Lily'
    rarity_id       TEXT NOT NULL REFERENCES rarities(id),
    element_id      TEXT NOT NULL REFERENCES elements(id),
    role_id         TEXT NOT NULL REFERENCES roles(id),  -- primary role (from Prydwen)
    image           TEXT,                            -- 'assets/characters/lily.webp'
    image_detail    TEXT,                            -- 'assets/characters/lily_detail.webp'
    color           TEXT,                            -- hex '#6366f1' for UI accent
    deprecated_roles TEXT,                          -- 'Support, Buffer' (legacy, nullable)
    is_published    INTEGER NOT NULL DEFAULT 1,      -- 0=hidden/draft
    created_at      TEXT NOT NULL DEFAULT (datetime('now')),
    updated_at      TEXT NOT NULL DEFAULT (datetime('now'))
);

-- Index for common filters
CREATE INDEX idx_heroes_rarity   ON heroes(rarity_id);
CREATE INDEX idx_heroes_element  ON heroes(element_id);
CREATE INDEX idx_heroes_role     ON heroes(role_id);
CREATE INDEX idx_heroes_name     ON heroes(name);
```

**Sample data:**
```sql
INSERT INTO heroes (id, name, rarity_id, element_id, role_id, image, image_detail, color)
VALUES ('lily', 'Lily', 'ssr', 'disorder', 'support',
        'assets/characters/lily.webp',
        'assets/characters/lily_detail.webp',
        '#6366f1');
```

**Migration notes:**
- `id` = existing `hero.id` (already slug-cased)
- `rarity_id` = `hero.rarity.toLowerCase()`
- `element_id` = `hero.element.toLowerCase()`
- `role_id` = `hero.role.toLowerCase()`
- `deprecated_roles` = `hero.allRoles` (nullable, preserved for reference)
- `color` = `hero.color`

---

### 2.2 `shells`

Shell equipment sets — 43 records at launch.

```sql
CREATE TABLE shells (
    id              TEXT PRIMARY KEY,               -- 'adamantine-shield'
    name            TEXT NOT NULL,
    rarity_id       TEXT NOT NULL REFERENCES shell_rarities(id),
    set_2pc         TEXT,                            -- 2-piece set effect description
    set_4pc         TEXT,                            -- 4-piece set effect description
    image           TEXT,                            -- 'assets/shells/adamantine-shield.webp'
    is_published    INTEGER NOT NULL DEFAULT 1,
    created_at      TEXT NOT NULL DEFAULT (datetime('now')),
    updated_at      TEXT NOT NULL DEFAULT (datetime('now'))
);

CREATE INDEX idx_shells_rarity ON shells(rarity_id);
```

**Sample data:**
```sql
INSERT INTO shells (id, name, rarity_id, set_2pc, set_4pc, image)
VALUES ('adamantine-shield', 'Adamantine Shield', 'mythic',
        'DEF +15%',
        'When HP drops below 50%, gain shield equal to 20% max HP for 2 turns',
        'assets/shells/adamantine-shield.webp');
```

---

### 2.3 `matrix_sets`

Matrix equipment sets — 27 records at launch.

```sql
CREATE TABLE matrix_sets (
    id              TEXT PRIMARY KEY,               -- 'hawkeye'
    name            TEXT NOT NULL,
    set_2pc         TEXT,                            -- 2-piece effect
    set_4pc         TEXT,                            -- 4-piece effect
    image           TEXT,                            -- 'assets/matrix/hawkeye.webp'
    is_published    INTEGER NOT NULL DEFAULT 1,
    created_at      TEXT NOT NULL DEFAULT (datetime('now')),
    updated_at      TEXT NOT NULL DEFAULT (datetime('now'))
);
```

**Sample data:**
```sql
INSERT INTO matrix_sets (id, name, set_2pc, set_4pc, image)
VALUES ('hawkeye', 'Hawkeye',
        'CRIT Rate +12%',
        'After landing a crit, ATK +20% for 2 turns',
        'assets/matrix/hawkeye.webp');
```

---

### 2.4 `team_compositions`

Prebuilt team compositions — 5 at launch, extensible.

```sql
CREATE TABLE team_compositions (
    id              TEXT PRIMARY KEY,               -- 'team-1'
    name            TEXT NOT NULL,                   -- 'Meta PvP Team'
    description     TEXT,                            -- full description
    synergy         TEXT,                            -- synergy summary
    is_published    INTEGER NOT NULL DEFAULT 1,
    sort_order      INTEGER NOT NULL DEFAULT 0,
    created_at      TEXT NOT NULL DEFAULT (datetime('now')),
    updated_at      TEXT NOT NULL DEFAULT (datetime('now'))
);
```

**Sample data:**
```sql
INSERT INTO team_compositions (id, name, description, synergy)
VALUES ('team-1', 'Meta PvP Team',
        'Balanced team with burst damage and sustain capabilities',
        'High burst damage with sustain');
```

---

## 3. Junction Tables

All many-to-many relationships are expressed through junction tables with composite primary keys.

### 3.1 `hero_tiers`

Maps each hero's tier rating per game mode.

```sql
CREATE TABLE hero_tiers (
    hero_id     TEXT NOT NULL REFERENCES heroes(id) ON DELETE CASCADE,
    mode_id     TEXT NOT NULL REFERENCES game_modes(id) ON DELETE CASCADE,
    tier_id     TEXT NOT NULL REFERENCES tiers(id),
    notes       TEXT,                               -- optional: why this tier
    created_at  TEXT NOT NULL DEFAULT (datetime('now')),
    updated_at  TEXT NOT NULL DEFAULT (datetime('now')),
    PRIMARY KEY (hero_id, mode_id)
);

CREATE INDEX idx_hero_tiers_tier ON hero_tiers(tier_id);
CREATE INDEX idx_hero_tiers_mode ON hero_tiers(mode_id);
```

**Sample data:**
```sql
INSERT INTO hero_tiers (hero_id, mode_id, tier_id) VALUES
    ('lily', 'story_pve',            't0'),
    ('lily', 'pvp',                  't0'),
    ('lily', 'threshold_aurora',     't0'),
    ('lily', 'threshold_dokidoki',   't0'),
    ('lily', 'threshold_terrormaton','t0'),
    ('lily', 'nether_quarry',        't0');
```

**Migration note:** Flatten `hero.tiers` object → iterate keys, map key to `game_modes.id`, value to `tiers.id`. Each hero × mode = one row.

---

### 3.2 `hero_tags`

Skill tags for each hero (e.g., 'Buff', 'Follow-Up', 'AoE').

```sql
-- Tags themselves are freeform text, not a separate reference table,
-- because they're numerous and change frequently.
CREATE TABLE hero_tags (
    hero_id     TEXT NOT NULL REFERENCES heroes(id) ON DELETE CASCADE,
    tag         TEXT NOT NULL,                       -- 'Buff', 'Follow-Up', 'AoE'
    created_at  TEXT NOT NULL DEFAULT (datetime('now')),
    PRIMARY KEY (hero_id, tag)
);

CREATE INDEX idx_hero_tags_tag ON hero_tags(tag);
```

**Sample data:**
```sql
INSERT INTO hero_tags (hero_id, tag) VALUES
    ('lily', 'Buff'),
    ('lily', 'Follow-Up');
```

**Migration note:** Flatten `hero.skillTags` array → one row per tag per hero.

---

### 3.3 `shell_heroes`

Which heroes are recommended for each shell set.

```sql
CREATE TABLE shell_heroes (
    shell_id    TEXT NOT NULL REFERENCES shells(id) ON DELETE CASCADE,
    hero_id     TEXT NOT NULL REFERENCES heroes(id) ON DELETE CASCADE,
    priority    INTEGER NOT NULL DEFAULT 0,          -- 0=top recommendation
    notes       TEXT,                                -- optional recommendation note
    created_at  TEXT NOT NULL DEFAULT (datetime('now')),
    PRIMARY KEY (shell_id, hero_id)
);

CREATE INDEX idx_shell_heroes_hero ON shell_heroes(hero_id);
```

**Sample data:**
```sql
INSERT INTO shell_heroes (shell_id, hero_id, priority) VALUES
    ('adamantine-shield', 'valerian', 0),
    ('adamantine-shield', 'holden',   1);
```

**Migration note:** Flatten `shell.heroes` array → one row per shell × hero, with index as priority.

---

### 3.4 `matrix_heroes`

Which heroes are recommended for each matrix set.

```sql
CREATE TABLE matrix_heroes (
    matrix_id   TEXT NOT NULL REFERENCES matrix_sets(id) ON DELETE CASCADE,
    hero_id     TEXT NOT NULL REFERENCES heroes(id) ON DELETE CASCADE,
    priority    INTEGER NOT NULL DEFAULT 0,
    notes       TEXT,
    created_at  TEXT NOT NULL DEFAULT (datetime('now')),
    PRIMARY KEY (matrix_id, hero_id)
);

CREATE INDEX idx_matrix_heroes_hero ON matrix_heroes(hero_id);
```

**Sample data:**
```sql
INSERT INTO matrix_heroes (matrix_id, hero_id, priority) VALUES
    ('hawkeye', 'lily',  0),
    ('hawkeye', 'plume', 1);
```

---

### 3.5 `team_heroes`

Heroes in each team composition, with slot ordering.

```sql
CREATE TABLE team_heroes (
    team_id     TEXT NOT NULL REFERENCES team_compositions(id) ON DELETE CASCADE,
    hero_id     TEXT NOT NULL REFERENCES heroes(id) ON DELETE CASCADE,
    slot        INTEGER NOT NULL,                   -- 1-5, display order in team
    role_note   TEXT,                               -- e.g. 'Main DPS', 'Flex slot'
    created_at  TEXT NOT NULL DEFAULT (datetime('now')),
    PRIMARY KEY (team_id, hero_id),
    UNIQUE (team_id, slot)                          -- one hero per slot per team
);

CREATE INDEX idx_team_heroes_hero ON team_heroes(hero_id);
```

**Sample data:**
```sql
INSERT INTO team_heroes (team_id, hero_id, slot) VALUES
    ('team-1', 'lily',     1),
    ('team-1', 'plume',    2),
    ('team-1', 'valerian', 3),
    ('team-1', 'liliam',   4),
    ('team-1', 'raymerry', 5);
```

**Migration note:** Iterate `team.heroes` array → slot = array index + 1.

---

## 4. Auth & Audit Tables

### 4.1 `admin_users`

Admin accounts for the management panel.

```sql
CREATE TABLE admin_users (
    id              INTEGER PRIMARY KEY AUTOINCREMENT,
    username        TEXT NOT NULL UNIQUE,
    password_hash   TEXT NOT NULL,                   -- bcrypt/argon2 hash
    display_name    TEXT,
    email           TEXT,
    role            TEXT NOT NULL DEFAULT 'editor',  -- 'admin', 'editor', 'viewer'
    is_active       INTEGER NOT NULL DEFAULT 1,
    last_login_at   TEXT,
    created_at      TEXT NOT NULL DEFAULT (datetime('now')),
    updated_at      TEXT NOT NULL DEFAULT (datetime('now'))
);

CREATE UNIQUE INDEX idx_admin_users_username ON admin_users(username);
```

---

### 4.2 `changelog`

Audit log for all data modifications.

```sql
CREATE TABLE changelog (
    id              INTEGER PRIMARY KEY AUTOINCREMENT,
    table_name      TEXT NOT NULL,                   -- 'heroes', 'shells', etc.
    record_id       TEXT NOT NULL,                   -- primary key of affected record
    action          TEXT NOT NULL CHECK (action IN ('INSERT', 'UPDATE', 'DELETE')),
    old_values      TEXT,                            -- JSON snapshot before change
    new_values      TEXT,                            -- JSON snapshot after change
    changed_by      INTEGER REFERENCES admin_users(id),
    changed_at      TEXT NOT NULL DEFAULT (datetime('now'))
);

CREATE INDEX idx_changelog_table    ON changelog(table_name);
CREATE INDEX idx_changelog_record   ON changelog(table_name, record_id);
CREATE INDEX idx_changelog_date     ON changelog(changed_at);
CREATE INDEX idx_changelog_user     ON changelog(changed_by);
```

---

## 5. Views

### 5.1 `v_heroes_full`

Denormalized hero view with element, rarity, role names — for list pages and API responses.

```sql
CREATE VIEW v_heroes_full AS
SELECT
    h.id,
    h.name,
    h.color,
    h.image,
    h.image_detail,
    h.is_published,
    r.name      AS rarity,
    r.color     AS rarity_color,
    e.name      AS element,
    e.color     AS element_color,
    ro.name     AS role,
    h.deprecated_roles,
    h.created_at,
    h.updated_at
FROM heroes h
JOIN rarities  r  ON h.rarity_id  = r.id
JOIN elements  e  ON h.element_id = e.id
JOIN roles     ro ON h.role_id    = ro.id;
```

### 5.2 `v_hero_tiers`

Hero tier matrix — one row per hero with all tiers pivoted (for tier list pages).

```sql
CREATE VIEW v_hero_tiers AS
SELECT
    h.id            AS hero_id,
    h.name          AS hero_name,
    gm.id           AS mode_id,
    gm.name         AS mode_name,
    gm.category     AS mode_category,
    t.name          AS tier,
    t.priority      AS tier_priority,
    t.color         AS tier_color,
    ht.notes
FROM heroes h
JOIN hero_tiers ht ON h.id      = ht.hero_id
JOIN game_modes gm ON ht.mode_id = gm.id
JOIN tiers       t  ON ht.tier_id = t.id;
```

### 5.3 `v_hero_tags`

Aggregated tags per hero — comma-separated for display.

```sql
CREATE VIEW v_hero_tags AS
SELECT
    h.id    AS hero_id,
    h.name  AS hero_name,
    GROUP_CONCAT(ht.tag, ', ') AS tags
FROM heroes h
LEFT JOIN hero_tags ht ON h.id = ht.hero_id
GROUP BY h.id, h.name;
```

### 5.4 `v_shells_full`

Shells with recommended heroes.

```sql
CREATE VIEW v_shells_full AS
SELECT
    s.id,
    s.name,
    s.set_2pc,
    s.set_4pc,
    s.image,
    sr.name     AS rarity,
    sr.color    AS rarity_color,
    GROUP_CONCAT(h.name, ', ') AS recommended_heroes
FROM shells s
JOIN shell_rarities sr ON s.rarity_id = sr.id
LEFT JOIN shell_heroes sh ON s.id = sh.shell_id
LEFT JOIN heroes h        ON sh.hero_id = h.id
GROUP BY s.id, s.name, s.set_2pc, s.set_4pc, s.image, sr.name, sr.color;
```

### 5.5 `v_matrix_full`

Matrix sets with recommended heroes.

```sql
CREATE VIEW v_matrix_full AS
SELECT
    ms.id,
    ms.name,
    ms.set_2pc,
    ms.set_4pc,
    ms.image,
    GROUP_CONCAT(h.name, ', ') AS recommended_heroes
FROM matrix_sets ms
LEFT JOIN matrix_heroes mh ON ms.id = mh.matrix_id
LEFT JOIN heroes h          ON mh.hero_id = h.id
GROUP BY ms.id, ms.name, ms.set_2pc, ms.set_4pc, ms.image;
```

### 5.6 `v_teams_full`

Team compositions with ordered hero lists.

```sql
CREATE VIEW v_teams_full AS
SELECT
    tc.id,
    tc.name,
    tc.description,
    tc.synergy,
    tc.sort_order,
    GROUP_CONCAT(h.name, ', ') AS heroes_ordered
FROM team_compositions tc
LEFT JOIN team_heroes th ON tc.id = th.team_id
LEFT JOIN heroes h        ON th.hero_id = h.id
GROUP BY tc.id, tc.name, tc.description, tc.synergy, tc.sort_order
ORDER BY tc.sort_order;
```

### 5.7 `v_hero_equipment`

Complete equipment recommendations for a hero (shells + matrix sets).

```sql
CREATE VIEW v_hero_equipment AS
SELECT
    h.id    AS hero_id,
    h.name  AS hero_name,
    'shell' AS equipment_type,
    s.id    AS equipment_id,
    s.name  AS equipment_name,
    sh.priority
FROM heroes h
JOIN shell_heroes sh ON h.id = sh.hero_id
JOIN shells s        ON sh.shell_id = s.id

UNION ALL

SELECT
    h.id    AS hero_id,
    h.name  AS hero_name,
    'matrix' AS equipment_type,
    ms.id   AS equipment_id,
    ms.name AS equipment_name,
    mh.priority
FROM heroes h
JOIN matrix_heroes mh ON h.id = mh.hero_id
JOIN matrix_sets ms    ON mh.matrix_id = ms.id;
```

---

## 6. Indexes Summary

```sql
-- Hero filtering (most common queries)
CREATE INDEX idx_heroes_rarity   ON heroes(rarity_id);
CREATE INDEX idx_heroes_element  ON heroes(element_id);
CREATE INDEX idx_heroes_role     ON heroes(role_id);
CREATE INDEX idx_heroes_name     ON heroes(name);
CREATE INDEX idx_heroes_published ON heroes(is_published);

-- Tier list lookups
CREATE INDEX idx_hero_tiers_tier ON hero_tiers(tier_id);
CREATE INDEX idx_hero_tiers_mode ON hero_tiers(mode_id);

-- Tag searches
CREATE INDEX idx_hero_tags_tag   ON hero_tags(tag);

-- Equipment ↔ hero lookups (reverse direction)
CREATE INDEX idx_shell_heroes_hero   ON shell_heroes(hero_id);
CREATE INDEX idx_matrix_heroes_hero  ON matrix_heroes(hero_id);
CREATE INDEX idx_team_heroes_hero    ON team_heroes(hero_id);

-- Shell/matrix filtering
CREATE INDEX idx_shells_rarity ON shells(rarity_id);

-- Audit trail
CREATE INDEX idx_changelog_table  ON changelog(table_name);
CREATE INDEX idx_changelog_record ON changelog(table_name, record_id);
CREATE INDEX idx_changelog_date   ON changelog(changed_at);
CREATE INDEX idx_changelog_user   ON changelog(changed_by);

-- Admin
CREATE UNIQUE INDEX idx_admin_users_username ON admin_users(username);
```

---

## 7. Seed Data Strategy

### 7.1 Approach

Use a single `seed.sql` file that:

1. Inserts all reference table rows (elements, rarities, roles, game_modes, tiers, shell_rarities)
2. Inserts all hero rows
3. Inserts all junction rows (hero_tiers, hero_tags)
4. Inserts all shell + matrix rows
5. Inserts all junction rows (shell_heroes, matrix_heroes)
6. Inserts team compositions + team_heroes
7. Inserts default admin user

### 7.2 Migration Script (JS/JSON → SQLite)

```js
// migrate.js — Node.js script to convert existing JSON/JS data to SQLite
import Database from 'better-sqlite3';
import heroes from './data/heroes.json' assert { type: 'json' };
import shells from './data/shells.json' assert { type: 'json' };
import matrixSets from './data/matrix.json' assert { type: 'json' };
import teams from './data/teams.json' assert { type: 'json' };

const db = new Database('etheria.db');
db.pragma('journal_mode = WAL');
db.pragma('foreign_keys = ON');

// Run schema
const schema = fs.readFileSync('schema.sql', 'utf8');
db.exec(schema);

// Prepare insert statements
const insertHero = db.prepare(`
  INSERT INTO heroes (id, name, rarity_id, element_id, role_id, image, image_detail, color, deprecated_roles)
  VALUES (@id, @name, @rarity_id, @element_id, @role_id, @image, @image_detail, @color, @deprecated_roles)
`);

const insertHeroTier = db.prepare(`
  INSERT INTO hero_tiers (hero_id, mode_id, tier_id)
  VALUES (@hero_id, @mode_id, @tier_id)
`);

const insertHeroTag = db.prepare(`
  INSERT INTO hero_tags (hero_id, tag)
  VALUES (@hero_id, @tag)
`);

const insertShell = db.prepare(`
  INSERT INTO shells (id, name, rarity_id, set_2pc, set_4pc, image)
  VALUES (@id, @name, @rarity_id, @set_2pc, @set_4pc, @image)
`);

const insertShellHero = db.prepare(`
  INSERT INTO shell_heroes (shell_id, hero_id, priority)
  VALUES (@shell_id, @hero_id, @priority)
`);

const insertMatrixSet = db.prepare(`
  INSERT INTO matrix_sets (id, name, set_2pc, set_4pc, image)
  VALUES (@id, @name, @set_2pc, @set_4pc, @image)
`);

const insertMatrixHero = db.prepare(`
  INSERT INTO matrix_heroes (matrix_id, hero_id, priority)
  VALUES (@matrix_id, @hero_id, @priority)
`);

const insertTeam = db.prepare(`
  INSERT INTO team_compositions (id, name, description, synergy)
  VALUES (@id, @name, @description, @synergy)
`);

const insertTeamHero = db.prepare(`
  INSERT INTO team_heroes (team_id, hero_id, slot)
  VALUES (@team_id, @hero_id, @slot)
`);

// Helper: slugify mode name
const modeMap = {
  'Story/PvE': 'story_pve',
  'PvP': 'pvp',
  'Threshold (Aurora)': 'threshold_aurora',
  'Threshold (DokiDoki)': 'threshold_dokidoki',
  'Threshold (Terrormaton)': 'threshold_terrormaton',
  'Nether Quarry': 'nether_quarry',
};

// Run all inserts in a transaction
const migrate = db.transaction(() => {
  // Heroes
  for (const h of heroes) {
    insertHero.run({
      id: h.id,
      name: h.name,
      rarity_id: h.rarity.toLowerCase(),
      element_id: h.element.toLowerCase(),
      role_id: h.role.toLowerCase(),
      image: h.image,
      image_detail: h.imageDetail,
      color: h.color,
      deprecated_roles: h.allRoles || null,
    });

    // Tiers
    if (h.tiers) {
      for (const [modeName, tierValue] of Object.entries(h.tiers)) {
        const modeId = modeMap[modeName];
        if (modeId) {
          insertHeroTier.run({
            hero_id: h.id,
            mode_id: modeId,
            tier_id: tierValue.toLowerCase(),
          });
        }
      }
    }

    // Tags
    if (h.skillTags) {
      for (const tag of h.skillTags) {
        insertHeroTag.run({ hero_id: h.id, tag });
      }
    }
  }

  // Shells
  for (const s of shells) {
    insertShell.run({
      id: s.id,
      name: s.name,
      rarity_id: s.rarity.toLowerCase(),
      set_2pc: s.set2pc,
      set_4pc: s.set4pc,
      image: s.image,
    });
    if (s.heroes) {
      s.heroes.forEach((heroId, i) => {
        insertShellHero.run({ shell_id: s.id, hero_id: heroId, priority: i });
      });
    }
  }

  // Matrix sets
  for (const m of matrixSets) {
    insertMatrixSet.run({
      id: m.id,
      name: m.name,
      set_2pc: m.set2pc,
      set_4pc: m.set4pc,
      image: m.image,
    });
    if (m.heroes) {
      m.heroes.forEach((heroId, i) => {
        insertMatrixHero.run({ matrix_id: m.id, hero_id: heroId, priority: i });
      });
    }
  }

  // Teams
  for (const t of teams) {
    insertTeam.run({
      id: t.id,
      name: t.name,
      description: t.description,
      synergy: t.synergy,
    });
    if (t.heroes) {
      t.heroes.forEach((heroId, i) => {
        insertTeamHero.run({ team_id: t.id, hero_id: heroId, slot: i + 1 });
      });
    }
  }
});

migrate();
console.log('Migration complete.');
```

---

## 8. Migration Notes

### Field Mapping Cheat Sheet

| Existing JS field | SQLite table.column | Transform |
|---|---|---|
| `hero.id` | `heroes.id` | direct |
| `hero.name` | `heroes.name` | direct |
| `hero.rarity` | `heroes.rarity_id` | `.toLowerCase()` → FK |
| `hero.element` | `heroes.element_id` | `.toLowerCase()` → FK |
| `hero.role` | `heroes.role_id` | `.toLowerCase()` → FK |
| `hero.allRoles` | `heroes.deprecated_roles` | direct (nullable) |
| `hero.skillTags[]` | `hero_tags` rows | one row per tag |
| `hero.tiers{}` | `hero_tiers` rows | key→mode_id, value→tier_id |
| `hero.image` | `heroes.image` | direct |
| `hero.imageDetail` | `heroes.image_detail` | direct |
| `hero.color` | `heroes.color` | direct |
| `shell.id` | `shells.id` | direct |
| `shell.rarity` | `shells.rarity_id` | `.toLowerCase()` → FK |
| `shell.set2pc` | `shells.set_2pc` | direct |
| `shell.set4pc` | `shells.set_4pc` | direct |
| `shell.heroes[]` | `shell_heroes` rows | one row per hero, index→priority |
| `matrix.id` | `matrix_sets.id` | direct |
| `matrix.set2pc` | `matrix_sets.set_2pc` | direct |
| `matrix.set4pc` | `matrix_sets.set_4pc` | direct |
| `matrix.heroes[]` | `matrix_heroes` rows | one row per hero, index→priority |
| `team.id` | `team_compositions.id` | direct |
| `team.heroes[]` | `team_heroes` rows | one row per hero, index+1→slot |

### Data Integrity Checks (post-migration)

```sql
-- Verify all heroes have tiers for all 6 modes
SELECT h.id, h.name, COUNT(ht.mode_id) AS tier_count
FROM heroes h
LEFT JOIN hero_tiers ht ON h.id = ht.hero_id
GROUP BY h.id
HAVING tier_count < 6;

-- Verify no orphaned junction rows
SELECT 'shell_heroes orphan' AS issue, sh.hero_id
FROM shell_heroes sh
LEFT JOIN heroes h ON sh.hero_id = h.id
WHERE h.id IS NULL;

-- Verify all FK references resolve
PRAGMA foreign_key_check;
```

---

## 9. Backup Strategy

### 9.1 WAL-mode Hot Backup

```bash
# Online backup (safe while app is running)
sqlite3 etheria.db ".backup 'backups/etheria-$(date +%Y%m%d-%H%M%S).db'"
```

### 9.2 SQL Dump (portable)

```bash
# Full schema + data as SQL text
sqlite3 etheria.db ".dump" > backups/etheria-$(date +%Y%m%d).sql
```

### 9.3 Automated Cron Backup

```bash
# /etc/cron.d/etheria-backup
0 3 * * * ubuntu sqlite3 /home/ubuntu/etheria.db ".backup '/home/ubuntu/backups/etheria-$(date +\%Y\%m\%d).db'" && \
  find /home/ubuntu/backups -name 'etheria-*.db' -mtime +30 -delete
```

### 9.4 Pre-deploy Snapshot

Before any data migration or admin bulk edit:

```bash
sqlite3 etheria.db ".backup 'backups/pre-deploy-$(date +%Y%m%d-%H%M%S).db'"
```

---

## Entity-Relationship Diagram (text)

```
┌───────────┐     ┌───────────────┐     ┌───────────┐
│ rarities  │────<│    heroes     │>────│ elements  │
└───────────┘     └───────┬───────┘     └───────────┘
                          │
            ┌─────────────┼─────────────┬──────────────┐
            │             │             │              │
     ┌──────┴──────┐ ┌────┴─────┐ ┌─────┴──────┐ ┌────┴──────┐
     │ hero_tiers  │ │hero_tags │ │shell_heroes│ │team_heroes│
     └──────┬──────┘ └──────────┘ └─────┬──────┘ └────┬──────┘
            │                           │              │
     ┌──────┴──────┐              ┌─────┴──────┐ ┌────┴────────────────┐
     │ game_modes  │              │   shells   │ │team_compositions    │
     └─────────────┘              └────────────┘ └─────────────────────┘
                                   ┌────────────┐
                                   │shell_rarity│
                                   └────────────┘

     ┌─────────────┐
     │matrix_sets  │───────── matrix_heroes ──── heroes
     └─────────────┘

     ┌──────────────┐         ┌────────────┐
     │ admin_users  │────<────│ changelog  │
     └──────────────┘         └────────────┘
```

---

## Complete Schema (single-file reference)

```sql
-- ============================================================
-- Etheria Restart GameGuide — SQLite Schema v1.0.0
-- ============================================================
-- Run with: sqlite3 etheria.db < schema.sql
-- Pragmas
PRAGMA journal_mode = WAL;
PRAGMA foreign_keys = ON;
PRAGMA busy_timeout = 5000;

-- ============================================================
-- REFERENCE TABLES
-- ============================================================

CREATE TABLE IF NOT EXISTS elements (
    id          TEXT PRIMARY KEY,
    name        TEXT NOT NULL UNIQUE,
    color       TEXT,
    sort_order  INTEGER NOT NULL DEFAULT 0,
    created_at  TEXT NOT NULL DEFAULT (datetime('now')),
    updated_at  TEXT NOT NULL DEFAULT (datetime('now'))
);

CREATE TABLE IF NOT EXISTS rarities (
    id          TEXT PRIMARY KEY,
    name        TEXT NOT NULL UNIQUE,
    color       TEXT,
    sort_order  INTEGER NOT NULL DEFAULT 0,
    created_at  TEXT NOT NULL DEFAULT (datetime('now')),
    updated_at  TEXT NOT NULL DEFAULT (datetime('now'))
);

CREATE TABLE IF NOT EXISTS roles (
    id          TEXT PRIMARY KEY,
    name        TEXT NOT NULL UNIQUE,
    icon        TEXT,
    sort_order  INTEGER NOT NULL DEFAULT 0,
    created_at  TEXT NOT NULL DEFAULT (datetime('now')),
    updated_at  TEXT NOT NULL DEFAULT (datetime('now'))
);

CREATE TABLE IF NOT EXISTS game_modes (
    id          TEXT PRIMARY KEY,
    name        TEXT NOT NULL UNIQUE,
    category    TEXT,
    sort_order  INTEGER NOT NULL DEFAULT 0,
    created_at  TEXT NOT NULL DEFAULT (datetime('now')),
    updated_at  TEXT NOT NULL DEFAULT (datetime('now'))
);

CREATE TABLE IF NOT EXISTS tiers (
    id          TEXT PRIMARY KEY,
    name        TEXT NOT NULL UNIQUE,
    priority    INTEGER NOT NULL,
    color       TEXT,
    created_at  TEXT NOT NULL DEFAULT (datetime('now')),
    updated_at  TEXT NOT NULL DEFAULT (datetime('now'))
);

CREATE TABLE IF NOT EXISTS shell_rarities (
    id          TEXT PRIMARY KEY,
    name        TEXT NOT NULL UNIQUE,
    color       TEXT,
    sort_order  INTEGER NOT NULL DEFAULT 0,
    created_at  TEXT NOT NULL DEFAULT (datetime('now')),
    updated_at  TEXT NOT NULL DEFAULT (datetime('now'))
);

-- ============================================================
-- CORE ENTITIES
-- ============================================================

CREATE TABLE IF NOT EXISTS heroes (
    id               TEXT PRIMARY KEY,
    name             TEXT NOT NULL,
    rarity_id        TEXT NOT NULL REFERENCES rarities(id),
    element_id       TEXT NOT NULL REFERENCES elements(id),
    role_id          TEXT NOT NULL REFERENCES roles(id),
    image            TEXT,
    image_detail     TEXT,
    color            TEXT,
    deprecated_roles TEXT,
    is_published     INTEGER NOT NULL DEFAULT 1,
    created_at       TEXT NOT NULL DEFAULT (datetime('now')),
    updated_at       TEXT NOT NULL DEFAULT (datetime('now'))
);

CREATE TABLE IF NOT EXISTS shells (
    id           TEXT PRIMARY KEY,
    name         TEXT NOT NULL,
    rarity_id    TEXT NOT NULL REFERENCES shell_rarities(id),
    set_2pc      TEXT,
    set_4pc      TEXT,
    image        TEXT,
    is_published INTEGER NOT NULL DEFAULT 1,
    created_at   TEXT NOT NULL DEFAULT (datetime('now')),
    updated_at   TEXT NOT NULL DEFAULT (datetime('now'))
);

CREATE TABLE IF NOT EXISTS matrix_sets (
    id           TEXT PRIMARY KEY,
    name         TEXT NOT NULL,
    set_2pc      TEXT,
    set_4pc      TEXT,
    image        TEXT,
    is_published INTEGER NOT NULL DEFAULT 1,
    created_at   TEXT NOT NULL DEFAULT (datetime('now')),
    updated_at   TEXT NOT NULL DEFAULT (datetime('now'))
);

CREATE TABLE IF NOT EXISTS team_compositions (
    id           TEXT PRIMARY KEY,
    name         TEXT NOT NULL,
    description  TEXT,
    synergy      TEXT,
    is_published INTEGER NOT NULL DEFAULT 1,
    sort_order   INTEGER NOT NULL DEFAULT 0,
    created_at   TEXT NOT NULL DEFAULT (datetime('now')),
    updated_at   TEXT NOT NULL DEFAULT (datetime('now'))
);

-- ============================================================
-- JUNCTION TABLES
-- ============================================================

CREATE TABLE IF NOT EXISTS hero_tiers (
    hero_id    TEXT NOT NULL REFERENCES heroes(id) ON DELETE CASCADE,
    mode_id    TEXT NOT NULL REFERENCES game_modes(id) ON DELETE CASCADE,
    tier_id    TEXT NOT NULL REFERENCES tiers(id),
    notes      TEXT,
    created_at TEXT NOT NULL DEFAULT (datetime('now')),
    updated_at TEXT NOT NULL DEFAULT (datetime('now')),
    PRIMARY KEY (hero_id, mode_id)
);

CREATE TABLE IF NOT EXISTS hero_tags (
    hero_id    TEXT NOT NULL REFERENCES heroes(id) ON DELETE CASCADE,
    tag        TEXT NOT NULL,
    created_at TEXT NOT NULL DEFAULT (datetime('now')),
    PRIMARY KEY (hero_id, tag)
);

CREATE TABLE IF NOT EXISTS shell_heroes (
    shell_id   TEXT NOT NULL REFERENCES shells(id) ON DELETE CASCADE,
    hero_id    TEXT NOT NULL REFERENCES heroes(id) ON DELETE CASCADE,
    priority   INTEGER NOT NULL DEFAULT 0,
    notes      TEXT,
    created_at TEXT NOT NULL DEFAULT (datetime('now')),
    PRIMARY KEY (shell_id, hero_id)
);

CREATE TABLE IF NOT EXISTS matrix_heroes (
    matrix_id  TEXT NOT NULL REFERENCES matrix_sets(id) ON DELETE CASCADE,
    hero_id    TEXT NOT NULL REFERENCES heroes(id) ON DELETE CASCADE,
    priority   INTEGER NOT NULL DEFAULT 0,
    notes      TEXT,
    created_at TEXT NOT NULL DEFAULT (datetime('now')),
    PRIMARY KEY (matrix_id, hero_id)
);

CREATE TABLE IF NOT EXISTS team_heroes (
    team_id    TEXT NOT NULL REFERENCES team_compositions(id) ON DELETE CASCADE,
    hero_id    TEXT NOT NULL REFERENCES heroes(id) ON DELETE CASCADE,
    slot       INTEGER NOT NULL,
    role_note  TEXT,
    created_at TEXT NOT NULL DEFAULT (datetime('now')),
    PRIMARY KEY (team_id, hero_id),
    UNIQUE (team_id, slot)
);

-- ============================================================
-- AUTH & AUDIT
-- ============================================================

CREATE TABLE IF NOT EXISTS admin_users (
    id            INTEGER PRIMARY KEY AUTOINCREMENT,
    username      TEXT NOT NULL UNIQUE,
    password_hash TEXT NOT NULL,
    display_name  TEXT,
    email         TEXT,
    role          TEXT NOT NULL DEFAULT 'editor',
    is_active     INTEGER NOT NULL DEFAULT 1,
    last_login_at TEXT,
    created_at    TEXT NOT NULL DEFAULT (datetime('now')),
    updated_at    TEXT NOT NULL DEFAULT (datetime('now'))
);

CREATE TABLE IF NOT EXISTS changelog (
    id          INTEGER PRIMARY KEY AUTOINCREMENT,
    table_name  TEXT NOT NULL,
    record_id   TEXT NOT NULL,
    action      TEXT NOT NULL CHECK (action IN ('INSERT', 'UPDATE', 'DELETE')),
    old_values  TEXT,
    new_values  TEXT,
    changed_by  INTEGER REFERENCES admin_users(id),
    changed_at  TEXT NOT NULL DEFAULT (datetime('now'))
);

-- ============================================================
-- INDEXES
-- ============================================================

CREATE INDEX IF NOT EXISTS idx_heroes_rarity     ON heroes(rarity_id);
CREATE INDEX IF NOT EXISTS idx_heroes_element    ON heroes(element_id);
CREATE INDEX IF NOT EXISTS idx_heroes_role       ON heroes(role_id);
CREATE INDEX IF NOT EXISTS idx_heroes_name       ON heroes(name);
CREATE INDEX IF NOT EXISTS idx_heroes_published  ON heroes(is_published);

CREATE INDEX IF NOT EXISTS idx_hero_tiers_tier   ON hero_tiers(tier_id);
CREATE INDEX IF NOT EXISTS idx_hero_tiers_mode   ON hero_tiers(mode_id);

CREATE INDEX IF NOT EXISTS idx_hero_tags_tag     ON hero_tags(tag);

CREATE INDEX IF NOT EXISTS idx_shell_heroes_hero ON shell_heroes(hero_id);
CREATE INDEX IF NOT EXISTS idx_matrix_heroes_hero ON matrix_heroes(hero_id);
CREATE INDEX IF NOT EXISTS idx_team_heroes_hero  ON team_heroes(hero_id);

CREATE INDEX IF NOT EXISTS idx_shells_rarity     ON shells(rarity_id);

CREATE INDEX IF NOT EXISTS idx_changelog_table   ON changelog(table_name);
CREATE INDEX IF NOT EXISTS idx_changelog_record  ON changelog(table_name, record_id);
CREATE INDEX IF NOT EXISTS idx_changelog_date    ON changelog(changed_at);
CREATE INDEX IF NOT EXISTS idx_changelog_user    ON changelog(changed_by);

-- ============================================================
-- TRIGGERS — auto-update updated_at
-- ============================================================

CREATE TRIGGER IF NOT EXISTS trg_heroes_updated
AFTER UPDATE ON heroes
BEGIN
    UPDATE heroes SET updated_at = datetime('now') WHERE id = NEW.id;
END;

CREATE TRIGGER IF NOT EXISTS trg_shells_updated
AFTER UPDATE ON shells
BEGIN
    UPDATE shells SET updated_at = datetime('now') WHERE id = NEW.id;
END;

CREATE TRIGGER IF NOT EXISTS trg_matrix_sets_updated
AFTER UPDATE ON matrix_sets
BEGIN
    UPDATE matrix_sets SET updated_at = datetime('now') WHERE id = NEW.id;
END;

CREATE TRIGGER IF NOT EXISTS trg_team_compositions_updated
AFTER UPDATE ON team_compositions
BEGIN
    UPDATE team_compositions SET updated_at = datetime('now') WHERE id = NEW.id;
END;

CREATE TRIGGER IF NOT EXISTS trg_hero_tiers_updated
AFTER UPDATE ON hero_tiers
BEGIN
    UPDATE hero_tiers SET updated_at = datetime('now')
    WHERE hero_id = NEW.hero_id AND mode_id = NEW.mode_id;
END;
```
