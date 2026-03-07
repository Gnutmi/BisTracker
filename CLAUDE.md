# CLAUDE.md — WoW Midnight Season 1 Raid BiS Reference Tool

## Project Overview

This is an interactive **Best-in-Slot gear reference** for all 40 specializations in World of Warcraft: Midnight (patch 12.0.1, Season 1). It is a single standalone HTML file (`midnight-bis-reference.html`) that opens in any browser with no build tools or dependencies required.

## What It Does

- Lists all **40 specs** across 13 classes (including the new Devourer Demon Hunter spec)
- For each spec shows:
  - **Tier set pieces** (5 slots: Head, Shoulders, Chest, Hands, Legs) with the 4 pieces to equip and 1 to skip
  - **2-piece and 4-piece set bonuses**
  - **BiS weapon** and alternative weapon
  - **BiS trinket 1 & 2** and alternative trinkets
- All items with known Wowhead item IDs are **hyperlinked to Wowhead** and show **hover tooltips** via the Wowhead tooltip script
- Supports **filtering** by class (dropdown), role (buttons: Tank/Healer/Melee DPS/Ranged DPS), and free-text search
- Click any spec row to expand/collapse the detailed view
- Dark void-themed UI with WoW class colors

## File

- `midnight-bis-reference.html` — the complete standalone app (single file, ~52KB)
- `CLAUDE.md` — AI assistant documentation (this file)
- `README.md` — user-facing documentation with GitHub Pages link

## Architecture

**Single-file HTML** — everything is in one file:
- `<head>`: Wowhead tooltip configuration (colorLinks, iconizeLinks) + external script tag
- `<style>` block: All CSS (dark theme, grid layout, responsive design)
- `<body>`: Static shell HTML (header, filter bar, empty list container, footer)
- `<script>` block: All data + rendering logic in vanilla JavaScript

**No framework dependencies.** Pure vanilla JS with `innerHTML`-based rendering. A `render()` function rebuilds the spec list whenever filters change or a row is toggled. After each render, `$WowheadPower.refreshLinks()` is called after a 300ms delay to activate tooltips on newly added DOM nodes.

**External dependency:** `https://wow.zamimg.com/js/tooltips.js` — loaded async, non-blocking.

---

## JavaScript Structure

The `<script>` block is organized in the following order:

### 1. Item Databases

**`TRINKETS` object**
Maps trinket name → `{ id: wowheadItemId, src: "drop source" }`. Used by `trinketLink()` and `trinketFull()`. Use `id: null` when the live Wowhead ID is unknown.
```js
"Gaze of the Alnseer": { id: 249343, src: "Chimaerus (Dreamrift)" },
"Resonant Bellowstone": { id: null, src: "Dungeon Drop" },
```

**`WEAPONS` object**
Maps named weapon → Wowhead item ID. Only populated for weapons with specific known IDs. Used by `weaponLink()` as a fallback lookup.
```js
"Bellamy's Final Judgement": 249277,
"Blade of the Final Twilight": 249281,
```

**`TIER_IDS` object**
Maps class name → `{ H: headId, Sh: shoulderId, Ch: chestId, Gl: gloveId, Le: legId }`. Provides Wowhead IDs for tier piece links. Referenced via `SLOT_KEY` during rendering.
```js
"Death Knight": { H: 249970, Sh: 249968, Ch: 249973, Gl: 249971, Le: 249969 },
```

**`SLOT_KEY` object**
Maps full slot names to the abbreviated keys used in `TIER_IDS`:
```js
{ Head: "H", Shoulders: "Sh", Chest: "Ch", Hands: "Gl", Legs: "Le" }
```

**`CC` object**
WoW class colors (hex strings) keyed by class name. Used to color class labels and border accents on expanded rows.

**`RI` object**
Maps role strings to emoji icons for the UI:
```js
{ Tank: "🛡️", Healer: "💚", "Melee DPS": "⚔️", "Ranged DPS": "🏹" }
```

---

### 2. Link Helper Functions

| Function | Purpose |
|---|---|
| `whLink(id, name)` | Returns a Wowhead `<a>` tag if `id` is non-null, otherwise plain colored text |
| `trinketLink(name)` | Wraps `whLink()` using the `TRINKETS` database for ID lookup |
| `trinketFull(name)` | Returns `trinketLink(name)` + ` — drop source` |
| `weaponLink(w)` | Returns link for `{name, id}` weapon object; falls back to `WEAPONS` lookup, then plain text |

