# HEXILE — Engineering Workplan & Micro‑Ticket Backlog (v0.2)

Owner: Systems Architect & PM (AI)
Engine: Godot 4.3 (GDScript, strict typing)
Scope: Divide work into minimum shippable coding units for other AI to implement. **No code here**—only specs, contracts, and acceptance.

---

## A. Guiding References → Feature Parity Targets

* **Alina of the Arena (hex tactics):** fast turns, readable ranges, kite/zone control. Targets: green/invalid tinting, min/max range handling, kite AI when too close, cone/line attacks, push/pull interactions.
* **Slay the Spire (deckbuilding):** small hand, clear costs, draw/discard/exhaust, reward screens, shops. Targets: hand size caps, pile counts on HUD, choose‑1‑of‑3 reward, exhaust pile inspection.
* **Path of Exile (mapping & economy):** node map with affixes, elites with modifiers, boss maps with guaranteed special drop, deterministic crafting hooks. Targets now: map nodes (combat/elite/shop/boss), elite modifiers, boss node guarantees; later: affix pools and runes/sockets.

**Design KPIs:**

* Rules clarity: 100% of targetable tiles resolve exactly as previewed (LOS/cover/adjacency).
* Turn pace: median player turn time < 15 seconds in playtests.
* Predictable AI: ≥ 95% agreement between telegraphed intent and action taken.

---

## B. Architecture Baseline

**Autoload Singletons (Project → AutoLoad):**

* `RunState` — run variables (seed, node position), RNG service, difficulty flags.
* `SaveSystem` — 3 slots, atomic writes, metadata (playtime, node, deck hash).
* `AudioBus` — SFX routing, volume persistence.
* `CardDB` — load cards from data/cards.json; factory for runtime cards.
* `EnemyDB` — archetypes and elite/boss modifiers.
* `MapDB` — load encounter graphs and map JSONs.
* `Constants` — tunables (AP/Mana caps, damage numbers).
* `FX` — damage numbers, camera shake, hit flashes.
* `DevOverlay` — F3 console, spawn/clear tools, perf meters.
* `CombatLog` — append‑only combat events + replay recorder.

**Core Scenes:**

* `Main.tscn` — game shell (HUD, hand, board, overlays), state machine.
* `MTDemo.tscn` — minimal test scene (4×Strike + 1×Dash, 1 melee dummy).
* `HexGrid.tscn` — grid renderer + path/LOS services.
* `Enemy.tscn` — base enemy with AI brain and intent label.
* `Player.tscn` — pawn, resources, status container.
* `Card.tscn` — card view; drags/hover; tooltips.
* `OverlayVictory.tscn`, `OverlayGameOver.tscn`, `OverlayPause.tscn`.
* `RewardScreen.tscn`, `ShopScreen.tscn`, `MapScreen.tscn`.

**Directories:**

* `autoload/`, `scenes/`, `ui/`, `data/`, `ai/`, `systems/`, `tests/`, `fx/`, `audio/`.

---

## C. Data Contracts (Prose Schema)

**Card (data/cards.json):** id (string), name, cost: AP(int)/Mana(int), rarity(enum), piles(exhaust?\:bool), tags(list), targeting(type: self/adjacent/any LOS/minRange/maxRange, coverBlock?\:bool, requiresLOS?\:bool), effects(list of EffectRefs), sfxID, vfxID, rulesText.

**Effect:** type(enum: Damage, Shield, Burn, Move, Teleport, GrapplePull, SpawnObstacle, ApplyStatus), magnitude(int), ttl(optional), displacement(vec axial), noiseDelta(int), heatDelta(int), conditions(list), order(priority int).

**EnemyArchetype:** id, hp, baseDamage, speed(moves/turn), attackProfile(melee/ranged with min/max/LOS), behaviors(list of AIAction ids), intentStrings(map), onHit effects, modifiers hooks.

