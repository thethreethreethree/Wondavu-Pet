# Wondavu Pet — Sprite List

The complete art roster needed across all phases. Spec a designer against
this. All sprites land in `public/pet/` in the WAVIVI repo; current files
there are placeholder SVGs that match the slugs below 1:1 — real art can
swap in any time without code changes.

**Common spec for all sprites:**
- Format: SVG preferred (scales, themes well), PNG @2x acceptable
- Square canvas: pet/dormant/stage = 120×120 viewBox, items/icons = 24–80
- Single artwork file per slug — theme variants done via CSS, not separate files (unless explicitly listed)
- Style: chibi watercolor — warm sunset palette (peach #ffc892, sunset #e5814b, leaf #86b894, cheek #ff9bb0, line #9b5520)
- Outline: ~2px earthy-brown line; transparent background; gentle inner glow

---

## Phase 1 — MVP (ship-blocking)

The bare minimum to ship the pet page and reward toasts.

### Pet states — single species "wanderling"
| Slug             | Size    | Notes                                            |
|------------------|---------|--------------------------------------------------|
| `egg.svg`        | 120×120 | Speckled egg with leaf wisp on top. Default state on signup. |
| `hatchling.svg`  | 120×120 | Chibi blob with backpack peeking, leaf wisp, sparkle eyes, smile. |
| `dormant.svg`    | 120×120 | Sleeping/curled form, X-eyes, zzz, muted palette. Shown when neglected. |

### Mood variants of hatchling
| Slug                   | Notes                                                |
|------------------------|------------------------------------------------------|
| `hatchling-sick.svg`   | Same body; flat eyes, sad mouth, fever cheeks, green tint. |
| `hatchling-happy.svg`  | Same body; closed-arc happy eyes, wide smile, sparkle accents around. |

### UI essentials
| Slug                     | Size   | Notes                                       |
|--------------------------|--------|---------------------------------------------|
| `wc-coin.svg`            | 64×64  | Wondacoin currency icon — gold with "W" mark. |
| `sparkle-burst.svg`      | 80×80  | Reward toast effect. Goes on the +10 WC popup. |
| `evolve-glow.svg`        | 160×160 | Background halo for stage-up moments. Used behind sprite during egg→hatchling transition. |

### Stat icons (already line-art, render in `currentColor`)
| Slug                       | Size  |
|----------------------------|-------|
| `stat-hunger.svg`          | 24×24 |
| `stat-happiness.svg`       | 24×24 |
| `stat-energy.svg`          | 24×24 |
| `stat-cleanliness.svg`     | 24×24 |
| `stat-wanderlust.svg`      | 24×24 |
| `stat-bond.svg`            | 24×24 |

**Total Phase 1: 14 sprites**

---

## Phase 2 — Evolution stages (single form each)

When the pet grows past Hatchling. Each stage adds 1–2 visual elements
to the same character — they should read as the same Wanderling at
different ages.

| Slug             | Stage     | Visual additions over previous                       |
|------------------|-----------|-------------------------------------------------------|
| `pup.svg`        | Pup       | Slightly bigger body, tiny boots, leaf wisp grows.   |
| `explorer.svg`   | Explorer  | Red neckerchief/scarf, full visible backpack with strap loop, boots more defined. |
| `wayfarer.svg`   | Wayfarer  | Sun hat with red band, weathered shoulder bag with rolled blanket. |
| `elder.svg`      | Elder     | Wise round spectacles, walking stick with hanging leaf charm, longer flowing leaf wisp. |

**Total Phase 2: 4 sprites**

---

## Phase 3 — Branch variants

At Explorer onward, the pet picks a path. Each branch is a re-skin of the
same stage sprite with thematic accessories. Order of priority:

### Explorer branches
| Slug                       | Branch     | Theme                                  |
|----------------------------|-----------|----------------------------------------|
| `explorer-explorer.svg`    | Explorer  | Telescope or binoculars, map sticking out |
| `explorer-social.svg`      | Social    | Group chat icon, friendship bracelet, phone tucked in pocket |
| `explorer-foodie.svg`      | Foodie    | Chef hat tilt, mini chopsticks/fork accessory |
| `explorer-homebody.svg`    | Homebody  | Cozy blanket cape, slippers |

### Wayfarer branches (same idea, mature versions)
| Slug                       |
|----------------------------|
| `wayfarer-explorer.svg`    |
| `wayfarer-social.svg`      |
| `wayfarer-foodie.svg`      |
| `wayfarer-homebody.svg`    |
| `wayfarer-adventurer.svg`  |

### Elder branches (rare forms)
| Slug                       |
|----------------------------|
| `elder-explorer.svg`       |
| `elder-social.svg`         |
| `elder-foodie.svg`         |
| `elder-homebody.svg`       |
| `elder-adventurer.svg`     |

**Total Phase 3: 14 sprites**

---

## Phase 4 — Shop catalog

### Food items (consumable)
Each ~80×80, isometric-ish painted vibe.

| Slug                            | Type          | Region tag    | Notes                              |
|---------------------------------|---------------|---------------|-------------------------------------|
| `item-food-rice-ball.svg`       | Food          | —             | Generic; cheapest item, 5 WC       |
| `item-food-bread.svg`           | Food          | —             |                                    |
| `item-food-gelato.svg`          | Food          | —             | Pink scoop, summer vibe            |
| `item-food-taco.svg`            | Food          | MX            | Region-unlock                      |
| `item-food-pho.svg`             | Food          | VN            | Region-unlock                      |
| `item-food-banh-mi.svg`         | Food          | VN            | Region-unlock                      |
| `item-food-pad-thai.svg`        | Food          | TH            | Region-unlock                      |
| `item-food-adobo.svg`           | Food          | PH            | Region-unlock                      |
| `item-food-sinigang.svg`        | Food          | PH            | Region-unlock                      |
| `item-food-curry.svg`           | Food          | IN            | Region-unlock                      |
| `item-food-sushi.svg`           | Food          | JP            | Region-unlock                      |
| `item-food-ramen.svg`           | Food          | JP            | Region-unlock                      |
| `item-food-croissant.svg`       | Food          | FR            | Region-unlock                      |
| `item-food-pizza.svg`           | Food          | IT            | Region-unlock                      |

**Goal:** 1–2 dishes per major travel region. ~14 to start.

### Toys (consumable, restore happiness)
| Slug                          | Notes                            |
|-------------------------------|----------------------------------|
| `item-toy-ball.svg`           | Soccer / beach ball              |
| `item-toy-kite.svg`           | Diamond kite                     |
| `item-toy-frisbee.svg`        | Beach frisbee                    |
| `item-toy-music-box.svg`      | Small wind-up box                |

### Accessories — hats (cosmetic, equipped)
| Slug                          | Notes                            |
|-------------------------------|----------------------------------|
| `item-hat-sun-hat.svg`        | Wide-brim straw                  |
| `item-hat-beanie.svg`         | Warm grey beanie                 |
| `item-hat-conical.svg`        | Asian conical farmer hat         |
| `item-hat-snorkel-mask.svg`   | Beach snorkel goggles            |
| `item-hat-baseball-cap.svg`   | Cap                              |
| `item-hat-bucket-hat.svg`     | Bucket hat                       |
| `item-hat-flower-crown.svg`   | Flower crown                     |

### Accessories — body (cosmetic, equipped)
| Slug                          | Notes                            |
|-------------------------------|----------------------------------|
| `item-body-backpack.svg`      | Bigger backpack replacement      |
| `item-body-scarf.svg`         | Knit scarf                       |
| `item-body-hawaiian.svg`      | Hawaiian shirt                   |
| `item-body-raincoat.svg`      | Yellow raincoat                  |
| `item-body-camera-strap.svg`  | Camera on neck strap             |
| `item-body-passport.svg`      | Mini passport tucked in pocket   |

### Backgrounds (cosmetic, displayed behind pet on /pet page)
~360×240 painted environments.

| Slug                          | Notes                            |
|-------------------------------|----------------------------------|
| `bg-beach.svg`                | Beach + palm tree                |
| `bg-mountain.svg`             | Snowy peak                       |
| `bg-cafe.svg`                 | Cozy café window                 |
| `bg-jungle.svg`               | Lush jungle                      |
| `bg-city-rooftop.svg`         | City skyline at dusk             |
| `bg-night-sky.svg`            | Stars + moon                     |
| `bg-bamboo-grove.svg`         | Bamboo forest                    |

### Boost icons (small, decorative)
| Slug                          | Notes                            |
|-------------------------------|----------------------------------|
| `item-boost-xp.svg`           | Star with +XP                    |
| `item-boost-freeze.svg`       | Snowflake (stat freeze)          |
| `item-boost-wake-up.svg`      | Sparkly potion (revive dormant)  |

### Special
| Slug                          | Notes                            |
|-------------------------------|----------------------------------|
| `item-special-evolution-stone.svg` | Crystal with embedded leaf — for branch override |

**Total Phase 4: ~45 sprites**

---

## Phase 5 — Future / nice-to-have

### Additional species (alternate pet types unlocked at Wayfarer stage)
Each needs its own full sprite progression (egg → elder, branches).

| Species    | Vibe                                         |
|------------|----------------------------------------------|
| `compass-cat` | Cat-like creature whose tail is a compass needle |
| `cloudpup`    | Dog made of cloud, floats above ground       |
| `mapfox`      | Fox with tail patterned like a folded map    |

**Each species adds ~25 sprites if all stages + branches are covered.**

### Theme variants
WAVIVI has 4 themes — Light, Dark, Cute, Rustic. If theme-specific
sprites become desirable later, prefix with theme name:

```
hatchling-cute.svg
hatchling-rustic.svg
```

Currently the placeholder SVGs use a palette that works across all
themes via warm tones + earthy outlines. Theme-specific variants are not
required until art polish.

---

## Summary by phase

| Phase | Sprites | Cumulative |
|-------|--------:|-----------:|
| 1 — MVP states & UI essentials       | 14    | 14    |
| 2 — Evolution stages (4 more)        | 4     | 18    |
| 3 — Branch variants                  | 14    | 32    |
| 4 — Shop catalog                     | ~45   | 77    |
| 5 — Additional species               | +25 each | varies |

---

## File-naming rules

- All lowercase, kebab-case
- Stage sprites: `<stage>.svg` (e.g. `wayfarer.svg`)
- Branch variants: `<stage>-<branch>.svg` (e.g. `wayfarer-foodie.svg`)
- Mood variants: `<base>-<mood>.svg` (e.g. `hatchling-sick.svg`)
- Items: `item-<category>-<name>.svg` (e.g. `item-food-banh-mi.svg`)
- Backgrounds: `bg-<name>.svg`
- Stat icons: `stat-<key>.svg`

This naming maps 1:1 to `pet_item.sprite` and `pet_item.slug` columns in
the schema — no rename layer in code.

---

## How to drop new art into the codebase

1. Save to `WAVIVI/public/pet/` with the slug above.
2. For shop items, add a row to `pet_item` (use the admin console once
   it's built, or insert via migration).
3. The pet sprite mapper at `WAVIVI/src/features/pet/lib/sprites.ts`
   already returns paths by `(stage, status)` — extending to branches is
   a one-line switch update.

Placeholder SVGs currently live in those slots, so the dev environment
works end-to-end. Real art swaps file-for-file; no code changes needed.
