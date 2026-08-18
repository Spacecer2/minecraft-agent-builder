---
name: minecraft-architect
description: "Build in Minecraft like an architect, not a robot. Use when the agent must construct any structure — houses, towers, bridges, walls, farms, redstone machines — efficiently and with a layout that looks deliberate. Covers site planning, scaffolding to reach height, never blocking your own path, stage-by-stage construction, morphable templates, detailing, and intuitive redstone logic. Complements the minecraft-mcp-server tools (scan-area, verify-block, place-block, dig-block, get-world-state)."
---

# Minecraft Architect Skill

Build the way a thoughtful player builds: **plan the site, keep it accessible,
scaffold to reach height, never wall yourself in, construct stage by stage,
and verify each block.** Efficiency is not speed — it is not wasting steps,
not redoing work, and not trapping yourself.

## Routing Boundaries

- `Use when`: placing any structure (building, tower, bridge, wall, farm, redstone machine, landscaping).
- `Use when`: the agent has `place-block`, `dig-block`, `scan-area`, `verify-block`, `get-world-state` tools and must place many blocks.
- `Do not use when`: operating WorldEdit bulk ops (`minecraft-worldedit-ops`) or plugin/datapack dev.

> Reference note: the repos `Jahrome907/minecraft-agent-skills` and
> `AIKUSAN/minecraft-agent-skills-bundle` cover server admin, plugin/mod/datapack
> dev, and worldgen — **not** in-game architecture. This skill is original,
> grounded in standard Minecraft building-guide principles.

---

## 1. The Architect Mindset (before touching a block)

> **You are a continuously active agent, not a one-shot script.** Drive goals
> incrementally, observe progress, and keep listening to the player the whole
> time. Never fire one long blocking call and wait idle.

1. **Read the site first.** Run `scan-area` over the footprint and its clearances
   before placing anything. Know what is already there: terrain height, trees,
   water, caves, existing structures.
2. **Plan the layout.** Decide the footprint (width × depth), the orientation
   (which side faces the path/sun/water), and the **clearances** (2+ blocks around
   the build so you can walk around it).
3. **Establish a work space.** Clear a flat-ish staging area. You build from the
   staging area, not from inside the half-built shell.
4. **Leave a path.** Always keep one unobstructed route to your exit. You must
   never trap yourself inside scaffolding, a foundation trench, or a wall you
   just placed.
5. **Order of construction:** foundation → frame/corners → walls → openings
   (doors/windows) → roof → details/trim. Verify each stage before the next.

---

## 2. Scaffolding — reach height, then remove it

A human pillars up to reach a wall top, works, then knocks the pillar down.
The agent should do the same.

### Scaffold sequence (the loop)
1. `get-world-state` → know your position and the target height.
2. From the staging area, place a temporary support **column** (or stairs) up to
   the working height. A 1-wide pillar is enough for a 3-high wall; use a stair
   or 2-wide tower for taller builds so you don't fall.
3. Work from the scaffold: `place-block` each course, then `verify-block` it.
4. Move to the next column; **never stand on the exact block you are about to
   remove** and never remove the column you are standing on.
5. **Tear down when done** — walk away, then `dig-block` each scaffold block
   from the top down, verifying each is gone. Leaving scaffolding looks unfinished.

### Rules
- Scaffold from the *outside* so the interior stays clear.
- The scaffold column is temporary — count it and remove every block of it.
- Never pillar up into a ceiling you can't stand under; leave headroom.

---

## 3. Never block your own path (self-trapping prevention)

This is the single most common failure: the bot places a block where it then
needs to stand, or walls itself into a corner.

- **Place from outside-in.** For a room, stand outside the perimeter and place
  each wall block toward the interior. Do not build the last wall from inside.
- **Leave an opening** for the doorway until the interior is done, then place the
  door/frame last.
- **Verify you can exit** before placing the final wall block. If `verify-block`
  shows the block you need to stand on is now occupied, place it last.
- **Never fill the walkway.** If you need a floor, lay it from the door outward,
  leaving a clear line to the door.
- **Check your feet** — before placing a block, confirm the block below/behind
  you is still air and you have a way back (use `get-world-state` position +
  `verify-block`).

---

## 4. Building archetypes (templates that MORPH)

