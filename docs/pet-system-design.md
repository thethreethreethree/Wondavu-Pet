# Wondavu Pet — System Design (v0.1)

A Tamagotchi-style **travel companion** that lives inside WAVIVI. Every user
gets a pet. The pet thrives when its owner explores, meets people, and joins
communities — and weakens when neglected. Users earn **tokens** from in-app
activity and spend them on food, toys, accessories, and upgrades.

This is a design draft, not a final spec. Everything here is meant to be
iterated on before code is written.

---

## 0. Locked decisions (2026-05-28)

| # | Decision                       | Value                                                  |
|---|--------------------------------|--------------------------------------------------------|
| 1 | Code location                  | Direct WAVIVI feature module: `src/features/pet/`      |
| 2 | Real-money tokens?             | No — earned through app activity only                  |
| 3 | Pets per user                  | 1:1 (PK = `user_id`)                                   |
| 4 | Onboarding                     | Auto-hatch a wanderling on signup                      |
| 5 | UI placement                   | Floating badge on Home + dedicated `/pet` page         |
| 6 | Death model                    | No permadeath — dormant + revive                       |
| 7 | Currency name                  | **Wondacoins (WC)**                                    |
| 8 | Pet visibility on `/u/[username]` | Always visible (public)                              |
| 9 | "New friend" trigger           | Mutual traveler notes (both wrote about each other)    |
| 10 | Pet naming                    | User names at hatch, renameable anytime free           |
| 11 | Art for MVP                   | Placeholder SVG sprites (real art swaps in later)      |
| 12 | Push notifications            | Toast-only now; wire to WAVIVI Web Push when it ships  |
| 13 | MVP scope (PR #1)             | Pet + ledger + rule tables, 5 reward hooks, egg+hatchling, free care, no shop |

---

## 1. Design pillars

1. **The pet rewards the real product.** Every action that grows the pet is
   also a WAVIVI growth action — visiting places, making friends, joining
   groups, writing notes. The pet is a gamification layer, not a separate
   game.
2. **The pet has a travel identity.** It's not a generic blob. It has a
   "wanderlust" stat, evolves down branches based on *how* you travel
   (explorer / social / foodie / homebody…), and its habitat reflects places
   the owner has visited.
3. **Care is light, not punishing.** Real Tamagotchis could die in a day if
   you ignored them. That's hostile for a travel app where users disappear
   for a week on a beach. Decay is slow, sickness is recoverable, and the
   pet never permanently dies — it just goes dormant and can be revived.
4. **Seamless WAVIVI integration.** Same stack (Next.js 16 + Supabase),
   same theme system (Light · Dark · Cute · Rustic), same feature-module
   layout, same typography rules. The pet lives at `src/features/pet/` in
   the WAVIVI repo when it ships.

---

## 2. The pet itself

### 2.1 Species (pick one to MVP — the rest are post-launch)

| Slug          | Vibe                                          |
|---------------|-----------------------------------------------|
| `wanderling`  | Tiny chibi explorer w/ backpack (MVP default) |
| `compass-cat` | Cat-like creature whose tail is a compass     |
| `cloudpup`    | Dog made of cloud — floats, weather-themed    |
| `mapfox`      | Fox with a tail patterned like a folded map   |

Each species has the same stats and lifecycle — they differ only in art and
evolution sprites.

### 2.2 Lifecycle stages

```
egg → hatchling → traveler-pup → explorer → wayfarer → elder
```

| Stage         | XP needed | Care difficulty | Forms     |
|---------------|----------:|-----------------|-----------|
| Egg           |         0 | none            | 1         |
| Hatchling     |         1 | high (frequent) | 1         |
| Traveler-pup  |        50 | medium          | 1         |
| Explorer      |       200 | medium          | 2 branches |
| Wayfarer      |       600 | low             | 4 branches |
| Elder         |      1500 | very low        | 4+ rare    |

**Branching at Explorer:** the pet picks a path based on the dominant reward
source over its life so far.

| Path        | Trigger (majority of recent XP from)                 |
|-------------|------------------------------------------------------|
| Explorer    | New places visited                                   |
| Social      | New friends, group joins, chat activity              |
| Foodie      | Restaurant check-ins, food-tagged places             |
| Homebody    | Daily streaks, notes, profile editing — but few new places |

Wayfarer adds a fourth path **Adventurer** (events + experiences). Each
branch has distinct sprites and unlocks branch-specific shop items.

### 2.3 Stats (the care loop)

| Stat        | Range | Decay     | Restored by                                  |
|-------------|------:|-----------|----------------------------------------------|
| Hunger      | 0–100 | -2/hour   | Feeding (food items)                         |
| Happiness   | 0–100 | -1/hour   | Play, visiting places, social actions        |
| Energy      | 0–100 | -1/hour   | Sleep (set "rest" mode 6h, free)             |
| Cleanliness | 0–100 | -0.5/hour | Bath item (cheap), grows back faster after travel |
| Wanderlust  | 0–100 | -3/day idle | Visiting a new place; reading travel notes  |
| Bond        | 0–100 | never decays | Daily interaction; cumulative, slow growth |

**Health** is derived: `min(Hunger, Happiness, Energy, Cleanliness) >= 30 →
healthy`. Below 30 on any stat → **sick** (slowed XP, sad sprite). Below 10
on any stat for 48h → **dormant** (pet curls up; revival costs tokens OR a
single "wake-up" reward action — e.g. one new place visit).

No permadeath. Ever.

### 2.4 Activity modes

- **With you** (default): pet sits in a corner badge across the app
- **At rest**: sleeping; stats decay slower; can't interact for X hours
- **Exploring**: when the user is checked-in at a place, pet appears on the
  map pin and gains a small idle XP trickle
- **Dormant**: dim/grayed; tap to revive

---

## 3. The reward → token loop

### 3.1 Reward triggers

These all hook into **existing WAVIVI events** — the pet system is a
listener, not a parallel tracker.

| Action (WAVIVI source)                                  | XP  | Tokens | Stat bump          |
|---------------------------------------------------------|----:|-------:|--------------------|
| Visit a **new** place (first check-in)                  |  15 |   10   | +Wanderlust 15     |
| Re-visit a saved place                                  |   3 |    2   | +Happiness 5       |
| Save a place to a list                                  |   2 |    1   | —                  |
| Join a new group chat                                   |  10 |   15   | +Happiness 10      |
| Send first message in a group                           |   5 |    5   | —                  |
| New mutual friend (both wrote a traveler_note about each other) |  20 |   20   | +Happiness 15, +Bond 5 |
| Write a traveler note about someone                     |  10 |   10   | +Bond 5            |
| Receive a traveler note                                 |   8 |    8   | +Happiness 10      |
| RSVP to an event                                        |   8 |   10   | +Wanderlust 10     |
| Attend event (check-in at venue during event time)      |  25 |   25   | +Happiness 20, +XP big |
| Complete a trip plan (`trip_planner`)                   |  40 |   50   | +Wanderlust 30     |
| Daily login                                             |   2 |    5   | +Bond 1            |
| 7-day login streak (rolling)                            |  +0 |  +25   | +Bond 5            |
| 30-day login streak                                     |  +0 | +150   | +Bond 15           |
| Profile completed (avatar + bio + 3 countries)          |  20 |   30   | one-time           |
| Verify Instagram                                        |  10 |   20   | one-time           |
| First scan of a new region (admin)                      |   — |    — | — (admin doesn't count) |

**Anti-farming:** each unique place/group/event awards tokens only once per
user. Daily login is once per UTC day. Mutual-friend reward fires only
when *both* sides exist (mirrors the relationship — no spammable one-way
follows). All grants idempotent via `(user_id, action_kind, source_id)`
uniqueness on the ledger.

### 3.2 Token name

Suggestion: **Wondacoins** (`WC`). Alternatives in the open questions
below.

### 3.3 Token sinks (the shop)

| Category         | Examples                                       | Price (WC) |
|------------------|-----------------------------------------------|-----------:|
| Food (consumable)| Rice ball, taco, pho, gelato                  |     5–20   |
| Region food      | Adobo (PH), banh mi (VN), pad thai (TH)       |    15–30   |
| Toys             | Ball, kite, beach frisbee                     |    20–50   |
| Accessory: hat   | Sun hat, beanie, conical hat, snorkel mask    |    50–150  |
| Accessory: body  | Backpack, scarf, hawaiian shirt, raincoat     |    80–200  |
| Background       | Beach, mountain, café, jungle, city rooftop   |   100–300  |
| Boost: XP +20%   | 24h                                           |        80  |
| Boost: stat freeze | 6h (stats don't decay)                      |        50  |
| Wake-up potion   | Revive a dormant pet instantly                |       100  |
| Evolution stone  | Branch override (force pick a path)           |       500  |

Region food is the obvious **soft commerce hook** — feeding your pet a
banh mi after checking in to a Vietnamese place feels great and ties the
pet's "diet" to the user's real travel.

### 3.4 Economy targets

Rough sketch — these are tuning dials, not commitments:

- A casual user (logs in 3×/wk, visits 1 new place/wk, in 2 group chats)
  earns ~80–120 WC/week. Enough for 1 accessory or 4–6 food items.
- An active user (daily login, 3+ new places/wk, in 5+ groups, attends
  events) earns ~400–600 WC/week.
- Power user (trip planner, events, notes) easily 800+ WC/week.

Sink prices target ~2 weeks of casual earning per "wardrobe set"
(hat + outfit + background), so customization feels reachable but not
trivial.

---

## 4. Data model (Supabase)

All tables namespaced `pet_*` to keep WAVIVI migrations clean. Numbered
migration `00XX_pet_core.sql` (number assigned at PR time — currently
**0034** is next after `0033_avatars_dedupe.sql`).

### 4.1 Core tables

```sql
-- One pet per user (1:1)
create table pet (
  user_id        uuid primary key references profiles(id) on delete cascade,
  species        text not null default 'wanderling',
  name           text not null,
  stage          text not null default 'egg',     -- egg|hatchling|pup|explorer|wayfarer|elder
  branch         text,                            -- null until Explorer stage
  xp             integer not null default 0,
  hunger         smallint not null default 80,
  happiness      smallint not null default 80,
  energy         smallint not null default 80,
  cleanliness    smallint not null default 80,
  wanderlust     smallint not null default 50,
  bond           smallint not null default 0,
  status         text not null default 'healthy', -- healthy|sick|dormant
  last_tick_at   timestamptz not null default now(),
  hatched_at     timestamptz,
  created_at     timestamptz not null default now()
);

-- Catalog of shop items (read-only for users; admin-edited)
create table pet_item (
  slug           text primary key,
  category       text not null,                   -- food|toy|hat|body|background|boost|special
  name           text not null,
  description    text,
  price_wc       integer not null,
  effect         jsonb not null default '{}',     -- {hunger:+30}, {xp_mult:1.2,ttl_h:24}, etc.
  region         text,                            -- nullable; for region-specific food
  sprite         text not null,                   -- asset path
  unlock_stage   text,                            -- gate behind a stage if needed
  active         boolean not null default true,
  created_at     timestamptz not null default now()
);

-- What the user owns
create table pet_inventory (
  user_id        uuid references profiles(id) on delete cascade,
  item_slug      text references pet_item(slug),
  qty            integer not null default 0,
  equipped       boolean not null default false,  -- for cosmetics
  acquired_at    timestamptz not null default now(),
  primary key (user_id, item_slug)
);

-- Token ledger — append only, double-entry-ish
create table pet_token_ledger (
  id             bigserial primary key,
  user_id        uuid not null references profiles(id) on delete cascade,
  delta          integer not null,                -- + earned, - spent
  balance_after  integer not null,                -- denormalized for fast reads
  reason         text not null,                   -- 'reward:visit_place', 'spend:item', 'refund', 'admin'
  source_kind    text,                            -- 'place'|'group'|'event'|'note'|'item' etc.
  source_id      text,                            -- the FK id of the source row
  meta           jsonb default '{}',
  created_at     timestamptz not null default now(),
  unique (user_id, reason, source_kind, source_id) -- idempotency for rewards
);

-- Reward rule book (so we can tune without code deploys)
create table pet_reward_rule (
  action_kind    text primary key,                -- 'visit_new_place', 'join_group', ...
  xp             integer not null default 0,
  tokens         integer not null default 0,
  stat_bumps     jsonb not null default '{}',     -- {happiness:10,wanderlust:5}
  cap_per_day    integer,                         -- null = no cap
  one_time       boolean not null default false,
  active         boolean not null default true
);

-- Audit log of pet interactions (feed, play, etc.) — useful for analytics + animations
create table pet_event (
  id             bigserial primary key,
  user_id        uuid not null references profiles(id) on delete cascade,
  kind           text not null,                   -- 'feed','play','sleep','wake','evolve','sick','revive'
  meta           jsonb default '{}',
  at             timestamptz not null default now()
);
```

### 4.2 RLS

- `pet`: user can `select`/`update` own row; insert via trigger on signup.
- `pet_inventory`, `pet_token_ledger`, `pet_event`: user reads own only.
  Inserts only via **server actions** (no client inserts).
- `pet_item`, `pet_reward_rule`: public read; admin-only write.

### 4.3 Triggers / hooks

- **On signup** (`handle_new_user`): also insert a `pet` row with a
  default name placeholder (`"Egg of <username>"`).
- **Reward dispatcher**: server action `awardPetReward(userId, actionKind,
  sourceKind, sourceId)` — wraps a transaction that:
  1. Looks up `pet_reward_rule[actionKind]`
  2. Inserts into `pet_token_ledger` (idempotent on the unique key)
  3. Updates `pet.xp` and stat bumps
  4. Evaluates stage transitions
  5. Inserts a `pet_event` row
  6. Returns `{tokens_earned, xp_earned, stage_changed}` so the UI can
     pop a toast.
- **Decay tick**: a Postgres function `pet_tick(user_id)` that brings the
  pet's stats forward to "now" based on `last_tick_at`. Called lazily on
  every pet read — no cron required for MVP.

### 4.4 Call-sites (where rewards fire)

| Existing flow                                                  | Hook                                       |
|----------------------------------------------------------------|--------------------------------------------|
| Place check-in (new code, ties to `places`/`stays`/etc.)       | `awardPetReward('visit_new_place', ...)`   |
| `join_chat_group` server action (already in `chat/`)           | `awardPetReward('join_group', ...)`        |
| First message insert in a chat group                           | `awardPetReward('first_message', ...)`     |
| `traveler_notes` insert                                        | `awardPetReward('write_note', ...)`        |
| `events` RSVP create                                           | `awardPetReward('rsvp_event', ...)`        |
| `trip_planner` mark-complete action                            | `awardPetReward('complete_trip', ...)`     |
| Auth session created today and no row for today in ledger       | `awardPetReward('daily_login', ...)`       |

Each hook is **one line** added to the existing server action — the
pet feature stays self-contained behind `awardPetReward`.

---

## 5. UI surfaces

### 5.1 Where the pet lives in WAVIVI

| Surface                       | What the pet does                                  |
|-------------------------------|----------------------------------------------------|
| Home / radial hub             | Small pet sprite idling in a corner                |
| Bottom nav                    | Pet icon replaces one slot (between Meet & Notes?) |
| Map (when checked in)         | Pet sits on the pin                                |
| After a reward                | Toast: pet pops up, "+10 WC, +15 XP"               |
| Pet page (`/pet`)             | Full pet view: stats, feed/play/sleep, shop, inv   |
| Shop (`/pet/shop`)            | Browse by category, filter by region              |

### 5.2 Pet page (sketch)

```
┌────────────────────────────────────────────────┐
│  ☰ Wondavu Pet                          ⚙       │
├────────────────────────────────────────────────┤
│        [ pet sprite — animated ]                │
│           "Pippa"  ·  Stage: Explorer           │
│                  Branch: Foodie                  │
│                                                  │
│  ❤  Happiness    ████████░░  82                 │
│  🍴 Hunger       █████░░░░░  54                 │
│  ⚡ Energy       ███████░░░  71                 │
│  ✨ Cleanliness  ████████░░  80                 │
│  🧭 Wanderlust   █████████░  88                 │
│  💞 Bond         ██████░░░░  62                 │
│                                                  │
│  [Feed]  [Play]  [Sleep]  [Bathe]                │
│                                                  │
│  Wondacoins: 320 WC          [Shop ▸]            │
│  XP: 287 / 600 to Wayfarer                       │
└────────────────────────────────────────────────┘
```

Theme system: every sprite/asset gets a Cute and a Rustic variant; Light
and Dark use the same sprites with different ambient backgrounds.

### 5.3 Asset folder (where to put art)

User already has `C:\Users\johns\OneDrive\Documents\GitHub\WAVIVI\ASSETS
SOURCE\Amenities` registered as a working directory. We'll mirror that:

```
ASSETS SOURCE/
  Pet/
    species/wanderling/{egg,hatchling,...}/{cute,rustic}.png
    items/food/{rice-ball,banh-mi,...}.png
    items/accessories/{sun-hat,backpack,...}.png
    backgrounds/{beach,mountain,...}.png
```

---

## 6. Phased rollout

**Phase 0 — Design lock (you are here)**
- Agree on species, stat list, decay rates, token name, reward values

**Phase 1 — MVP (no shop, no evolution)**
- Single species (wanderling), egg + hatchling only
- `pet` table + `pet_token_ledger` + `pet_reward_rule`
- 4 reward hooks: visit place, join group, daily login, write note
- Pet page with stats + feed (1 free food/day) + play
- Wondacoins displayed but no shop yet

**Phase 2 — Shop & evolution**
- Full lifecycle through Explorer (branches activate)
- Shop with food, toys, 1 hat, 1 background
- Boosts (XP boost, stat freeze)

**Phase 3 — Polish & social**
- Wayfarer + Elder stages
- Region-themed food unlocked by check-ins in that region
- "Show off" your pet on your `/u/[username]` profile
- Pet-on-map sprite during check-ins

**Phase 4 — Future**
- Pet-to-pet interactions when two users in same group chat
- Trading items between users
- Seasonal events (limited-time outfits)

---

## 7. MVP build plan (PR #1)

Now that Section 0 is locked, here's the concrete first PR. All work
happens inside the WAVIVI repo at `src/features/pet/`.

### 7.1 Migration `0034_pet_core.sql`

Creates 5 tables: `pet`, `pet_item`, `pet_inventory`, `pet_token_ledger`,
`pet_reward_rule`, `pet_event`. RLS as described in §4.2. A trigger
extends `handle_new_user` (added in `0007`) to insert a default `pet`
row on signup.

Seeds the `pet_reward_rule` table with the 5 MVP rules:

| action_kind        | xp | tokens | stat_bumps                          | one_time |
|--------------------|---:|-------:|-------------------------------------|----------|
| `visit_new_place`  | 15 |   10   | `{wanderlust:+15, happiness:+5}`    | per place |
| `join_group`       | 10 |   15   | `{happiness:+10}`                   | per group |
| `mutual_note`      | 20 |   20   | `{happiness:+15, bond:+5}`          | per pair |
| `write_note`       | 10 |   10   | `{bond:+5}`                         | per note |
| `daily_login`      |  2 |    5   | `{bond:+1}`                         | per UTC day |

### 7.2 Feature module `src/features/pet/`

```
features/pet/
  api/
    award-reward.ts       server action `awardPetReward(userId, actionKind, sourceKind, sourceId)`
    feed-play-sleep.ts    server actions for free care (rate-limited)
    rename.ts             server action: update pet.name
  components/
    pet-badge.tsx         floating sprite for Home (Client Component)
    pet-page.tsx          the /pet route content
    stat-bar.tsx          shared progress bar
    hatch-modal.tsx       first-hatch naming flow
  hooks/
    use-pet.ts            client-side pet state (subscribes to changes)
  lib/
    decay.ts              pet_tick math: apply decay since last_tick_at
    stages.ts             stage transition logic + branch evaluation
    sprites.ts            map (species, stage, branch) -> placeholder SVG path
  types.ts
  index.ts                exports `awardPetReward`, `<PetBadge />`, `<PetPage />`
```

### 7.3 Reward hook integration

Five one-liner edits to existing WAVIVI server actions:

| File (in WAVIVI)                                          | Add                                                |
|-----------------------------------------------------------|----------------------------------------------------|
| place check-in action (TBD — new code if not present)     | `awardPetReward(userId, 'visit_new_place', 'place', placeId)` |
| `features/chat/.../join-group` action                     | `awardPetReward(userId, 'join_group', 'chat_group', groupId)` |
| `features/notes/.../write-note` action                    | `awardPetReward(userId, 'write_note', 'traveler_note', noteId)` + check mutual → `awardPetReward(both, 'mutual_note', 'traveler_note_pair', pairKey)` |
| `middleware.ts` or session refresh                        | once-per-UTC-day check → `awardPetReward(userId, 'daily_login', 'session', dayKey)` |

### 7.4 Routes

- `/pet` — Server Component reads pet state, runs `pet_tick`, renders `<PetPage />`.
- Home page — embed `<PetBadge />` in a corner.
- `/u/[username]` — server-fetch the user's pet and render a read-only mini badge below the avatar.

### 7.5 Placeholder SVG sprites to generate

- `egg.svg` — speckled egg
- `hatchling.svg` — small chibi character peeking out
- 6 stat icons: hunger / happiness / energy / cleanliness / wanderlust / bond
- `pet-dormant.svg` — grayed-out curled-up form
- `wc-coin.svg` — Wondacoin icon

All under `public/pet/` so they can be `<Image>`-loaded.

### 7.6 Out of scope for PR #1

- Shop, inventory UI, accessories, backgrounds, boosts (Phase 2)
- Stages beyond Hatchling (Phase 2 introduces Pup → Explorer w/ branches)
- Region-themed food (Phase 3)
- Pet on map pins during check-in (Phase 3)
- Web Push notifications (wait for WAVIVI's push to ship)
- Bond/Wanderlust above 50 doing anything visible (stats tracked, not yet gated)

### 7.7 Deferred decisions (still to settle, not blocking MVP)

- Exact decay rates (proposed values in §2.3 — will tune after week-1 telemetry).
- Daily-login boundary (proposed UTC; may switch to user-local once a timezone column lands).
- Bond stat growth curve (currently linear — might need diminishing returns).
- Sprite art direction (placeholder for now; Cute + Rustic theme variants when real art arrives).
