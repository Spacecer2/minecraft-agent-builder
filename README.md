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

The local working tree of `yuniko-software/minecraft-mcp-server` carries fixes
that make the agent behave like a careful human player. Tracked as GitHub issues:

- #3 Fix pathfinding & movement truth (Movements config, fly-to gamemode guard,
  actual-result positions, validateWorldY, honest move-in-direction).
- #4 Fix tool truth (real crafted names, exact-match item lookups,
  verify-after-write, get-world-state / scan-area / verify-block tools).
- #5 Fix chat & stdio protocol safety (removed stdout filter, event-driven
  wait-for-chat, from / onlyMentionsMe chat filters).

## Building ethos

Building is efficiency. Read the site before you place a block, keep a path,
scaffold to reach height and tear the scaffold down, construct foundation →
walls → roof → detail, and verify every block. Never wall yourself in.
