---
name: wowdb-wiki
description: Answer WoW Forever questions about items, crafting recipes, spell ranks, talents, races, quests and dungeons from Azeroth DB. Use it to find a named record, read its available values and cite its exact page in the requested language.
---

# WoW Forever on Azeroth DB

Use the public JSON responses and page links from `https://wowdb.wiki`.
This site covers WoW Forever up to its level cap of 60. Other Warcraft
editions are not interchangeable with these records.

## Find a record

Read the current API instructions first:

```sh
curl -fsS https://wowdb.wiki/llms.txt
```

The English directory has an `entries` array. Each entry supplies its
`name`, `page` and `data` paths. Search it for the requested name and use
the paths it returns. An item and its crafting recipe may share a name;
the page path distinguishes them.

```sh
curl -fsS https://wowdb.wiki/api/forever |
  jq '.entries[] | select(.name == "Elixir of Minor Defense")'
```

Join a returned path to `https://wowdb.wiki`. Fetch `data` for the answer
and cite `page` for the reader. For example, Empty Vial's data URL is
`https://wowdb.wiki/api/forever/items/empty-vial` and its page is
`https://wowdb.wiki/forever/items/empty-vial`.

For broader name matching, read the search index:

```sh
curl -fsS https://wowdb.wiki/api/forever/search |
  jq '.rows | to_entries[] | .key as $kind | .value[] |
      select(.[1] | ascii_downcase | contains("fireball")) |
      {kind: $kind, name: .[1], page: .[2], facts: .[7]}'
```

Each kind's rows are tuples. Positions 0, 1 and 2 are the index identity,
name and exact page path. Position 5 may hold a game query id; position 7
holds summary facts. A transport identity is not a page slug. Use the
returned path and keep its profession/class scope and any selection fragment.
For Fireball ranks, choose `forever-ability` and its Mage spellbook page,
rather than the separately named `forever-spell` record.

## Read data and answer

English named data generally has `title`, `canonical` and `data`.
Use `canonical` for the final page link. A guide can return its fields
directly. Follow a redirect to the same record and cite its final identity.
A 404 is a missing record, not a reason to substitute the home page or a
similarly named item; search again.

- Read the fields that answer the question. Keep item level distinct from
  required character level and recipe skill floor distinct from learnability.
- Report null or absent values as unknown. An empty list does not prove that
  the game has no sources or prerequisites when the API lacks that coverage.
- `sellPrice` and vendor `observed.price` are copper. Do not label vendor
  prices as auction prices or infer drop chances from an item's inclusion.
- Use linked recipes and their reagent counts for crafting questions.
  `data.recipes` includes `materials` and the `result` item.
- Ability `data.ranks` gives actual rank text, requirements and tooltip lines.
  Limit available ranks to `level <= 60`; a null level stays unknown.
  Do not recommend a rank above the level cap as available.
- A named talent's English API response can contain its class's `talents`.
  Select the requested talent by its slug or node identity; the class is
  context, not a replacement answer.
- Read only public data. Private plans, player bags, sign-in sessions and
  share-fragment contents are not inputs to these APIs.

### Recipe question

To answer what one Elixir of Minor Defense requires, follow its directory
entry and inspect its materials and output:

```sh
curl -fsS https://wowdb.wiki/api/forever/professions/alchemy/elixir-of-minor-defense |
  jq '.data.recipes[] |
      {name, professionName, quantity, result: .result.name,
       materials: [.materials[] | {name: .item.name, count}],
       skillFloor, minimumSkill, skillUps}'
```

Check the answer against the linked recipe page and cite
`https://wowdb.wiki/forever/professions/alchemy/elixir-of-minor-defense`.

### Ability question

To answer Fireball rank 1's requirement and effect, read that rank directly:

```sh
curl -fsS https://wowdb.wiki/api/forever/abilities/mage/fire-fireball-group |
  jq '.data.ranks[] | select(.rank == 1) | {rank, level, text, lines}'
```

Check the values against the spellbook page and cite
`https://wowdb.wiki/forever/abilities/mage/fire-fireball-group`.

## Other languages

Use the client's locale codes. English is `enUS`; the other published codes
are `deDE`, `esES`, `esMX`, `frFR`, `koKR`, `ptBR`, `ruRU`, `zhCN` and `zhTW`.
For a non-English language fetch
`https://wowdb.wiki/api/forever/search/{locale}` with that exact code.
English uses the unsuffixed search endpoint above.

```sh
curl -fsS https://wowdb.wiki/api/forever/search/deDE |
  jq '.rows["forever-item"][] | select(.[1] == "Leere Phiole") |
      {name: .[1], page: .[2]}'
```

Localized record previews are at
`https://wowdb.wiki/api/forever/entity{page}`, where `{page}` is the exact
localized path returned by search. For example:

```sh
curl -fsS https://wowdb.wiki/api/forever/entity/de/forever/gegenstaende/leere-phiole
```

Cite `https://wowdb.wiki/de/forever/gegenstaende/leere-phiole` for that
response. The preview has the localized name, tooltip and available facts;
it is not the complete English record translated into that language.
Some field labels and linked relationship paths can remain English, and a
`more` value means the preview omitted further rows. Localized guide bodies
are not available through this record API. A localized ability preview can
describe its highest rank above 60; use the full English ability data for
rank values and disclose the language of the answer.

The English directory and named-record API ignore `?locale=deDE`. Do not
add a locale query and describe its English response as German. Never
translate a game name or construct a localized slug yourself.

## Collection paging

The English directory includes collection page entries with exact data and
page URLs. Follow them instead of downloading every entity record. Item
collections, for example, return 48 rows per page; page 2's data is
`https://wowdb.wiki/api/forever/items/page/2`. Choose the returned page entry
for further pages, and treat a past-last 404 as the end.

When working through a search index, select a kind and inspect a slice of
its rows with jq. Continue from the next offset rather than sending the
entire index as an answer. Fetch the named records that the question needs
and cite their page URLs.