All link helpers use `data-wowhead="item={id}"` attributes which the Wowhead tooltip script reads.

---

### 3. Spec Data

**`DATA_VERSION`** — string in `"YYYY-MM-DD"` format (e.g., `"2026-03-07"`). Displayed in the page header. Update this when you update any gear data.

**`mk()` factory function** — creates a spec object from positional arguments. Argument order:
```
mk(class, spec, role, armor, tierSetName, tierSlotObject,
   usedSlotsArray, skippedSlot, tier2pcText, tier4pcText,
   bisWeaponObj, altWeaponObj, trinketsArray, altTrinketsArray)
```

**Tier slot constants** — one per class, named by 2-letter abbreviation. Each maps slot name → `{ name, stats }`:
```
DK (Death Knight), DH (Demon Hunter), DR (Druid), EV (Evoker),
HU (Hunter), MA (Mage), MO (Monk), PA (Paladin),
PR (Priest), RO (Rogue), SH (Shaman), WL (Warlock), WA (Warrior)
```
Example:
```js
const DK = {
  Head:      { name: "Relentless Rider's Crown",      stats: "Haste / Mastery" },
  Shoulders: { name: "Relentless Rider's Shoulderguards", stats: "Haste / Mastery" },
  Chest:     { name: "Relentless Rider's Breastplate",  stats: "Haste / Crit" },
  Hands:     { name: "Relentless Rider's Gauntlets",    stats: "Crit / Mastery" },
  Legs:      { name: "Relentless Rider's Greaves",      stats: "Haste / Mastery" },
};
```

**Weapon/item shorthand helpers:**
- `c(n)` — crafted weapon: `{ name: "Crafted [n] w/ Darkmoon Sigil: Hunt", id: null }`
- `r(n)` — raid item placeholder: `{ name: n, id: null }`

**`SPECS` array** — 40 spec objects ordered alphabetically by class, then spec within class. Each is created via `mk()`.

---

### 4. Rendering Logic

**State variables** (module-level):
- `openKey` — key of currently expanded row (`"ClassName-SpecName"` or `null`)
- `currentClassFilter` — selected class name or `"All"`
- `currentRoleFilter` — selected role or `"All"`
- `currentSearch` — lowercase search string

**`renderRow(s)`** — converts a spec object to a complete HTML string for one collapsible row. Includes: clickable header with class color, tier piece table, set bonus descriptions, weapon boxes, trinket boxes.

**`getFiltered()`** — returns `SPECS` filtered by all three active filters (class, role, search). Search matches case-insensitively against `class`, `spec`, and `tierSet`.

**`render()`** — calls `getFiltered()`, maps results through `renderRow()`, updates `#list` innerHTML and count display, then calls `$WowheadPower.refreshLinks()` after 300ms.

**`toggle(k)`** — toggles `openKey` between `k` and `null`, then calls `render()`.

**Filter event listeners** — wired to the class dropdown and search input at the bottom of the script.

**Role buttons** — created programmatically and injected before the count span. Each button updates `currentRoleFilter` and toggles `.on`/`.off` CSS classes.

---

## Key Data Structures

### Spec object (output of `mk()`)
```js
{
  class: "Death Knight",
  spec: "Frost",
  role: "Melee DPS",          // "Tank" | "Healer" | "Melee DPS" | "Ranged DPS"
  armor: "Plate",              // "Cloth" | "Leather" | "Mail" | "Plate"
  tierSet: "Relentless Rider's Lament",
  tierSlots: {
    Head:      { name: "...", stats: "Haste / Mastery" },
    Shoulders: { name: "...", stats: "..." },
    Chest:     { name: "...", stats: "..." },
    Hands:     { name: "...", stats: "..." },
    Legs:      { name: "...", stats: "..." },
  },
  tierUsed:    ["Head", "Chest", "Hands", "Legs"],  // 4 equipped pieces
  tierSkipped: "Shoulders",                           // 1 skipped piece
  tier2pc: "...",
  tier4pc: "...",
  bisWeapon: { name: "...", id: 249281 },  // id is Wowhead item ID or null
  altWeapon: { name: "...", id: null },
  trinkets: [
    { name: "Gaze of the Alnseer", r: "BiS 1" },
    { name: "Light Company Guidon", r: "BiS 2" },
  ],
  altTrinkets: ["Heart of Wind", "Algethar Puzzle Box"],
}
```

