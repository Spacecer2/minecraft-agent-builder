# Minecraft Architect reference

Supporting schematics and lookup tables for the `minecraft-architect` skill.
Read this alongside `SKILL.md`; it holds the concrete numbers and layouts the
skill refers to, so the agent does not have to hold them in working memory.

## Material palette cheat-sheet

| Role   | Blocks (pick ≤ 4 total)                      |
| ------ | -------------------------------------------- |
| Base   | oak logs, spruce logs, stone, stone bricks    |
| Infill | oak planks, spruce planks, cobblestone, brick |
| Accent | stone, diorite, andesite, terracotta, quartz  |
| Roof   | spruce planks, dark oak, deepslate tiles, bricks |
| Trim   | stairs/slabs matching infill or accent        |

Cohesion rule: **2 base + 1 accent + 1 roof = 4 max.**

## Block dimensions to memorize

- Door opening: 1 wide × 2 tall (3 tall for grand).
- Interior ceiling height: 3 (build 4-5 wall blocks + roof above).
- Wall thickness: 1 (small), 2 (fortress).
- Step-up: 1 block free; jump clears 1.25; sprint-jump ~4 wide.
- Fall: ≤ 3 blocks safe; 4 = half heart; 23 = death. Water breaks falls.
- Mob spawn: light level 0, 24-128 blocks from player. Light every room + edge.

## Archetype footprints (from SKILL §4)

| Archetype | Footprint | Height | Key ratios |
| --------- | --------- | ------ | ---------- |
| Starter   | 6×5       | 4 walls | gable roof, door center |
| Cottage   | 8×7       | 4 walls | porch, 2 windows/side, chimney |
| Tower     | 5×5       | 12-16  | recessed windows, crenellation |
| Bridge    | 3×N       | 1 deck | pillars every 8, rails |
| Dock      | 2-3×N     | flush  | posts every 4, rails |
| Shed      | 3×3/4×3   | 3-4    | single door, clear interior |

## Roof style shapes

- Gable: triangle end, overhang 1.
- Hip: all four sides slope (cut corners with stairs).
- Flat: slab cap + parapet.
- Pyramid: 1-inward per course (towers).
- Gambrel: steep lower, gentle upper (barns).

## Redstone cheat-sheet

| Gate/Part      | Build                                             | Use                 |
| -------------- | ------------------------------------------------- | ------------------- |
| NOT (inverter) | torch on block, signal in → inverted out          | reverse a sensor    |
| AND            | two inputs to one line                            | both conditions     |
| OR             | two lines join                                    | either condition    |
| RS latch       | two crossed torches                               | set/reset memory    |
| Repeater       | boost + delay 1-4 ticks                           | extend/retime       |
| Comparator     | compare / read container fullness                 | inventories, edges  |
| Pulse          | observer, or comparator+repeater edge detector    | short triggers      |

Wiring: run dust in a 1-deep trench under floors/behind walls. Never cross your
own walking path with exposed redstone. Keep power lines separated to avoid
accidental AND. Test with a lever + lamp before integrating.

## Scaffold tear-down check

After work at height, confirm every temporary support is removed: walk away and
`dig-block` each pillar from top down, `verify-block` it is gone. Leaving a pillar
looks unfinished and blocks movement.

## Self-trapping guard

Before placing the "closing" block of any shell, confirm your exit is still open:
you can step out, the door frame is unblocked, and the block you would stand on
is air. If not, place the closing block last.

## Goal phrases (back brain, Round 2)

| Phrase                  | What the back brain does                            |
| ----------------------- | --------------------------------------------------- |
| `barricade <target>`    | Place a defensive wall to shield against a nearby enemy (barricade capability). |
| `trade <item> <n>`      | Get items from a villager.                          |
| `open chest <item> <n>` | Get items from a nearby chest.                      |
| `makeFood`              | Food; falls back harvest → villager → chest by utility. |

**Observe loop (Round 4)**: `run-goal` starts a goal and returns a task id
immediately (non-blocking) — the background runner advances the deterministic
plan on its own cadence. The front brain stays in the loop: poll
`run-task-status` for progress, poll `wait-for-chat`/`read-new-chat` (short
timeout) for the player, and resolve any `BLOCKED` / NEED_DECISION pause with
`resolve-task` (provides the instruction) so the goal resumes.

## Fallback & resilience notes (Round 2)

- **Utility weighting**: fallback options are scored
  `utility = (value × importance) / (1 + distance + time + risk)`; the cheapest
  source wins, so the bot behaves like a human deciding whether the far villager
  is worth the walk.