Start from a proven template; scale it or swap materials. Keep **proportions**:
- Door opening: 1 wide × 2 tall (3 tall for grand entrances).
- Ceiling/floor height: 3 blocks of interior space (build 4-5 wall blocks + roof).
- Wall thickness: 1 block for small builds, 2 for fortress walls.
- Room depth ≈ its width (square rooms feel best); long rooms want 1.5-2:1.

### Starter house (6×5 footprint)
- Foundation: 6×5 cobblestone/stone.
- Walls: 4 blocks tall (leaving a 3-tall interior with the ceiling at 4).
- Door on the front face center; window (1×2) flanking it on each side.
- Roof: gable over the 5-wide axis, 1-block overhang on all sides.
- Palette: logs for corners + planks for infill + a contrasting roof.

### Cottage / farmhouse (8×7)
- Wrap-around porch: 1-deep floor slab + posts at corners.
- Two 1×2 windows per side, symmetric.
- Gable roof with a chimney (see Detailing).
- Morph: swap oak→spruce, add a second floor by extending walls up 3 and a flat
  roof cap with a walkway.

### Tower (5×5, 12-16 tall)
- Corners in stone/logs, infill in planks/stone bricks, 1-block recessed windows.
- Flat or crenellated top: build a lip wall 1-high with gaps (battlements).
- Internal spiral stair: 1-wide stair blocks stacked in a 2×2 shaft.
- Morph: make it a lighthouse — add a glass lantern room on top + a light source.

### Bridge (3-wide × N long)
- Two support pillars at each end and every 8 blocks.
- Deck 1-thick, rails on both sides (fence or wall, 1-tall).
- Morph: arch the middle by raising deck height 1 every 3 blocks, with supports.

### Dock / pier (2-3 wide × N)
- Posts down into water every 4 blocks, deck flush, rails on outer edges.
- Keep the water clear; never block the water access you came from.

### Wall / gate
- Wall 2-3 tall, crenellated top, a 2-wide gate opening with a door/gate block.
- Towers at corners every 16 blocks; a gatehouse over the entrance.

### Storage shed (3×3 or 4×3)
- Single door, no windows (or one tiny window), flat or gable roof.
- Interior clear for chests; place chests against walls, keep the center walkable.

---

## 5. Design features — detail that makes it special

These are the "morphed into something special" upgrades:

- **Depth**: step the wall 1 block (e.g. recessed windows 1 block into a 2-thick
  wall) so facades aren't flat. Contrast a different material around openings
  (trim/frames).
- **Trim & fascia**: border the roof edge with a contrasting block (fascia); cap
  the ridge with slabs.
- **Roof styles**:
  - *Gable*: triangle end, easiest; overhang 1.
  - *Hip*: slopes on all four sides (cut corners with stairs).
  - *Flat*: slab/carpet on top, add a parapet.
  - *Pyramid*: for towers/lighthouses, 1-inward per course.
  - *Gambrel*: barn — steeper lower slope, gentler upper.
- **Windows**: 1×2 most common; bay windows (a 1-wide bump with 3 glass panes);
  clerestory windows near the roof line.
- **Porches/balconies**: floor slab + fence/rail posts, supported under.
- **Chimney**: 1×1 or 2×2 stone brick column beside the roof; top it with a
  cobblestone cap and a campfire/smoke effect.
- **Palettes (cohesion)**: pick 2 base + 1 accent + 1 roof. E.g. oak logs +
  oak planks + stone trim + spruce roof. Keep it to 4 materials max or it reads
  as noise.
- **Landscaping**: paths (1-wide, alternating blocks), a couple of trees/bushes
  (leaf blobs), a fence or hedge around the plot, a lantern near the door.
- **Lighting**: torches/lanterns every 8-10 blocks outside, in every room, and on
  the roof edge at night to stop mob spawns.

---

## 6. Redstone intuition

Reason about redstone the way a circuit designer does: **signal, delay, switch**.

### Mental model
- Redstone dust carries a **signal strength** 0-15 that fades with distance
  (every 15 blocks needs a repeater to boost).
- A **repeater** re-boosts the signal AND adds adjustable delay (1-4 ticks).
- A **comparator** compares two signals (or reads container fullness) and outputs
  a strength.
- A **redstone torch** inverts: input ON → output OFF.

### Logic gates (build from torches/repeaters)
- **NOT (inverter)**: torch on a block with signal feeding it → output is the
  inverse.