---

## CSS Key Classes

| Class | Purpose |
|---|---|
| `.hdr` | Page header (gradient background, purple border) |
| `.filters` | Filter bar (search + dropdown + role buttons) |
| `.list` | Main content container (max 1280px) |
| `.row` | Spec row container (collapsible) |
| `.row.open` | Expanded row with class-colored border |
| `.row-head` | Clickable spec header bar |
| `.detail` | Hidden by default; shown when row is open |
| `.detail-grid` | Two-column grid layout; collapses to one column at ≤800px |
| `.tier-h3` | Tier set heading (purple `#a78bfa`) |
| `.wt-h3` | Weapons & Trinkets heading (gold `#fbbf24`) |
| `.stats` | Item stat text (green `#86efac`) |
| `.use-y` | "Equipped" checkmark indicator |
| `.use-n` | "SKIP" indicator (red) |
| `.bonuses` | 2pc/4pc bonus description box |
| `.wep-val` | Weapon box (gold left border) |
| `.tr1` / `.tr2` | Trinket boxes (purple accent shades) |

---

## How to Modify

### Update an existing spec's gear
Find the spec in `SPECS` (ordered alphabetically by class, then spec). Edit the `mk()` call arguments directly. Update `DATA_VERSION` to today's date.

### Add a new trinket
1. Add to `TRINKETS`:
```js
"New Trinket Name": { id: 123456, src: "Boss Name (Voidspire)" },
```
2. Reference it by exact name string in spec `trinkets` or `altTrinkets` arrays.

### Update a Wowhead item ID
- **Tier pieces:** Update `TIER_IDS["ClassName"]` object
- **Trinkets:** Update `TRINKETS["Trinket Name"].id`
- **Named weapons:** Update `WEAPONS["Weapon Name"]`
- **Spec weapon/altWeapon:** Update the `id` field in the `mk()` call directly

### Add a new tier slot constant (new class)
Define a new `const XX = { Head: {...}, Shoulders: {...}, Chest: {...}, Hands: {...}, Legs: {...} }` constant, add the class to `TIER_IDS`, and reference the constant in all specs for that class.

### Styling changes
All CSS is in the `<style>` block at the top of the file. The design uses a dark purple/void color palette. Key colors: background `#0a0a12`, accent purple `#a78bfa`, accent gold `#fbbf24`, stats green `#86efac`.

---

## Data Sources & Accuracy

Data was compiled from:
- **Wowhead** (tier set item IDs, item names, stats) — IDs updated from beta (254xxx range) to live IDs in March 2026
- **Method.gg** class guides (trinket rankings, weapon recommendations, tier set bonuses)
- **Maxroll.gg** (tier set bonus text)
- **Icy Veins** (spec list, BiS structure)
- Various community sources (Overgear, BlazingBoost, BoostMatch)

**Important caveats:**
- Voidspire raid opened **March 17, 2026** — many recommendations were pre-season/theoretical at time of creation
- Tier piece "skip" recommendations are generalized — actual skip depends on available non-tier alternatives at your item level
- Trinket rankings will shift as tuning happens — **Gaze of the Alnseer** was widely regarded as potentially overtuned
- Always recommend users sim on Raidbots for personalized results

---

## Known Issues / Areas to Improve

1. **Trinket ID verification**: Some trinket IDs may still need verification against live Wowhead as the database stabilizes post-launch.
2. **Missing specific weapon drops**: Most specs list crafted weapons. Specific raid weapon drops per spec could be added (e.g., which boss drops which weapon type).
3. **Tier skip variability**: The skipped piece is a general recommendation. Some specs may prefer different skip slots depending on available M+ or crafted alternatives.
4. **Mobile responsiveness**: The collapsed row flexbox wrapping works okay on mobile but could be improved. The expanded detail grid switches to single-column below 800px.
5. **No Mythic+ BiS**: This is raid-only. A separate M+ column or toggle could be added.
6. **Healer trinkets**: Healer trinket meta is less settled than DPS. Consider adding more nuanced notes for healing specs.
7. **Embellishments**: Not currently tracked. Could add a "Best Embellishments" field per spec (most use Darkmoon Sigil: Hunt + Arcanoweave Lining).
8. **Sort options**: Could add sorting by class, role, or alphabetical spec name.
9. **No search by trinket/weapon name**: Search currently only matches on class, spec, and tier set name.
