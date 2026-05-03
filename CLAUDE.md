# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Project: RimWorld Mod

## Context
A mod for RimWorld built using XML definitions and optionally C# (via Harmony patching).
- **Tech Stack:** RimWorld 1.5, C# / .NET Framework 4.7.2 (for assembly mods), Harmony 2.x, Visual Studio 2022.
- **Mod approach:** XML-first for data definitions; C# only when behavior cannot be expressed in XML (Harmony patches, custom `ThingComp`, `JobDriver`, etc.).
- **Target Platform:** PC (Windows/Linux/Mac via Steam Workshop).

## RimWorld Mod Structure

```
RimworldMod1/
├── About/
│   ├── About.xml          # Mod metadata (name, author, description, version, dependencies)
│   └── Preview.png        # Steam Workshop thumbnail
├── Defs/                  # XML definitions (ThingDef, RecipeDef, HediffDef, etc.)
│   ├── ThingDefs/
│   ├── RecipeDefs/
│   └── ...
├── Patches/               # XML patches (PatchOperations to modify base-game Defs)
├── Assemblies/            # Compiled .dll output (C# mod assembly)
├── Textures/              # Sprite/texture assets (PNG)
├── Languages/             # Translation strings
│   └── English/
│       └── Keyed/
└── Source/                # C# source code (compiled to Assemblies/)
    └── YourMod/
        ├── YourMod.csproj
        └── *.cs
```

## Build & Run Commands

**Build C# assembly (if using C#):**
```bash
dotnet build Source/
```

**Validate XML (basic well-formedness):**
```bash
# No official CLI tool; open RimWorld and check debug log for Def errors
```

**Open project in Visual Studio:**
```bash
start Source/*.sln
```

> RimWorld loads mods from `%AppData%\LocalLow\Ludeon Studios\RimWorld by Ludeon Studios\Config\ModsConfig.xml`. For development, symlink or copy this repo into the RimWorld `Mods/` folder.

## XML vs C# Decision Guide

| Need | Approach |
|------|----------|
| New item, building, plant, animal stat block | XML `ThingDef` |
| New recipe, bill, workbench operation | XML `RecipeDef` |
| New trait, health condition | XML `TraitDef` / `HediffDef` |
| Patch/tweak an existing base-game Def | XML `PatchOperation` in `Patches/` |
| Custom AI job behavior | C# `JobDriver` |
| Attach new data/behavior to an existing Thing | C# `ThingComp` + `CompProperties` |
| Intercept or override a game method | C# Harmony `[HarmonyPatch]` |
| New pawn need, thought, or mental state | XML Def + optional C# |

## Architecture

### XML Definitions
All new game content is declared as XML Defs under `Defs/`. Use `<defName>` that is uniquely prefixed with the mod name (e.g. `RM1_MyThing`) to avoid conflicts. Reference the RimWorld wiki and decompiled source for available Def fields.

### C# Assembly (optional)
If C# is needed:
- Target `.NET Framework 4.7.2` (RimWorld requirement).
- Reference `Assembly-CSharp.dll` and `UnityEngine.CoreModule.dll` from the RimWorld install.
- Use Harmony 2.x for patches; decorate patch classes with `[HarmonyPatch(typeof(TargetClass), nameof(TargetClass.TargetMethod))]`.
- Register Harmony in a `[StaticConstructorOnStartup]` class.
- Prefer `ThingComp` over Harmony patches where possible (less fragile across RimWorld updates).

### Patching Base-Game Defs (XML)
Use `Patches/` XML files with `PatchOperation` nodes to modify vanilla or other-mod Defs instead of copying entire Defs (copy-paste Defs break with updates and incompatible mods).

### Naming Conventions
- Def `defName`: `RM1_PascalCase`
- C# namespaces: `RimworldMod1`
- Texture paths: `Textures/RM1/ThingName`

## Feature Development Workflow (Autonomous Cycle)

Run `/feature` to start. This is a gated process that runs autonomously once requirements are met.

| Stage | Persona | Output |
|-------|---------|--------|
| 1 — Requirements | `@pm` | **STOP:** User answers questions → Spec created. |
| 2 — Design | `@arch` | (Autonomous) Def structure, class design, patch plan. |
| 3 — Implementation | `@dev` | (Autonomous) XML Defs + C# scripts as needed. |
| 4 — QA | `@qa` | (Autonomous) Build check + XML validation + PASS/FAIL report. |

All output is written to `docs/features/<slug>.md` (one file per feature).

## Custom Slash Commands (Agent Personas)

| Command | Persona | Responsibility |
|---------|---------|----------------|
| `/pm` | Project Manager | Requirements, milestones, scope management |
| `/arch` | Architect | Def structure, C# class design, Harmony patch plan |
| `/dev` | Developer | XML authoring, C# implementation |
| `/art` | Artist | Texture specs, UI layouts |
| `/qa` | QA Engineer | XML validation, build check, in-game correctness |
| `/feature` | Full pipeline | Runs the autonomous cycle above |

## Current Priorities
1. [ ] Define mod identity: `About/About.xml` with name, description, and dependencies.
2. [ ] Decide XML-only vs XML+C# scope for first feature.
3. [ ] Set up C# project structure under `Source/` if C# is needed.

## Reference
- RimWorld Modding Tutorials: https://rimworldwiki.com/wiki/Modding_Tutorials