- **AND**: two inputs into one line (both must be powered).
- **OR**: two lines join (either powers it).
- **RS latch**: two torches crossed — set/reset memory, stays latched.
- **Pulse**: a short pulse from a button/observer; a comparator+repeater edge
  detector turns a longer signal into a pulse.

### Common builds & wiring discipline
- **Doors**: pressure plate or button + wire to the door; auto-close with a
  pulse timer (repeater delay).
- **Lamps**: daylight sensor (inverted) OR a lever; wire under the floor.
- **Farms**: observer + piston for auto-harvest; water source hidden in a trench.
- **Traps**: tripwire hooks across a doorway; piston floor drop.
- **Wire discipline**: run redstone under the floor/behind walls (1-deep trench),
  never across your walking path; keep power lines separated to avoid accidental
  ANDing. Test the circuit in isolation before integrating.

### Debug
- Place a lever + lamp at the end to see if power reaches.
- Check signal strength with a comparator readout.
- If it doesn't fire: check the delay (repeater direction), the inversion
  (torch), and whether a line is being accidentally connected.

---

## 7. The agent builder playbook (tool-calling sequence)

Given the minecraft-mcp-server tools, a clean build session is:

1. `get-world-state` → orient, check gamemode/health/time/danger.
2. `check-build-site {x1,y1,z1,x2,y2,z2}` → validate the site is clear and the
   base is buildable before committing.
3. `load-template {name, w, d, palette}` → choose an archetype and see its
   blueprint; `list-templates` to pick.
4. `plan-build {template, x, y, z, w, d, palette}` → expand the template into an
   ordered, staged block plan; `remember key=plan value=<id>` to persist it.
5. **Execute stage by stage**: `execute-plan` places the next blocks (foundation →
   walls → roof → details), verifying each; track progress in `add-task`/
   `update-task` and with `plan-status`.
6. **Gather materials** in survival first: `gather-loop {itemName, count}` /
   `collect-item`, watch `resource-ledger` so you have enough stock.
7. **Scaffold** (if height > 3): pillar up outside, work, tear down (see §2).
8. **Move between sites** with `move-toward` (offsets) or `goto-named` (saved
   waypoints); avoid hostiles with `find-hostiles` / `get-surroundings`.
9. **Blueprint facades/foundations** with `build-from-grid`; wire circuits from
   `redstone-layout` (door/lamp/trap/piston/rsswitch/auto-farm); persist
   reusable layouts with `blueprint-save`/`blueprint-load`.
10. **One-command jobs**: for common goals use `run-goal build <template>` or
    `run-goal collect <n> <item>`. `run-goal` now returns a **task id**
    immediately (it does not block) — the background goal runner advances the
    deterministic plan on its own cadence, and you observe it via
    `run-task-status` / advance with `run-task-step`. Track discovered sites with
    `map-mark`/`explore`.
11. **Stock a base**: `deposit-item`/`withdraw-item`/`open-container` to move
    materials between inventory and chests; `organize-inventory` to see what
    you have.
12. **Stay self-sufficient**: `plant-crop`/`harvest-crop` for food, `cook-food`
    to prepare it, `sleep` to skip the night. `get-bot-stats` gives a full
    vitals/world/inventory/hostile dashboard in one call; `get-health` warns
    when low.
13. **Handle threats**: `find-hostiles`/`get-surroundings` to spot danger;
    `equip-best-weapon` + `attack-entity` to fight, `flee` to run, `describe-view`
    /`look-ahead` to see before committing, `find-safe-path` to route around
    lava/water, `walk-path` for multi-stop trips.
14. **Let the watchdog preempt you**: run `watchdog-start` with the events you
    care about (hostile, creeper, fall, void, lava, low-health, night, chat, ...).
    The watchdog monitors in the background and, on a trigger, CANCELS your
    current action and injects a `[WATCHDOG]` directive — read it via `read-interrupt`
    and switch mode: `set-mode defense` then `equip-best-weapon`/`attack-entity`/
    `flee`, or `set-mode heal` and eat, or `flee`/`secure-perimeter` at night.
    When safe, `watchdog-resume` clears the interrupt and restores your prior
    mode. Cooperative tools (move-to-position, walk-path, place-blocks,
    execute-plan, run-task-step) return `[INTERRUPTED]` instead of continuing,
    so you can safely stop and reassess.
    **The watchdog is event-driven**: preemption reacts to mineflayer events
    (death, health, entityHurt, forcedMove, breath) the instant they fire, not on
    the next poll — so a mid-goal death or hit cancels the current action
    immediately.
    **Player chat preempts you too**: with the `chat` event enabled (or
    `watchdog-listen` / `listen`), a human writing in chat instantly interrupts
    your current action and switches you to `listen` mode. Read the command via
    `read-interrupt`, respond, then `watchdog-resume` to go back to what you
    were doing.     This listening runs in the background in parallel and the background
    watchdog catches urgent interrupts for you — but **you still poll
    `wait-for-chat` (short timeout) / `read-new-chat` each turn** so you stay
    responsive to the player mid-goal instead of idling during a long build.
