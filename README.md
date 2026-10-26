# Azeroth DB agent skill

An agent skill that answers WoW Forever questions about items, crafting recipes, spell
ranks, talents, races, quests and dungeons from the public data of
[Azeroth DB](https://wowdb.wiki), and cites the exact page of every answer in the player's
language.

`SKILL.md` is the skill. Copy this folder into your agent's skills directory as
`wowdb-wiki`, or upload it as a zip containing the folder `wowdb-wiki/SKILL.md`.

- Ask: what does one Elixir of Minor Defense need, at which levels does a Mage learn each
  Fireball rank, which races can I play.
- The skill reads `https://wowdb.wiki/llms.txt`, the site's search index and its record data.

The same lookups are available as an MCP server: [wowdb-mcp](https://github.com/azeroth-db/wowdb-mcp).
