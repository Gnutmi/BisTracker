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

## Architecture

**Single-file HTML** — everything is in one file:
- `<style>` block: All CSS (dark theme, grid layout, responsive)
- `<script>` block: All data + rendering logic in vanilla JavaScript
- Wowhead tooltip integration via external script: `https://wow.zamimg.com/js/tooltips.js`

**No framework dependencies.** Pure vanilla JS with innerHTML-based rendering. A `render()` function rebuilds the spec list whenever filters change or a row is toggled. After each render, `$WowheadPower.refreshLinks()` is called to activate tooltips on new DOM nodes.

## Key Data Structures

### `SPECS` array
Each spec is an object created via `mk()` helper:
```
{
  class: "Death Knight",
  spec: "Frost",
  role: "Melee DPS",       // "Tank" | "Healer" | "Melee DPS" | "Ranged DPS"
  armor: "Plate",           // "Cloth" | "Leather" | "Mail" | "Plate"
  tierSet: "Relentless Rider's Lament",
  tierSlots: {              // 5 tier piece slots, each with name + stats
    Head: { name: "...", stats: "Haste / Mastery" },
    Shoulders: { ... }, Chest: { ... }, Hands: { ... }, Legs: { ... }
  },
  tierUsed: ["Head","Chest","Hands","Legs"],  // which 4 to equip
  tierSkipped: "Shoulders",                    // which 1 to skip
  tier2pc: "...",           // 2-piece bonus description
  tier4pc: "...",           // 4-piece bonus description
  bisWeapon: { name: "...", id: 254170 },     // id = Wowhead item ID or null
  altWeapon: { name: "...", id: null },
  trinkets: [
    { name: "Gaze of the Alnseer", r: "BiS 1" },
    { name: "Light Company Guidon", r: "BiS 2" }
  ],
  altTrinkets: ["Heart of Wind", "Algethar Puzzle Box"]
}
```

### `TRINKETS` object
Maps trinket names → `{ id: wowheadItemId, src: "drop source" }`. Used for generating Wowhead links and source labels.

### `TIER_IDS` object
Maps class name → `{ H: itemId, Sh: itemId, Ch: itemId, Gl: itemId, Le: itemId }` for tier piece Wowhead links.

### `WEAPONS` object
Maps notable weapon names → Wowhead item IDs.

### `CC` object
WoW class colors (hex) keyed by class name.

## Data Sources & Accuracy

Data was scraped/compiled from:
- **Wowhead Beta** (tier set item IDs, item names, stats)
- **Method.gg** class guides (trinket rankings, weapon recommendations, tier set bonuses)
- **Maxroll.gg** (tier set bonus text)
- **Icy Veins** (spec list, BiS structure)
- Various community sources (Overgear, BlazingBoost, BoostMatch)

**Important caveats:**
- Voidspire raid opened **March 17, 2026** — many recommendations were pre-season/theoretical at time of creation
- Some Wowhead item IDs (especially trinkets) may need verification as the live database stabilizes
- Tier piece "skip" recommendations are generalized — actual skip depends on available non-tier alternatives at your item level
- Trinket rankings will shift as tuning happens — **Gaze of the Alnseer** was widely regarded as overtuned
- Always recommend users sim on Raidbots for personalized results

## Known Issues / Areas to Improve

1. **Item ID accuracy**: Some trinket IDs (254xxx range) were from beta datamining and may have changed on live. Verify against live Wowhead and update the `TRINKETS` object.
2. **Missing specific weapon drops**: Most specs list "Crafted w/ Darkmoon Sigil: Hunt" as BiS weapon. Specific raid weapon drops per spec could be added (e.g., which boss drops which weapon type).
3. **Tier skip may vary**: The "skip" piece is a general recommendation. Some specs may prefer different skip slots depending on available M+ or crafted alternatives. Consider making this configurable or adding notes.
4. **Mobile responsiveness**: The collapsed row uses flexbox wrapping which works okay on mobile but could be improved. The expanded detail grid switches to single-column below 800px.
5. **No Mythic+ BiS**: This is raid-only. A separate M+ column or toggle could be added.
6. **Healer trinkets**: Healer trinket meta is less settled than DPS. Consider adding more nuanced notes for healing specs.
7. **Embellishments**: Not currently tracked. Could add a "Best Embellishments" field per spec (most use Darkmoon Sigil: Hunt + Arcanoweave Lining).
8. **Sort options**: Could add sorting by class, role, or alphabetical spec name.

## How to Modify

### Adding/updating a spec's data
Find the spec in the `SPECS` array (they're ordered by class alphabetically). Each is a `mk()` call with positional arguments:
```
mk(class, spec, role, armor, tierSetName, tierSlotObject, usedArray, skipSlot, tier2pcText, tier4pcText, bisWeaponObj, altWeaponObj, trinketsArray, altTrinketsArray)
```

### Adding a new trinket
Add to the `TRINKETS` object at the top of the script:
```js
"New Trinket Name": { id: 123456, src: "Boss Name (Raid)" },
```
Then reference by exact name string in spec trinket arrays.

### Updating a Wowhead item ID
- Tier pieces: Update `TIER_IDS["ClassName"]` object
- Trinkets: Update `TRINKETS["Trinket Name"].id`
- Weapons: Update `WEAPONS["Weapon Name"]` value

### Styling
All CSS is in the `<style>` block at the top. Key classes:
- `.row` / `.row.open` — spec row container
- `.detail-grid` — the two-column expanded layout
- `.tr1` / `.tr2` — trinket box accent colors
- `.wep-val` — weapon box styling

## File

- `midnight-bis-reference.html` — the complete standalone app (single file, ~35KB)