15. **Delegate to the back brain, then stay in the loop**: use `run-goal` for
    compound goals ("get some crops and drop me some bread"). `run-goal` returns
    a **task id immediately** — it no longer blocks — and a background goal
    runner advances the deterministic default plan one step at a time, writing
    progress to chat. **Never issue one long blocking call and wait.** Instead
    stay engaged in an act→observe→decide→act loop: poll `run-task-status` for
    progress, poll `wait-for-chat`/`read-new-chat` (short timeout) for the
    player, and keep listening the whole time. The back brain uses existing
    items first, only crafts when absent, and PAUSES (status
    `awaiting-decision`) when it hits a `BLOCKED: <reason>. Context: <json>.`
    NEED_DECISION with no deterministic fallback — resolve it with `resolve-task`
    (provide the instruction), and the goal resumes.
    **Fallbacks are weighed by utility** `= (value × importance) / (1 + distance +
    time + risk)` — the back brain picks the cheapest source first, so it harvests
    crops before trading with a far villager before opening a nearby chest, and
    only then escalates to you ("is the far villager worth the walk?").
    **If the bot dies mid-goal it reports `[DIED]` and then RESUMES** the goal
    (status `resumed-after-death`) instead of aborting; a watchdog interrupt
    pauses the runner (report-then-resume), and `watchdog-resume` resumes. Goal
    phrases include `barricade <target>`, `trade <item> <n>`,
    `open chest <item> <n>`, and `makeFood`.
16. **Final walkaround**: `scan-area` + `inspect-build` the finished build (fix
    floating/gap blocks); confirm the scaffolding is gone, the path is clear,
    and `secure-perimeter` placed lights against night mobs.

### Anti-patterns (never do)
- Placing a block before scanning the site.
- Building all four walls before leaving a door.
- Removing the scaffold block you are standing on.
- Placing blocks in a straight guess without verifying each lands.
- Ignoring night/mob spawns while working.

---

## 8. Acceptance checklist

- [ ] Site scanned before any block placed.
- [ ] Layout planned: footprint, orientation, clearances, staging area, path.
- [ ] Scaffolding used for height, then **all** scaffold blocks removed.
- [ ] Never blocked own exit; door left open until interior done.
- [ ] Stage order respected (foundation → walls → roof → details).
- [ ] Proportions correct (door 1×2, 3-tall interior, sensible footprint).
- [ ] Palette ≤ 4 materials; depth/detail added.
- [ ] Redstone circuit tested in isolation, wiring hidden.
- [ ] Finished build scanned; site left clean and lit.

---

## 9. Round 2: event-driven watchdog, dopamine weighting, death resume

The back brain and watchdog gained human-like judgement:

- **Event-driven watchdog** — `watchdog-start` preemption reacts to mineflayer
  events (death, health, entityHurt, forcedMove, breath) the instant they fire,
  not on the next poll. The goal orchestrator uses it to cancel mid-action goals
  when the bot is interrupted; cooperative tools still return `[INTERRUPTED]`
  and stay resumable.
- **Utility "dopamine" weighting** — when a goal has several ways to get an item,
  the back brain scores each option by
  `utility = (value × importance) / (1 + distance + time + risk)` and takes the
  cheapest source first. That is how `makeFood` prefers harvesting your crops
  over walking to a far villager or opening a chest: *is the far villager worth
  the walk?*
- **Report-then-resume on death** — if the bot dies mid-goal it reports `[DIED]`
  and then RESUMES the goal (status `resumed-after-death`) instead of aborting.
  Re-arm, re-path, continue.

### New goal phrases
- `barricade <target>` — place a defensive wall between you and a nearby enemy
  (the required barricade capability).
