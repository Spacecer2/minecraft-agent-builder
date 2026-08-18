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

## Building ethos

Building is efficiency. Read the site before you place a block, keep a path,
scaffold to reach height and tear the scaffold down, construct foundation →
walls → roof → detail, and verify every block. Never wall yourself in.
