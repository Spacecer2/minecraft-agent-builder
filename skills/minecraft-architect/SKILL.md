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
    `run-goal collect <n> <item>`, then advance with `run-task-step` /
    `run-task-status`. Track discovered sites with `map-mark`/`explore`.
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
    **Player chat preempts you too**: with the `chat` event enabled (or
    `watchdog-listen` / `listen`), a human writing in chat instantly interrupts
    your current action and switches you to `listen` mode. Read the command via
    `read-interrupt`, respond, then `watchdog-resume` to go back to what you
    were doing. This listening runs in the background in parallel — you never
    need to poll for it.
15. **Delegate to the back brain, don't hand-drive**: use `run-goal` for compound
    goals ("get some crops and drop me some bread") — the back brain runs the
    deterministic default plan, checks circumstances, and returns a concise
    verified report (`harvested 4 wheat → made bread x1 → delivered bread x1 to
    Spacecer2`). It uses existing items first, only crafts when absent, and
    escalates to you (the front brain) only at the deepest blocked state with a
    `BLOCKED: <reason>. Context: <json>.` NEED_DECISION — direct it or call
    `run-task-resume`. Reasoning intensity scales with circumstances: cheap
    defaults, deeper reasoning only when reality demands.
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