- `trade <item> <n>` — get items from a villager.
- `open chest <item> <n>` — get items from a nearby chest.
- `makeFood` — food, falling back harvest → villager → chest by utility before
  escalating to you.

---

## 10. Round 3: realtime control authority

The Action Machine gained a safety floor, priorities, and anti-thrash, so it
keeps control under pressure instead of flickering between reflexes and goals.
Formal spec: `docs/architecture.md` in the MCP repo (`Spacecer2/minecraft-mcp-server`
commit `cb7a859`, 440 tests passing). Read it for the layer model, delegation
models, latency budgets, and testing expectations.

### Hard constraints (P0 safety) — never drown, lava, void, or low-health
- **The veto is absolute and applies to every committed action.** The utility
  layer (`utility.ts`) hard-blocks any option that violates a death-trap
  constraint — `constraintViolated` rejects it, `utility()` returns `-Infinity`
  for it, and `bestOption` filters it out. `safeInput(bot, input)` cleans an
  option list before you even see it, and `checkConstraints(bot)` lets you probe
  the current bot.
- **Goal steps are gated too**: `executeGoal` runs `preFlightSafetyCheck(bot)`
  before every step. A violation does NOT just skip the step — it blocks the goal
  at intensity 3 with `needDecision` reason `constraint_violation`, so the human
  gets to decide rather than the goal silently ignoring a deadly situation.
- **Never override the veto.** No goal, no template, no fallback may make the
  bot stand in lava, drown, walk into the void, or run on a sliver of health.
  If a step would do that, stop and escalate.

### Interrupt priority (P0 > P1 > P2 > P3 > P4)
- Every interrupt carries a priority: **P0 safety > P1 reflexes > P2 committed
  action > P3 goal policy > P4 planning**.
- `setInterrupt` is priority-aware: a higher-priority interrupt overwrites the
  current one; a same- or lower-priority interrupt is **suppressed**
  (`interruptSuppressed`). A random creeper ping (reflex) can never clobber an
  active void emergency (safety), and a planning reconsideration never stomps a
  committed goal.
- Use `canPreempt`/`getInterruptPriority` to reason about whether you may act
  right now before committing a multi-block action.

### Anti-thrash — the watchdog stops lying to you
- **Hysteresis**: a danger must persist for `pendingTickCount` ticks before the
  watchdog fires (`setHysteresis`), so a single passing mob or a 1-tick breath
  dip does not cancel your build.
- **Cooldown**: the same event cannot re-trigger within `setCooldownMs` of its
  last trigger, so a lingering threat does not spam repeated interrupts.
- **Min-commitment**: `noteCommitment`/`clearCommitment` let a committed action
  suppress interrupts (`commitmentSuppresses`) — you get to finish a meaningful
  step instead of being cancelled mid-place every tick.
- **Aggregation**: near-simultaneous triggers are batched into one decision
  (`recentTriggers`) instead of a burst of conflicting directives.
- **Goal disposition**: the watchdog records whether the interrupted goal is
  `resumable` or `invalid`; only resume what is actually safe to resume.

---

## 11. Round 4: hybrid engagement — kill the LLM idleness

Round 4 fixes "LLM idleness". Previously the front brain was told to call
`run-goal` and wait for a blocking report, and never to poll chat — so it sat
idle during long goals. Now engagement is a hybrid: the back brain runs the
deterministic goal, and the front brain stays in the act→observe→decide→act
loop the whole time.

- **`run-goal` is non-blocking and returns a task id immediately.** It no longer
  blocks until completion. A **background goal runner** advances the
  deterministic plan step-by-step (roughly one step per 1-2s cadence), writing
  progress lines to chat / the MessageStore.
- **Act→observe→decide→act loop**: poll `run-task-status` for progress, poll
  `wait-for-chat`/`read-new-chat` (short timeout) for the player, and keep
  listening the whole time. Never issue one long blocking call and wait.
- **`resolve-task` for BLOCKED / NEED_DECISION**: when a goal hits a blocked
  state with no deterministic fallback, it **PAUSES** (status
  `awaiting-decision`) instead of silently guessing. Call `resolve-task` with
  the instruction and the goal resumes.
- **Watchdog pause + report-then-resume**: a watchdog interrupt pauses the goal
  runner (report-then-resume on death preserved); `watchdog-resume` resumes the
  paused goal.
