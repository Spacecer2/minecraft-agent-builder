# Minecraft Agent Builder

Build in Minecraft like an architect. An LLM-controlled Minecraft agent
(mineflayer / minecraft-mcp-server) that plans sites, scaffolds to reach height,
never blocks its own path, and constructs buildings with deliberate layout —
plus intuitive redstone.

## Skills

| Skill | Use it for |
| ----- | ---------- |
| `minecraft-architect` | Planning and building structures like a human: layout, accessibility, scaffolding, never self-blocking, morphable templates, detailing, redstone intuition. |

Install: copy `skills/minecraft-architect/` into your agent host's skills folder
(e.g. `.claude/skills/`, `.opencode/skills/`, or `.agents/skills/`).

## Research note

The two upstream skill bundles (`Jahrome907/minecraft-agent-skills` and
`AIKUSAN/minecraft-agent-skills-bundle`) were reviewed. They cover server admin,
plugin/mod/datapack development, and worldgen — **not** in-game architecture.
The `minecraft-architect` skill here is original, grounded in standard Minecraft
building-guide principles.

## Companion work: minecraft-mcp-server fixes

The fork `Spacecer2/minecraft-mcp-server` carries fixes that make the agent
behave like a careful human player. Tracked as GitHub issues:

- #3 Fix pathfinding & movement truth (Movements config, fly-to gamemode guard,
  actual-result positions, validateWorldY, honest move-in-direction).
- #4 Fix tool truth (real crafted names, exact-match item lookups,
  verify-after-write, get-world-state / scan-area / verify-block tools).
- #5 Fix chat & stdio protocol safety (removed stdout filter, event-driven
  wait-for-chat, from / onlyMentionsMe chat filters).

## Builder-efficiency tools (later commits)

Beyond the fixes, the fork adds primitives that let the agent build in fewer
round-trips and navigate like a human:

- **Memory & batch**: `remember`/`recall`/`forget`, `add-task`/`list-tasks`/
  `update-task` intent ledger, `place-blocks`, `fill-area`, `place-relative`.
- **Navigation**: `move-toward`, `move-toward-bearing` (compass), `goto-entity`,
  `save-location`/`goto-named`/`list-locations` (waypoints).
- **Danger & world**: enriched `get-world-state`, `get-surroundings`,
  `find-hostiles`.
- **Coordination & safety**: `agent-share`/`agent-recall`/`agent-forget` blackboard,
  `list-bot-state`, awaited spawn, approach timeouts, post-craft verify.

## Round-2: build engine (later commits)

- **Planning & templates**: `load-template`/`list-templates` (archetypes),
  `plan-build`/`plan-status`/`execute-plan`/`abort-plan` (staged, verified build
  execution — makes the architect skill executable).
- **Perception**: `look-ahead` (raycast), `path-status` (reachability),
  `check-build-site` (validate a volume before building).
- **Gathering & economy**: `collect-item`/`gather-loop` (dig source blocks),
  `resource-ledger` (material tracking).
- **Blueprint & redstone**: `build-from-grid` (char-coded blueprint), `redstone-layout`
  (advisory designs for door/lamp/trap/piston/rsswitch/auto-farm).

## Round-3: autonomy + world + containers + QA (later commits)

- **Task runner**: `run-goal` (one-command build/gather), `run-task-status`/`run-task-step`/`abort-task`.
- **World map**: `map-mark`/`map-list`/`map-clear`/`map-nearby`, `explore` sweep.
- **Containers & interaction**: `find-container`/`open-container`/`deposit-item`/`withdraw-item`, `organize-inventory`, `activate-block`, `use-item-on`.
- **Build QA + redstone + persistence + safety**: `inspect-build`, `secure-perimeter`, `place-redstone`, `blueprint-save`/`blueprint-list`/`blueprint-load`.

## Round-4: self-sufficiency + awareness (later commits)

- **Combat**: `attack-entity`, `flee`, `equip-best-weapon`, `get-health`.
- **Farming/food/sleep**: `plant-crop`, `harvest-crop`, `feed-animal`, `cook-food`, `sleep`.
- **Motion**: `find-safe-path`, `walk-path`, `wait`, `until`.
- **Vision/awareness**: `get-bot-stats` (consolidated dashboard), `describe-view`, `interact-entity`.

## Round-5: event-driven preemption watchdog (later commit)