- **Report-then-resume on death**: a bot that dies mid-goal reports `[DIED]` and
  resumes the goal (status `resumed-after-death`) rather than aborting.
- **Event-driven watchdog**: preemption reacts to mineflayer events (death,
  health, entityHurt, forcedMove, breath) instantly, not only on a poll.

## Realtime control authority (Round 3)

Formal spec: `docs/architecture.md` in the MCP repo
(`Spacecer2/minecraft-mcp-server` commit `cb7a859`, 440 tests passing).

### Control-authority precedence
**P0 safety > P1 reflexes > P2 committed action > P3 goal policy > P4 planning.**
Higher priority overwrites; same/lower is suppressed.

### Hard constraints (P0)
`HARD_CONSTRAINTS` = drowning, lava, void, low-health. A violation means the
option is **vetoed, never degraded**: `constraintViolated` rejects,
`utility()` → `-Infinity`, `bestOption` filters, `safeInput(bot, input)` strips.
`executeGoal` → `preFlightSafetyCheck(bot)` before each step → blocked at
intensity 3 with `needDecision` reason `constraint_violation`.

### Interrupt API
- `setInterrupt` (priority-aware), `getInterruptPriority`, `canPreempt`,
  `interruptSuppressed`.
- Priorities via `InterruptPriority` enum (P0..P4).

### Anti-thrash controls
| Mechanism | Function | Effect |
| --------- | -------- | ------ |
| Hysteresis | `setHysteresis`/`pendingTickCount` | danger must persist N ticks before firing |
| Cooldown | `setCooldownMs`/`lastTriggerAt` | same event cannot re-trigger within window |
| Min-commitment | `noteCommitment`/`clearCommitment`/`commitmentSuppresses` | committed action suppresses interrupts |
| Aggregation | `recentTriggers` | near-simultaneous triggers batch into one decision |
| Goal disposition | resumable / invalid | only resume what is safe to resume |

Event → priority map: `PRIORITY_BY_EVENT` in `src/watchdog.ts`.

## Primal autonomy (Round 5)

Formal spec: `docs/architecture.md` in the MCP repo
(`Spacecer2/minecraft-mcp-server` commit `8663a37`, 452 tests passing).

### Arousal (src/arousal.ts, src/utility.ts)
| Axis | Sense direction | Sensed from |
| ---- | --------------- | ----------- |
| ANXIETY | bottom-up | danger sensors: hostiles, low health, low oxygen, lava/void/fire |
| BOREDOM | top-down | lack of novelty |

- `weightsFromArousal()`: high anxiety → risk-weight dominates (4× at panic) +
  higher importance floor; high boredom → lower risk-aversion + novelty bias.
- `defaultWeights()`: arousal-aware entry point; deterministic `utility()` stays
  backward-compatible — hard-constraint veto preserved.
- **Arousal is sensed by the primal brain, not by the LLM reading logs.**

### Eternal primal safety loop (src/primal-brain.ts, src/watchdog.ts)
- Runs continuously with the objective **reach safety while avoiding danger**.
- Deepest access: issues **P0** interrupts that **cancel anything** (any
  goal/tool/LLM action).
- On danger: (a) feed arousal sensor, (b) `setInterrupt(P0)`, (c) execute the
  safety micro-task ITSELF (escape-void / escape-fire / escape-lava / surface /
  defend / flee) via the bot API — **no LLM**, hard efficient code.
- Chains its own safety actions without per-step LLM latency (eternal tasking).
- Starts for real bots once connected: `watchdog.startPrimalLoopForBot`.

### LLM tool surface (src/tool-factory.ts, src/main.ts)
- LLM exposed **only** to MAJOR functions: run-goal, run-task-status,
  run-task-step, resolve-task, abort-task, watchdog-*, spawn/list/despawn-bot,
  chat, world-state, perception, memory, coordination, map, template.
- Micro-tools (dig-block, place-block, get-recipe, list-inventory,
  move-to-position, attack-entity, scan-area, craft, smelt, gather, combat, ...)
  are 'primal' and HIDDEN from the LLM — called internally by major functions and
  the primal brain.
- Mechanism: `visibility` ('major'|'primal') via `setPrimalToolNames()` at one
  chokepoint; primal tools keep their executor (`callPrimal`) but are not
  surfaced via `server.tool`.

### Direction
- TOP-DOWN = goal delegation → reason → instinct (boredom).
- BOTTOM-UP = event delegation → instinct → reason (anxiety).