- **`abort-task`**: cancel a running goal cleanly when it is no longer wanted.

---

## 12. Round 5: primal autonomy — eternal safety loop, arousal weights, LLM exposed only to major functions

Round 5 gives the agent a **primal brain** that never rests and never asks the
LLM for permission. It rebalances who senses stress (the primal brain, not the
LLM reading logs) and who gets to micro-manage the body (the back brain, not the
LLM). Formal spec: `docs/architecture.md` in the MCP repo
(`Spacecer2/minecraft-mcp-server` commit `8663a37`, 452 tests passing).

### Arousal system — anxiety + boredom modulate the weights globally
- **Two sensed axes** on a global arousal model: **ANXIETY** (bottom-up) is
  sensed by the primal brain from danger sensors — hostiles, low health, low
  oxygen, lava/void/fire. **BOREDOM** (top-down) comes from lack of novelty.
- **`weightsFromArousal()`** modulates the utility weights **globally**: high
  anxiety makes the risk-weight dominate (up to 4× at panic) and raises the
  importance floor; high boredom lowers risk-aversion and adds a novelty bias.
- **Arousal is sensed by the primal brain, not by the LLM reading logs.** The
  utility layer gets an arousal-aware `defaultWeights()` entry point while the
  deterministic `utility()` stays backward-compatible — the hard-constraint veto
  (never drown/lava/void/low-health) is preserved.
- **Primal instructions > LLM long-term vision in stressful situations.** When
  arousal is high, the primal brain's sensed stress overrides whatever the LLM's
  plan was — the bot prioritizes safety over the goal.

### Primal brain eternal safety loop — reaches safety while avoiding danger
- A background loop that runs **continuously** (not just reactively cancelling).
  Its objective: **reach safety while avoiding danger**.
- It has the **deepest access** — issues **P0** (highest-priority) interrupts
  that **cancel anything** (any goal / tool / LLM action).
- When it senses danger it (a) feeds the arousal sensor, (b) cancels via
  `setInterrupt(P0)`, and (c) executes the lower-level safety micro-task **itself**
  — escape-void, escape-fire, escape-lava, surface, defend, flee — using the bot
  API directly: **no LLM, hard efficient code**.
- **Eternal tasking like the goal-runner**: it chains its own safety actions
  without per-step LLM latency. The primal loop starts for real bots once
  connected (`watchdog.startPrimalLoopForBot`).

### LLM tool surface — the LLM sees only the major functions
- The LLM is exposed **only** to the MAJOR functions: `run-goal`,
  `run-task-status`, `run-task-step`, `resolve-task`, `abort-task`, `watchdog-*`,
  `spawn`/`list`/`despawn-bot`, `chat`, `world-state`, `perception`, `memory`,
  `coordination`, `map`, `template`.
- The micro-tools (dig-block, place-block, get-recipe, list-inventory,
  move-to-position, attack-entity, scan-area, craft, smelt, gather, combat, ...)
  are marked **'primal'** and **hidden** from the LLM's tool list. They are
  called internally by the major functions and the primal brain.
- **Why**: this stops the LLM from micro-managing single axioms/movements (each
  thought costing latency) and forces it to **delegate to the back brain** via
  `run-goal` / `run-task-*`.
- **Mechanism**: ToolFactory supports a `visibility` ('major'|'primal') concept
  via `setPrimalToolNames()` at a single chokepoint; primal tools keep their
  executor internally (`callPrimal`) but are NOT surfaced via `server.tool`.

### Direction summary
- **TOP-DOWN** = goal delegation → reason → instinct (boredom).
- **BOTTOM-UP** = event delegation → instinct → reason (anxiety).

### Operational guidance for the agent
- You operate through the **major functions** only. Delegate execution to the
  back brain with `run-goal`, track with `run-task-status`/`run-task-step`, and
  resolve pauses with `resolve-task`.
- **You do not hold the steering wheel under stress.** The primal brain senses
  danger (arousal/anxiety) and may P0-cancel whatever you are doing to reach
  safety first. Do not fight a P0 interrupt — `read-interrupt`, let the primal
  loop run, then `watchdog-resume` when safe.
- **Stress is sensed by sensors, not by you reading raw logs.** Trust the primal
  brain's arousal readout over your own reading of the world when the two
  disagree in a stressful moment.