- **Watchdog**: `watchdog-start` (hostile/creeper/fall/void/lava/low-health/hunger/on-fire/night/drowning/inventory-full/**chat**), `read-interrupt`, `watchdog-status`, `watchdog-stop`, `watchdog-resume`, `set-mode`/`get-mode`, `watchdog-listen`, `listen`.
- **Chat-triggered preemption**: a player writing in chat instantly interrupts the current action and switches to `listen` mode, via a background listener running in parallel (no polling). Read the command with `read-interrupt`, respond, then `watchdog-resume`.
- **Cooperative cancellation**: long-running tools (`move-to-position`, `walk-path`, `place-blocks`, `fill-area`, `execute-plan`, `run-task-step`) return `[INTERRUPTED]` and stay resumable, so the watchdog can cancel and switch the agent's mode via prompt injection.

## Back brain (later commit)

- **Permission-layer delegation brain**: `run-goal` runs deterministic hardcoded defaults per goal. It is now **non-blocking — it returns a task id immediately** and a background goal runner advances the plan one step at a time, writing progress to chat; the front brain stays in an act→observe→decide→act loop (poll `run-task-status` + `wait-for-chat`/`read-new-chat`) and resolves BLOCKED/NEED_DECISION pauses with `resolve-task`. Compound goals ("get some crops and drop me some bread" → harvest → make bread → deliver). Uses existing items first; crafts only when absent; PAUSES at the deepest blocked state via `BLOCKED: <reason>. Context: <json>.` NEED_DECISION. Reasoning intensity scales with circumstances (0 = no LLM, 1-2 = deterministic fallback, 3 = NEED_DECISION).

## Round 2: event-driven watchdog, dopamine weighting, death resume, goal skills

Round 2 of the architect-style agent (MCP side: `Spacecer2/minecraft-mcp-server` commit `2f84ff3`, 427 tests passing):

- **Event-driven watchdog**: `watchdog-start` preemption now reacts to mineflayer events (death, health, entityHurt, forcedMove, breath) instantly, not just polling. The goal orchestrator uses it to cancel mid-action goals when the bot is interrupted.
- **Utility "dopamine" weighting**: the goal back brain scores fallback options by `utility = (value × importance) / (1 + distance + time + risk)` and picks the cheapest source first — harvest crops vs trade with a villager vs withdraw from a chest — before escalating to the player. Makes the agent behave like a human deciding "is the far villager worth the walk?"
- **Report-then-resume on death**: when the bot dies mid-goal, it reports `[DIED]` and then RESUMES the goal (status `resumed-after-death`) instead of aborting.
- **New goal skills**: `barricade` (place a defensive wall against a nearby enemy — the required barricade capability), `trade <item> <n>` (items from a villager), `open chest <item> <n>` (items from a nearby chest). `makeFood` now falls back through harvest → villager → chest by utility.

## Round 3: realtime control authority (later commit)

Round 3 of the architect-style agent (MCP side: `Spacecer2/minecraft-mcp-server` commit `cb7a859`, 440 tests passing). Formal spec: `docs/architecture.md` in the MCP repo — layer model, two delegation models (back brain vs goal engine), control-authority precedence, interruption semantics, escalation rules, state ownership, latency budgets.

- **Hard constraints (P0 safety)**: the utility layer hard-vetoes death traps — drowning, lava, void, low-health. `checkConstraints(bot)` detects a violation, `safeInput(bot, input)` strips any option that violates, `utility()` returns `-Infinity` for a violating option, and `bestOption` refuses to pick one. `executeGoal` runs a `preFlightSafetyCheck` before every step: a violation blocks at intensity 3 with `needDecision` reason `constraint_violation`. The agent must never dive into lava or walk off a cliff, even when a goal or the plan says to.
- **Interrupt priority (P0 > P1 > P2 > P3 > P4)**: interrupts carry a priority (safety > reflexes > committed action > goal policy > planning). A higher-priority interrupt overwrites the current one; a same-or-lower one is suppressed, so a random creeper ping cannot clobber an active void emergency.
- **Anti-thrash (hysteresis, cooldown, min-commitment, aggregation)**: the watchdog suppresses flickering triggers — hysteresis requires a danger to persist `pendingTickCount` ticks before firing; cooldown gates re-triggering of the same event within `setCooldownMs`; min-commitment (`noteCommitment`/`commitmentSuppresses`) lets a committed goal finish; event aggregation (`recentTriggers`) batches near-simultaneous triggers into one decision; goal disposition tracks whether the goal is resumable or invalid.

## Round 4: hybrid engagement — kill the LLM idleness (later commit)

Round 4 of the architect-style agent fixes "LLM idleness". Previously the front brain was told to call `run-goal` and wait for a blocking report and never poll chat — so it sat idle during long goals. Now it is a hybrid: the back brain runs the deterministic goal, and the front brain stays in an act→observe→decide→act loop.

- **Non-blocking `run-goal` + task id**: `run-goal` returns a task id immediately (no longer blocks); a background goal runner advances the deterministic plan step-by-step (one step per ~1-2s cadence), writing progress to chat / the MessageStore.
- **Act→observe→decide→act loop**: poll `run-task-status` for progress, poll `wait-for-chat`/`read-new-chat` (short timeout) for the player, and keep listening the whole time. Never issue one long blocking call and wait.
- **`resolve-task` for BLOCKED / NEED_DECISION**: a goal with no deterministic fallback PAUSES (status `awaiting-decision`); the front brain resolves it via `resolve-task` (provides the instruction) and the goal resumes.
- **Watchdog pause + report-then-resume**: a watchdog interrupt pauses the goal runner (report-then-resume on death preserved); `watchdog-resume` resumes. `abort-task` cancels a running goal cleanly.

## Round 5: primal autonomy — eternal safety loop, arousal weights, LLM exposed only to major functions (later commit)

Round 5 of the architect-style agent (MCP side: `Spacecer2/minecraft-mcp-server` commit `8663a37`, 452 tests passing) adds a primal brain that never rests and rebalances who senses stress and who micro-manages the body.

- **Arousal system** (`src/arousal.ts`, `src/utility.ts`): a global arousal model with two sensed axes — **ANXIETY** (bottom-up, sensed by the primal brain from danger sensors: hostiles, low health, low oxygen, lava/void/fire) and **BOREDOM** (top-down, from lack of novelty). `weightsFromArousal()` modulates utility weights globally — high anxiety makes the risk-weight dominate (4× at panic) and raises the importance floor; high boredom lowers risk-aversion and adds a novelty bias. `defaultWeights()` keeps the deterministic `utility()` backward-compatible (hard-constraint veto preserved). **Arousal is sensed by the primal brain, not by the LLM reading logs.**
- **Primal brain eternal safety loop** (`src/primal-brain.ts`, `src/watchdog.ts`): a background loop that runs continuously (not just reactively cancelling) with the objective **reaching safety while avoiding danger**. Deepest access — issues **P0** interrupts that **cancel anything** (any goal/tool/LLM action). On sensing danger it (a) feeds the arousal sensor, (b) cancels via `setInterrupt(P0)`, (c) executes the lower-level safety micro-task itself (escape-void, escape-fire, escape-lava, surface, defend, flee) using the bot API directly — **no LLM, hard efficient code**. Eternal tasking like the goal-runner: chains its own safety actions without per-step LLM latency.
- **LLM tool surface** (`src/tool-factory.ts`, `src/main.ts`): the LLM is exposed **only** to the MAJOR functions (run-goal, run-task-status, run-task-step, resolve-task, abort-task, watchdog-*, spawn/list/despawn-bot, chat, world-state, perception, memory, coordination, map, template). The micro-tools (dig-block, place-block, get-recipe, list-inventory, move-to-position, attack-entity, scan-area, craft, smelt, gather, combat, ...) are marked **'primal'** and **hidden** from the LLM's tool list — called internally by the major functions and the primal brain. This stops the LLM from micro-managing single axioms/movements and forces it to delegate to the back brain. Mechanism: `visibility` ('major'|'primal') via `setPrimalToolNames()` at a single chokepoint; primal tools keep their executor (`callPrimal`) but are NOT surfaced via `server.tool`.
- **Direction**: TOP-DOWN = goal delegation → reason → instinct (boredom); BOTTOM-UP = event delegation → instinct → reason (anxiety). **Primal brain instructions > LLM long-term vision in stressful situations**, and stress is sensed by the primal brain (sensors/arousal), not by the LLM reading raw logs.

## Building ethos

Building is efficiency. Read the site before you place a block, keep a path,
scaffold to reach height and tear the scaffold down, construct foundation →
walls → roof → detail, and verify every block. Never wall yourself in.