**AIAction:** id, preconditions(list), scoring inputs (distance, cover, hp ratios, noise pings), tie‑breakers, execution (sequence of movement/attack steps), cooldown(optional).

**Map Graph (encounter\_map.json):** nodes(list) with id, type(start/combat/elite/shop/reward/boss), edges(list of target node ids), properties(affixes?, rewardTables?, guaranteedDrops?).

**Save Slot:** slotId, runSeed, playtimeSeconds, nodeId, deckList(card ids), relics, gold, hp, statuses, lastUpdated.

**Constants:** AP\_BASE, MANA\_BASE, PLAYER\_HP, MELEE\_DAMAGE, BURN\_TICK, ZAP\_NOISE, MOVE\_NOISE/HEAT, DECAY\_PER\_TURN, BOSS\_HP, BOSS\_PHASE\_THRESHOLD.

---

## D. Definition of Done (per ticket)

* All acceptance tests in GUT under tests/<area> pass.
* Editor Smoke Test loads target scenes without errors.
* No Variant typing warnings; exported vars typed.
* Tooltips and HUD reflect underlying state; CombatLog event emitted.
* DevOverlay command exists if practical to simulate the feature.

---

## E. Milestones & Sprints

**M0 — Project & CI (1 day)**

* Repo skeleton, styleguide, GUT, Editor Smoke pipeline, export presets.

**M1 — Grid & Movement (1–2 days)**

* Hex math, pathfinding, occupancy, targeting tints, Cancel targeting.

**M2 — Turn Core & Player Resources (1–2 days)**

* State machine, AP/Mana, End Turn, status ticks/decay.

**M3 — Card Engine & Baseline Cards (2–3 days)**

* Piles, draw/mulligan, Strike, Dash, Shield, Ignite, Zap, Smoke, Grapple.

**M4 — AI & Intents (2–3 days)**

* Utility AI shell, actions, intent UI, melee/ranged archetypes.

**M5 — Encounters & Meta (2 days)**

* Victory/defeat, reward screen, shop, map traversal, boss fight shell.

**M6 — Polish & Exports (1–2 days)**

* FX/SFX, save/load, options, stability pass.

---

## F. Micro‑Tickets Backlog (grouped by Epic)

Each ticket lists: Goal, Deliverables, Dependencies, Acceptance.

### EPIC 0: Repo, CI, Conventions

**HEX‑000 Repo Bootstrap & Conventions**

* Goal: Create project skeleton with directories and .gdignore, .gitattributes; document naming conventions.
* Deliverables: folders; CONTRIBUTING.md (branch naming, commit format), STYLE.md (signals, enums, autoload init order).
* Acceptance: Editor opens; lint script runs; README explains run.

**HEX‑001 GUT + Editor Smoke CI**

* Goal: Add GUT with sample test; add Editor Smoke test that loads Main.tscn and MTDemo.tscn headless.
* Deliverables: CI workflow; tests pass in pipeline.
* Acceptance: Failing scene load breaks CI.

**HEX‑002 Export Presets (Win/Linux/Web)**

* Goal: Configure export; add cache/mirror fallback for Godot download in CI.
* Acceptance: Web build artifact produced by CI on main.

---

### EPIC 1: Grid, LOS & Cover

**HEX‑010 Hex Axial Math**

* Goal: Provide axial coordinate helpers (distance, neighbors, lines, rings).
* Dependencies: HEX‑000.
* Acceptance: Unit tests for distance, get\_line corner cases (odd/even skew), 100% pass.

**HEX‑011 Pathfinding & Occupancy**

* Goal: Shortest path with blocked tiles; cost 1 per move.
* Acceptance: Given obstacles, returns expected route; moving entity reserves destination tile for this frame.

**HEX‑012 LOS Sampling**

* Goal: Line sampling between two hexes; returns blocked/clear and final pre‑target hex for cover check.
* Acceptance: Test grid fixtures verify LOS correctness around corners.

**HEX‑013 Cover Rule**

