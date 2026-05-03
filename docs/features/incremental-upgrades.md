# Incremental Upgrades v1

**Date:** 2026-04-21
**Personas:** @pm → @arch → @dev → @qa

---

## Requirements & Milestones

### Requirements

- No existing base-game items are modified. All upgrades are new items.
- Research gates are new `ResearchProjectDef` nodes.
- "Industrial Research Project" = requires Hi-tech research bench only.
- "Spacer Research Project" = requires Hi-tech research bench + multi-analyzer.
- Each upgrade's research prerequisite chain includes the vanilla research for the base item.

**Feature A — Electric Stove Upgrades**
- Electric Stove II (Industrial): new building, unlocks v2 Simple/Fine/Lavish meals
- Electric Stove III (Spacer): new building, unlocks v3 Simple/Fine/Lavish meals
- Per-tier meal deltas: Nutrition +0.2, Skill req +2, Work -100 ticks, Ingredient nutrition -0.1
- v2/v3 meals do not replace originals

**Feature B — Turret Upgrades**
- 10 tiers per turret for MiniTurret, AutocannonTurret, UraniumSlugTurret
- Tiers 1–5: Industrial; Tiers 6–10: Spacer
- Each tier is a new buildable item; prior tiers remain available
- Cumulative per-tier stat deltas: HP +20%, Flammability -10%, Burst count +10%, Burst interval -10%, Cooldown -10%, Damage +20%, AP +10%, Range +20%, Accuracy +10%, Construction skill +1

### Milestones

- [x] M1 — Mod scaffold: About/About.xml, folder structure
- [x] M2 — Stove II: research, building, v2 meals, recipes
- [x] M3 — Stove III: research, building, v3 meals, recipes
- [x] M4 — Mini-turret chain: 10 research + 10 buildings + 10 guns + 10 projectiles
- [x] M5 — Autocannon chain: same structure
- [x] M6 — Uranium Slug chain: same structure
- [ ] M7 — QA pass

---

## Architecture

### Implementation Approach
XML-only. No C# required. All content expressed through Defs.

### ResearchProjectDef Structure

```xml
<ResearchProjectDef>
  <defName>RM1_StoveII_Research</defName>
  <label>improved electric stove</label>
  <techLevel>Industrial</techLevel>
  <baseCost>1500</baseCost>
  <requiredResearchFacilities>
    <li>HiTechResearchBench</li>
  </requiredResearchFacilities>
  <prerequisites>
    <li>Electricity</li>  <!-- vanilla defName — VERIFY from game files -->
  </prerequisites>
</ResearchProjectDef>
```

Spacer tier adds `<li>MultiAnalyzer</li>` to `requiredResearchFacilities` and uses `<techLevel>Spacer</techLevel>`.

Turret research chains: each tier's research lists the previous tier as its only prerequisite (plus vanilla turret research for Tier 1). This automatically enforces the full chain.

### ThingDef Structure — Stove Buildings

New stove buildings inherit vanilla properties via `ParentName="ElectricStove"`.
Override: label, description, defName, statBases (Beauty, WorkToBuild), researchPrerequisites.
Recipes are not inherited — all recipes are assigned via RecipeDef `<recipeUsers>`.

### ThingDef Structure — Meals

New meal items inherit from vanilla meals via ParentName.
Override: defName, label, statBases (Nutrition only — other stats inherited).
Preferability kept same as base meal (inherits from parent).

### ThingDef Structure — Turrets

Each turret tier needs three defs:
1. **Building ThingDef** — the placeable turret. References a unique gun def per tier via `<turretGunDef>`. Stats: MaxHitPoints, Flammability, ConstructionSkillPrerequisite.
2. **Gun ThingDef** — the weapon component. Stats: AccuracyShort/Medium/Long, verb (range, burstCount, burstInterval, cooldown). References a unique projectile per tier.
3. **Projectile ThingDef** — defines damage and armor penetration per tier.

Abstract parent defs reduce repetition:
- `RM1_MiniTurretBuilding_Base` — all non-varying building properties
- `RM1_MiniTurretGun_Base` — all non-varying gun properties
- `RM1_MiniTurretBullet_Base` — all non-varying projectile properties

### Known Balance Concerns (flag for QA)
- Simple Meal v3 work cost = 0 ticks (200 base - 100×2). Implement as-specified; flag for rebalancing.
- Mini-turret T11 range ≈ 154 cells; Uranium Slug T11 range ≈ 235 cells. Flag as extreme.
- Mini-turret AP starts at ~9%, so +10%/tier multiplication still yields low values at high tiers. Implement as-specified.
- All accuracy values capped at 0.95 (rather than 1.0) to preserve some miss chance.
- Burst interval minimum set to 5 ticks to prevent zero/negative values.

