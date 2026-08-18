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

- **Permission-layer delegation brain**: `run-goal` runs deterministic hardcoded defaults per goal, checks circumstances, and returns concise verified reports. Compound goals ("get some crops and drop me some bread" → harvest → make bread → deliver). Uses existing items first; crafts only when absent; escalates to the front brain only at the deepest blocked state via `BLOCKED: <reason>. Context: <json>.` NEED_DECISION. Reasoning intensity scales with circumstances (0 = no LLM, 1-2 = deterministic fallback, 3 = NEED_DECISION).

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

## Building ethos

Building is efficiency. Read the site before you place a block, keep a path,
scaffold to reach height and tear the scaffold down, construct foundation →
walls → roof → detail, and verify every block. Never wall yourself in.