* Goal: Implement: cover on final pre‑target hex negates Zap.
* Acceptance: When cover present, preview = "blocked by cover"; damage preview shows 0; on play, no damage.

**HEX‑014 Targeting Tints & Info Label**

* Goal: Real‑time green valid tiles; info label shows distance/LOS/cover/valid.
* Acceptance: Matches rules in 100 scripted scenarios; UX cancel on right‑click/ESC.

---

### EPIC 2: Turn Core, Resources & Status Ticks

**HEX‑020 State Machine (PlayerTurn/EnemyTurn/Resolution)**

* Goal: Deterministic order; end turn button; signals fired on transitions.
* Acceptance: Log shows consistent sequence across seeds.

**HEX‑021 AP & Mana Pools**

* Goal: Reset each player turn; HUD updates; cost validation on play.
* Acceptance: Spend more than available is impossible; costs shown on card.

**HEX‑022 Status Tick & Decay**

* Goal: Burn tick at end of turn; Shield clears at player turn start; Heat/Noise −1 decay per end turn.
* Acceptance: CombatLog entries confirm order: player turn start → clear shields; enemy turn end → apply burns → decay heat/noise.

---

### EPIC 3: Card Engine & Piles

**HEX‑030 Card Data Loading (CardDB)**

* Goal: Load JSON; hot‑reload in DevOverlay; validate required fields.
* Acceptance: Missing field throws friendly error; DevOverlay shows card count.

**HEX‑031 Card Instance & Factory**

* Goal: Instantiate runtime card from DB id; attach effect list and targeting constraints.
* Acceptance: All baseline cards spawn from ids.

**HEX‑032 Piles: Draw/Hand/Discard/Exhaust**

* Goal: Shuffle deterministic per seed; mulligan at encounter start; HUD pile counts.
* Acceptance: Mulligan swaps selected cards; exhaust pile visible in DevOverlay.

**HEX‑033 Card Play Pipeline**

* Goal: Select card → select tile/target → validate → pay costs → resolve effects in order → move card to pile.
* Acceptance: CombatLog canonical entries for each step; failure reasons surfaced to tooltip.

**HEX‑034 Baseline Card: Strike**

* Goal: AP 1, Mana 0, deal 1 to adjacent; no LOS.
* Acceptance: MTDemo kill dummy in one hit; preview damage = 1.

**HEX‑035 Baseline Card: Dash**

* Goal: Short teleport/reposition; ignores occupancy rules for path but not destination.
* Acceptance: Cannot end on blocked tile; logs heat/noise per spec.

**HEX‑036 Baseline Card: Shield**

* Goal: Gain shield; absorbs next instance of damage.
* Acceptance: Incoming damage reduced to 0 once; shield removed at player turn start if unused.

**HEX‑037 Baseline Card: Ignite**

* Goal: Apply Burn status (tick 1 on end turn).
* Acceptance: Multiple Ignite stacks additively tick (if allowed by spec) or refresh per rules; specify behavior in tooltips.

**HEX‑038 Baseline Card: Zap (Ranged)**

* Goal: Damage with LOS; blocked by cover on final pre‑target hex; +2 noise at target tile.
* Acceptance: Validity/preview accurate across cover fixtures; noise ping recorded.

**HEX‑039 Baseline Card: Smoke**

* Goal: Place temporary obstacle with TTL 2 turns.
* Acceptance: Tile becomes blocking; TTL decrements on end turns; disappears after.

**HEX‑03A Baseline Card: Grapple**

* Goal: Pull enemy 1 hex closer; respects occupancy; fails with message if blocked.
* Acceptance: Enemy ends at expected hex or no‑op with clear reason.

---

### EPIC 4: Player Pawn, Resources & Noise/Heat

**HEX‑040 Player Pawn & HUD Binding**

* Goal: Player HP 3; AP/Mana/Shield shown; damage numbers.
* Acceptance: HP hits 0 → Game Over overlay.

**HEX‑041 Noise/Heat Counters**

