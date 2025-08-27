# CLAUDE.md — AI Coding Playbook for Hexile

## 1. Purpose & How to Help

Claude must:
- Read all context and existing code before making changes
- Produce a **Plan Artifact** before any file edits (UPI workflow mandatory)
- Prefer small, focused, strictly-typed diffs over large rewrites
- Write GUT tests for every feature; maintain determinism
- Emit CombatLog events for state changes

## 2. Repository Map (Godot)

**Directories:**
- `autoload/` — Singleton services
- `scenes/` — Scene files (.tscn)
- `systems/` — Core game logic modules
- `ai/` — AI actions and scoring
- `ui/` — UI components
- `data/` — JSON data files
- `tests/` — GUT test suites
- `fx/` — Visual effects
- `audio/` — Sound assets

**Autoload Singletons:**
- `RunState` — Seed, RNG service, run variables
- `SaveSystem` — 3 slots, atomic writes
- `AudioBus` — SFX routing, volume persistence
- `CardDB` — Card factory from data/cards.json
- `EnemyDB` — Enemy archetypes and modifiers
- `MapDB` — Encounter graphs and maps
- `Constants` — Game tunables
- `FX` — Damage numbers, camera shake
- `DevOverlay` — F3 console, debug tools
- `CombatLog` — Event recorder

**Core Scenes:**
- `Main.tscn` — Game shell with state machine
- `MTDemo.tscn` — Minimal test scene
- `HexGrid.tscn` — Grid renderer, path/LOS services
- `Enemy.tscn` — Base enemy with AI
- `Player.tscn` — Player pawn and resources
- `Card.tscn` — Card view with drag/hover
- `OverlayVictory.tscn`, `OverlayGameOver.tscn`, `OverlayPause.tscn`
- `RewardScreen.tscn`, `ShopScreen.tscn`, `MapScreen.tscn`

## 3. Non-Negotiables

- **Strict typing**: No Variant warnings allowed
- **Determinism**: Single RNG via RunState.seed; no wall-clock randomness
- **Preview = Resolve**: Same resolver for targeting preview and execution
- **Single-responsibility scenes**: Composition over inheritance
- **Signals**: For cross-scene events; avoid string NodePaths
- **Data-driven**: JSON for cards/enemies/maps; no hardcoded stats
- **CombatLog**: Emit events for all major state changes

## 4. Commands

```bash
# Run editor
godot4

# Headless smoke test
godot4 --headless --quit --render-thread 1 --path . --script res://tests/smoke_runner.gd

# Run GUT tests
godot4 --headless -s res://addons/gut/gut_cmdln.gd -gdir=res://tests -ginclude_subdirs -gexit

# Lint (if configured)
gdformat .
gdlint .
```

## 5. Pull Requests & Commits

- One ticket per PR
- PR title: `HEX-XYZ: <imperative verb>`
- PR body must include:
  - Plan Artifact link
  - Diff summary
  - Test results
  - Screenshots (if UI)
  - Acceptance checklist
- Commit format: `feat(system): implement HEX-XYZ feature`

## 6. Testing Policy

- **Parity**: Preview must match resolve across all LOS/cover fixtures
- **Determinism**: Same seed → same CombatLog output
- **Edge cases**: LOS corners, cover on final pre-target hex, occupancy conflicts
- **Golden fixtures**: For grid/pathfinding/LOS scenarios
- **Smoke test**: All scenes must load without errors

## 7. UPI Workflow (Ultrathink → Plan → Implement)

**Required for every task. Planning uses extended thinking mode.**

### Ultrathink / Plan Artifact (≤200 lines)
- Goal recap from ticket
- Minimal design: interfaces, data shapes, states, failure modes
- File plan: what to read/edit/create
- Test plan: GUT tests + smoke scenarios
- Risks & rollback strategy
- **Do NOT edit files during planning**

### Self-Check
- Verify: determinism, preview=resolve, strict typing, acceptance criteria

### Implement (small typed diffs)
- Update tests first
- Run: `godot4 --headless -s res://addons/gut/gut_cmdln.gd`
- Emit CombatLog events
- Update HUD/tooltip bindings
- Add CHANGELOG entry

### Verify
- Paste CI/test results
- Spot-check: LOS corners, cover negation, occupancy

### Document & Land
- Update imports if needed
- Open PR with full checklist

## 8. Modular Coding Rules (Godot)

- Heavy logic in `systems/`; scenes stay thin
- Typed signals for turn transitions, card plays, status ticks
- Inject dependencies via exported vars; avoid global lookups
- Pathfinder/LOS as pure modules with fixtures
- Single StatusBus as source of truth
- AI actions: stateless scorers + executors; debounced intents

## 9. Ticket Format (PM → Coding Agent)

PM assigns micro-tickets with:
- **ID**: `HEX-XYZ`
- **Goal**: One sentence
- **Deliverables**: Bullet list (files, scenes, data)
- **Dependencies**: Ticket IDs or modules
- **Acceptance**: 
  - Tests passing
  - Smoke passes
  - Preview=resolve parity
  - HUD/tooltips updated
  - CombatLog entries present
- **Notes** (optional): Edge cases, fixtures

*Never change public interfaces unless ticket explicitly allows.*

## 10. Ready Prompts

```
# Planning
"Produce a Plan Artifact for HEX-$ID following UPI. No code or edits."

# Implementation
"Implement HEX-$ID per Plan Artifact. Small typed diffs; update tests; run headless GUT + smoke; paste results; add CHANGELOG + PR checklist."

# Testing
"Create GUT tests for HEX-$ID: success, failure, LOS/cover/occupancy fixtures; include preview==resolve assertions."

# Triage
"CI failed for HEX-$ID. Repair Plan: hypothesis → repro → minimal fix → tests → verification."
```

## 11. Imports

- `@README` — Project overview
- `@docs/CONTRIBUTING.md` — PR etiquette & branching
- `@docs/STYLE.md` — GDScript patterns & signals
- `@docs/TESTING.md` — GUT/smoke setup & fixtures
- `@ENGINEERING_WORKPLAN.md` — Full ticket backlog

## 12. Acceptance Checklist

This file is complete when:
- [ ] ≤200 lines, concise sections
- [ ] Uses Hexile repo map (Godot 4.3)
- [ ] Defines UPI workflow clearly
- [ ] Contains PM ticket format
- [ ] Includes copyable commands & prompts
- [ ] No extra commentary