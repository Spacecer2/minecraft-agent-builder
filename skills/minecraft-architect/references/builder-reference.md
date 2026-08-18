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