* Goal: Move +1 heat & +1 noise at destination; Zap +2 noise; both decay −1 per end turn.
* Acceptance: InvestigateNoise AI uses last ping when not adjacent (tie into EPIC 5).

---

### EPIC 5: Enemies & Utility AI

**HEX‑050 Enemy Archetypes: Melee & Archer**

* Goal: Implement base stats, LOS/range profile, kiting rule for Archer.
* Acceptance: Archer retreats when player within min range; Melee closes then attacks.

**HEX‑051 AI Framework (Utility Scoring)**

* Goal: Action set: MeleeAttack, Shoot, MoveToward, Kite, InvestigateNoise.
* Acceptance: Best scoring action chosen deterministically; score breakdown visible in DevOverlay.

**HEX‑052 Intent Telegraphing**

* Goal: UI label over enemy updates after player turn; debounced for perf.
* Acceptance: ≥ 95% match between telegraph and action.

**HEX‑053 Elite Modifiers**

* Goal: Fast (2 moves/turn), Armored (DR + HP), Burning (applies burn on hit).
* Acceptance: Modifiers stack with archetypes; preview reflects DR.

---

### EPIC 6: Encounters, Rewards, Shops, Map

**HEX‑060 Victory/Defeat Flow**

* Goal: Win when all enemies dead; choose reward → proceed; defeat at 0 HP → Game Over overlay.
* Acceptance: Overlay buttons functional; continue advances node.

**HEX‑061 Reward Screen (Choose 1 of 3)**

* Goal: Show 3 card options or skip; add to discard on pick.
* Acceptance: Deck list updates; tutorial tip appears first time.

**HEX‑062 Shop Screen**

* Goal: Prices, Gold; purchase adds to discard; affordance disabled when poor.
* Acceptance: Gold persists; tooltips show card details.

**HEX‑063 Map Screen & Node Graph**

* Goal: Load encounter map JSON; support Start → Combat/Elite → Shop/Reward → Boss.
* Acceptance: Clicking valid edge moves cursor; boss node ends run with Victory overlay + recap.

**HEX‑064 Boss Fight Shell**

* Goal: Boss with 6 HP; phase at 50%; summons minions every 2 turns.
* Acceptance: Phase change visuals; summon cadence consistent; recap lists stats.

---

### EPIC 7: UI/UX & Options

**HEX‑070 Title Screen & Save Slots**

* Goal: New Run / Continue / Quit; 3 slots with metadata.
* Acceptance: Continue loads slot state; slot metadata updates playtime.

**HEX‑071 HUD Polish**

* Goal: AP/Mana, HP, Shield, pile counts, End Turn; hand hints and disables.
* Acceptance: All values correct under stress tests.

**HEX‑072 Overlays & Pause**

* Goal: Game Over, Victory, Pause/Options; volume sliders persist.
* Acceptance: AudioBus reflects volume; ESC toggles pause.

**HEX‑073 DevOverlay**

* Goal: F3 toggles; spawn/clear utilities; intent tooltips.
* Acceptance: No crashes from rapid spawn/clear.

**HEX‑074 FX & Audio Hooks**

* Goal: SFX ids wired to Strike/Zap/hurt/UI click; damage numbers; camera shake.
* Acceptance: Volumes on buses; no clipping; FX don't hinder readability.

---

### EPIC 8: Save/Load & Stability

**HEX‑080 Save System**

* Goal: user:// JSON; atomic writes; corruption guard; slot metadata.
* Acceptance: Kill process mid‑save leaves last good state; resume works.

**HEX‑081 Replay & CombatLog**

* Goal: Append events; re‑play seed reproduces same outcome.
* Acceptance: Deterministic across runs given seed.

---

### EPIC 9: Testing, Balancing, and Tools

**HEX‑090 Test Fixtures: Grid & LOS**

* Goal: Golden fixtures for LOS/cover edge cases.
* Acceptance: All baseline cards pass preview‑vs‑resolve parity tests.