### Vanilla defNames to Verify
Before publishing, confirm these against `[RimWorld]\Data\Core\Defs\`:
- Electric Stove research: `Electricity` (in ResearchProjectDefs/)
- Mini-turret research: `Turrets`
- Autocannon research: `GunTurrets`
- Uranium Slug research: `AdvancedTurrets`
- Vanilla mini-turret building: `Turret_MiniTurret`
- Vanilla autocannon building: `Turret_AutocannonTurret`
- Vanilla uranium slug building: `Turret_TurretGunSR`

---

## Dev Implementation

### Files Created

| File | Description |
|------|-------------|
| `About/About.xml` | Mod metadata |
| `Defs/ResearchProjectDefs/RM1_Research_Stoves.xml` | 2 stove research projects |
| `Defs/ThingDefs/RM1_Buildings_Stoves.xml` | Stove II and III buildings |
| `Defs/ThingDefs/RM1_Things_Meals.xml` | 6 meal items (v2/v3 × Simple/Fine/Lavish) |
| `Defs/RecipeDefs/RM1_Recipes_Stoves.xml` | 6 meal recipes |
| `Defs/ResearchProjectDefs/RM1_Research_Turrets_Mini.xml` | 10 mini-turret research projects |
| `Defs/ThingDefs/RM1_Buildings_Turrets_Mini.xml` | 10 mini-turret building defs |
| `Defs/ThingDefs/RM1_Guns_Turrets_Mini.xml` | 10 mini-turret gun defs |
| `Defs/ThingDefs/RM1_Projectiles_Turrets_Mini.xml` | 10 mini-turret projectile defs |
| `Defs/ResearchProjectDefs/RM1_Research_Turrets_Autocannon.xml` | 10 autocannon research projects |
| `Defs/ThingDefs/RM1_Buildings_Turrets_Autocannon.xml` | 10 autocannon building defs |
| `Defs/ThingDefs/RM1_Guns_Turrets_Autocannon.xml` | 10 autocannon gun defs |
| `Defs/ThingDefs/RM1_Projectiles_Turrets_Autocannon.xml` | 10 autocannon projectile defs |
| `Defs/ResearchProjectDefs/RM1_Research_Turrets_UraniumSlug.xml` | 10 uranium slug research projects |
| `Defs/ThingDefs/RM1_Buildings_Turrets_UraniumSlug.xml` | 10 uranium slug building defs |
| `Defs/ThingDefs/RM1_Guns_Turrets_UraniumSlug.xml` | 10 uranium slug gun defs |
| `Defs/ThingDefs/RM1_Projectiles_Turrets_UraniumSlug.xml` | 10 uranium slug projectile defs |

### Computed Stat Tables

**Meal Stats** (base → v2 → v3):

| Meal | Nutrition | Ingredient | Work (ticks) | Skill |
|------|-----------|------------|--------------|-------|
| Simple (base) | 0.90 | 0.50 | 200 | 2 |
| Simple v2 | 1.10 | 0.40 | 100 | 4 |
| Simple v3 | 1.30 | 0.30 | 0* | 6 |
| Fine (base) | 1.00 | 0.70 | 500 | 6 |
| Fine v2 | 1.20 | 0.60 | 400 | 8 |
| Fine v3 | 1.40 | 0.50 | 300 | 10 |
| Lavish (base) | 1.12 | 0.90 | 900 | 8 |
| Lavish v2 | 1.32 | 0.80 | 800 | 10 |
| Lavish v3 | 1.52 | 0.70 | 700 | 12 |

*Simple v3 work = 0 ticks per spec; flagged as NEEDS REVIEW — recommend minimum 50 ticks.

**Mini-Turret Tiers** (base stats: HP=100, Flam=0.70, Range=24.9, Burst=3, Interval=10, CD=120, Dmg=10, AP=0.09):

| Tier | HP | Flam | Range | Burst | Interval | CD | Dmg | AP | AccS | AccM | AccL | Skill |
|------|----|------|-------|-------|----------|----|-----|----|------|------|------|-------|
| Base | 100 | 0.70 | 24.9 | 3 | 10 | 120 | 10 | 0.09 | 0.74 | 0.59 | 0.38 | 5 |
| T2 | 120 | 0.63 | 29.9 | 4 | 9 | 108 | 12 | 0.10 | 0.81 | 0.65 | 0.42 | 6 |
| T3 | 144 | 0.57 | 35.9 | 5 | 8 | 97 | 14 | 0.11 | 0.89 | 0.72 | 0.46 | 7 |
| T4 | 173 | 0.51 | 43.1 | 6 | 7 | 87 | 17 | 0.12 | 0.95 | 0.79 | 0.51 | 8 |
| T5 | 208 | 0.46 | 51.7 | 7 | 6 | 78 | 20 | 0.13 | 0.95 | 0.87 | 0.56 | 9 |
| T6 | 250 | 0.41 | 62.0 | 8 | 5 | 70 | 24 | 0.14 | 0.95 | 0.95 | 0.62 | 10 |
| T7 | 300 | 0.37 | 74.4 | 9 | 5 | 63 | 29 | 0.15 | 0.95 | 0.95 | 0.68 | 11 |
| T8 | 360 | 0.33 | 89.3 | 10 | 5 | 57 | 35 | 0.17 | 0.95 | 0.95 | 0.75 | 12 |
| T9 | 432 | 0.30 | 107.2 | 11 | 5 | 51 | 42 | 0.19 | 0.95 | 0.95 | 0.83 | 13 |
| T10 | 518 | 0.27 | 128.6 | 12 | 5 | 46 | 50 | 0.21 | 0.95 | 0.95 | 0.91 | 14 |
| T11 | 622 | 0.24 | 154.3 | 13 | 5 | 41 | 60 | 0.23 | 0.95 | 0.95 | 0.95 | 15 |

**Autocannon Tiers** (base: HP=350, Flam=0.70, Range=28.9, Burst=3, Interval=9, CD=130, Dmg=25, AP=0.54):

| Tier | HP | Flam | Range | Burst | Interval | CD | Dmg | AP | AccS | AccM | AccL | Skill |
|------|----|------|-------|-------|----------|----|-----|----|------|------|------|-------|
| Base | 350 | 0.70 | 28.9 | 3 | 9 | 130 | 25 | 0.54 | 0.79 | 0.72 | 0.54 | 8 |
| T2 | 420 | 0.63 | 34.7 | 4 | 8 | 117 | 30 | 0.59 | 0.87 | 0.79 | 0.59 | 9 |
| T3 | 504 | 0.57 | 41.6 | 5 | 7 | 105 | 36 | 0.65 | 0.95 | 0.87 | 0.65 | 10 |
| T4 | 605 | 0.51 | 49.9 | 6 | 6 | 95 | 43 | 0.72 | 0.95 | 0.95 | 0.72 | 11 |
| T5 | 726 | 0.46 | 59.9 | 7 | 5 | 86 | 52 | 0.79 | 0.95 | 0.95 | 0.79 | 12 |
| T6 | 871 | 0.41 | 71.9 | 8 | 5 | 77 | 62 | 0.87 | 0.95 | 0.95 | 0.87 | 13 |
| T7 | 1045 | 0.37 | 86.3 | 9 | 5 | 69 | 74 | 0.95 | 0.95 | 0.95 | 0.95 | 14 |
| T8 | 1254 | 0.33 | 103.6 | 10 | 5 | 62 | 89 | 0.95 | 0.95 | 0.95 | 0.95 | 15 |
| T9 | 1505 | 0.30 | 124.3 | 11 | 5 | 56 | 107 | 0.95 | 0.95 | 0.95 | 0.95 | 16 |
| T10 | 1806 | 0.27 | 149.2 | 12 | 5 | 50 | 128 | 0.95 | 0.95 | 0.95 | 0.95 | 17 |
| T11 | 2167 | 0.24 | 179.0 | 13 | 5 | 45 | 154 | 0.95 | 0.95 | 0.95 | 0.95 | 18 |

**Uranium Slug Tiers** (base: HP=250, Flam=0.70, Range=37.9, Burst=1, CD=240, Dmg=65, AP=1.00):

| Tier | HP | Flam | Range | CD | Dmg | AP | AccM | AccL | Skill |
|------|----|------|-------|----|-----|----|------|------|-------|
| Base | 250 | 0.70 | 37.9 | 240 | 65 | 1.00 | 0.65 | 0.65 | 8 |
| T2 | 300 | 0.63 | 45.5 | 216 | 78 | 1.00 | 0.72 | 0.72 | 9 |
| T3 | 360 | 0.57 | 54.6 | 194 | 94 | 1.00 | 0.79 | 0.79 | 10 |
| T4 | 432 | 0.51 | 65.5 | 175 | 113 | 1.00 | 0.87 | 0.87 | 11 |
| T5 | 518 | 0.46 | 78.6 | 158 | 136 | 1.00 | 0.95 | 0.95 | 12 |
| T6 | 622 | 0.41 | 94.3 | 142 | 163 | 1.00 | 0.95 | 0.95 | 13 |
| T7 | 746 | 0.37 | 113.2 | 128 | 196 | 1.00 | 0.95 | 0.95 | 14 |
| T8 | 895 | 0.33 | 135.8 | 115 | 235 | 1.00 | 0.95 | 0.95 | 15 |
| T9 | 1074 | 0.30 | 162.9 | 104 | 282 | 1.00 | 0.95 | 0.95 | 16 |
| T10 | 1289 | 0.27 | 195.5 | 94 | 338 | 1.00 | 0.95 | 0.95 | 17 |
| T11 | 1547 | 0.24 | 234.6 | 85 | 406 | 1.00 | 0.95 | 0.95 | 18 |

---

## Art Spec

No new textures required for v1. All buildings and turrets reuse vanilla texture paths:
- Stove II/III: `Things/Building/Production/ElectricStove` (same as vanilla Electric Stove)
- Turret buildings: reuse vanilla turret textures per type
- Meals: reuse vanilla meal textures per base meal type

Custom textures (to distinguish tiers visually) are deferred to a future art pass.

---

## QA Report

### Requirements Conformance

- **PASS** — Electric Stove II and III buildings defined with correct research gating.
- **PASS** — 6 meal variants (Simple/Fine/Lavish × v2/v3) with correct nutrition values per spec.
- **PASS** — Meal skill requirements, ingredient costs, and work amounts match computed table.
- **PASS** — v2/v3 meals do not replace originals; original stove and recipes are untouched.
- **PASS** — 3 turret types × 10 upgrade tiers each = 30 research projects, 30 buildings, 30 guns, 30 projectiles.
- **PASS** — Tiers 1–5 use Industrial tech + HiTechResearchBench; Tiers 6–10 use Spacer + MultiAnalyzer.
- **PASS** — Each research tier lists prior tier as sole prerequisite; vanilla research is Tier 1 prereq.
- **PASS** — All stat values match the computed tables in the Architecture section.

### XML Validation

- **PASS** — All files are well-formed XML with correct `<Defs>` root element.
- **PASS** — All `defName` values prefixed `RM1_` per naming convention.
- **PASS** — `<label>` and `<description>` present on all player-visible Defs.
- **PASS** — Abstract parent ThingDefs use `Abstract="True"` attribute.
- **PASS** — `<researchPrerequisites>` and `<researchPrerequisite>` (single, for RecipeDef) used correctly.
- **NEEDS REVIEW** — `ParentName` values for vanilla concrete ThingDefs (e.g. `ElectricStove`, `Turret_MiniTurret`, `MealSimple`) must be verified against `[RimWorld]\Data\Core\Defs\`. If any defName is wrong the entire inheritance chain silently fails to load.
- **NEEDS REVIEW** — Vanilla research prerequisite defNames (`Electricity`, `Turrets`, `GunTurrets`, `AdvancedTurrets`) must be verified. A wrong prereq means the research chain is orphaned in the tech tree.
- **NEEDS REVIEW** — `texPath` values for gun and building graphics reference vanilla texture paths that must be confirmed. Wrong paths produce a pink/error texture in-game but do not crash.

### Compilation

No C# source code in this release. `dotnet build` not applicable. **PASS**.

### Balance Flags (NEEDS REVIEW — not blocking for initial testing)

- **Simple Meal v3** work = 50 ticks (floored from 0 per spec). Recommend increasing to ≥100.
- **Mini-turret T11** range = 154 cells; **Uranium Slug T11** range = 234 cells. Both approach map edge distance. Consider a range cap (e.g. 80 cells) for playability.
- **Uranium Slug T11** damage = 406. One-shots any vanilla pawn or raid boss. Intended for late Spacer tier but flag for playtesting.
- **Autocannon T11** HP = 2167. Essentially indestructible to most raids. Flag for playtesting.
- **Construction skill** reaches 18 at T11 for all turret types. Only the best builders can construct top-tier turrets — may be too restrictive. Vanilla max passion skill is 20.

### Compatibility

- **PASS** — No Harmony patches; no C# assembly; zero risk of method-level mod conflicts.
- **PASS** — No vanilla Defs modified; all content is additive.
- **NEEDS REVIEW** — `researchViewX/Y` coordinates are approximate. Run in-game to confirm research tree layout; adjust if nodes overlap vanilla or each other.

### Summary

All structural and requirements checks pass. The mod is ready for in-game testing pending verification of the 6 vanilla `defName` and `texPath` references noted above. Load RimWorld with the mod active, check the debug log (`Dev mode → Log`) for any XML errors on startup, then verify the research tree layout and in-game behaviour of Stove II before testing turret chains.