**HEX‑091 AI Scenario Tests**

* Goal: Fixed maps exercise each AI action and kiting.
* Acceptance: Intent vs action parity ≥ 95%.

**HEX‑092 Tuning Harness**

* Goal: Scripted auto‑plays to measure median turn times, kill turns, damage taken.
* Acceptance: Metrics logged per build; guardrails for regressions.

---

## G. Traceability Matrix (GDD → Tickets)

* GDD 1 "Clarity, Snappy": HEX‑014, HEX‑021, HEX‑071, HEX‑074, HEX‑092.
* GDD 2 "Core Loop": HEX‑020, HEX‑033, HEX‑060/61/63.
* GDD 3 "Resources/Statuses/Noise": HEX‑021/022/041.
* GDD 4 "Cards": HEX‑030→039, HEX‑03A.
* GDD 5 "Enemies/AI": HEX‑050→053.
* GDD 6 "Grid/Targeting": HEX‑010→014.
* GDD 7 "Maps/Progression": HEX‑060→064.
* GDD 8 "UI/UX": HEX‑070→074.
* GDD 9 "Tech/CI": HEX‑000→002, HEX‑080/081.
* GDD 10 "MTDemo": HEX‑034 + M1/M2 essentials.
* GDD 11 "Known Values": Constants + acceptance in relevant tickets.
* GDD 12 "Controls": HEX‑071/072/073.
* GDD 13 "Balance v1": HEX‑092.
* GDD 14 "Test Plan": HEX‑090/091 + CI gates.
* GDD 15 "Risks": HEX‑001/090/091/081 mitigations.

---

## H. Prompts for Implementation Agents (copy/paste skeletons)

For each ticket HEX‑XYZ, provide to a coding agent:

* "Implement ticket HEX‑XYZ per spec in the backlog. Do not change public interfaces. Add unit tests under tests/<area>. Ensure Editor Smoke test passes. Emit CombatLog events for state changes. Keep strict typing. Provide a brief CHANGELOG entry."

For a test agent:

* "Author GUT tests for HEX‑XYZ. Validate preview‑vs‑resolve parity and determinism. Add fixtures if grid/LOS‑related."

---

## I. Future Backlog (PoE‑style Affixes & Crafting, StS‑style Growth)

1. **Map Affixes (PoE)** — node modifiers (e.g., +1 elite, +cover tiles, enemies gain Armored). UI shows affixes with risk/reward.
2. **Boss Maps with Guaranteed Rune** — rare map nodes guaranteeing a rune drop to power a card or socket system.
3. **Sockets & Runes (PoE‑inspired)** — items/runes that slot into cards to alter effects (e.g., convert Strike to cone, add +1 Burn). Reserve mechanics for auras.
4. **Card Rarities & Pools (StS)** — curated common/uncommon/rare pools per node type and ascension level.
5. **Relic‑like Systems** — persistent passives for a run (e.g., +1 AP on turn 1).
6. **Advanced AI (Alina)** — hazard evaluation, trap avoidance, flanking bonuses.

Each of the above will get its own data contracts and ticket sets when green‑lit.

---

## J. Risks & Mitigations (Expanded)

* **Rule mis‑previews** → Golden fixtures + parity tests (HEX‑090); preview source of truth mirrors resolver.
* **AI indecision/perf** → Action cooldowns, tie‑breakers, capped search; intent debouncing (HEX‑052).
* **Save corruption** → Atomic writes + backups (HEX‑080).
* **Determinism drift** → Seeded RNG in one place (RunState); replay check (HEX‑081).

---

## K. Today's Start Plan (Do This First)

1. HEX‑000, HEX‑001, HEX‑002 in one PR (project opens, CI builds web).
2. HEX‑010, HEX‑011, HEX‑012 in a second PR (grid math). Add HEX‑090 tests.
3. HEX‑020, HEX‑021, HEX‑034 to ship MTDemo parity.

Ping PM for the next batch once these land.