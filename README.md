# @pipeworx/codex-gsfa

The FAO/WHO **Codex General Standard for Food Additives** (GSFA, CXS 192-1995) —
which food additives may be used in which foods, at what maximum level, under
which conditions.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1576+ live data sources.

## Tools

| Tool | Answers |
|---|---|
| `gsfa_check_use` | *Is INS 461 permitted in cooked fish, and at what level?* — the question the standard exists to answer |
| `gsfa_search_additive` | Resolve an INS number, name, or synonym to its GSFA entry |
| `gsfa_additive_provisions` | Every food category one additive is permitted in |
| `gsfa_food_category_additives` | Every additive permitted in one food category |
| `gsfa_search_food_categories` | Find a GSFA category number by name or number prefix |
| `gsfa_notes` | Resolve the numbered notes that condition a provision, or search them by wording |
| `gsfa_coverage` | What this dataset holds and how current it is |

## Two kinds of permission

GSFA grants use in two structurally different ways, and every provision returned
says which one it came from:

- **Table 1 & 2** (`provision_table: "table1_2"`) — a specific maximum use
  level, either numeric (`10,000 mg/kg`) or `GMP`, usually conditioned by
  numbered notes.
- **Table 3** (`provision_table: "table3"`) — the additive is permitted under
  good manufacturing practice with **no numeric limit**.

For methyl cellulose that is 17 Table 1/2 provisions against 61 Table 3 ones.
Treating them as one thing either invents limits the standard does not set or
hides most of where an additive is actually allowed.

## Subcategory inheritance

A GSFA provision applies to the food category named **and all of its
subcategories** — a provision on `01.4` also governs `01.4.4`. Lookups walk up
the numbering and report `inherited_from_parent` plus `matched_food_category`,
because "permitted at this level for this exact food" and "permitted because its
parent category is" are different answers.

`permitted: false` means the standard lists **no provision**. That is not the
same as an explicit prohibition, and the response says so rather than implying
a ban.

## Notes are part of the answer

Provisions are frequently qualified by numbered notes — *"for use in
ready-to-drink products only"*, *"as sulphur dioxide"*, *"on the fat basis"* —
and a maximum level is not fully specified without them. 2,590 of 15,377
provisions carry one.

Note text is **inlined on every provision** (`notes: [{id, text}]`), not left as
a bare id, because an id with no text invites a caller to read the level and
stop. `gsfa_notes` resolves ids directly or searches note wording.

The text is scraped from the note icon's tooltip on each additive page, so it
carries the same Codex session as the additive it qualifies. All 468 cited note
ids resolve.

## Auth

None. The dataset ships inside the pack.

## Freshness

Every additive result carries `cac_session`, `cac_year`, and `snapshot_date`.
Records come from archived snapshots of GSFA Online, so individual additives can
lag the newest Codex session — `gsfa_coverage` reports which sessions are
present and which additive ids could not be retrieved at all (entries whose
every archived snapshot is one of FAO's own error pages).

Regenerate with:

```bash
node --experimental-strip-types scripts/ingest-gsfa.ts
```

## Data sources

- Codex GSFA Online — <https://www.fao.org/gsfaonline/> (FAO fronts this with a
  Cloudflare managed challenge; we read Internet Archive snapshots instead —
  see `docs/codex-gsfa-plan.md` for the route and the freshness plan)
- Codex Alimentarius CXS 192-1995 — <https://www.fao.org/fao-who-codexalimentarius/codex-texts/dbs/gsfa/en/>
- JECFA specifications and evaluations, which GSFA links to for toxicological
  data — <https://apps.who.int/food-additives-contaminants-jecfa-database/>

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "codex-gsfa": {
      "url": "https://gateway.pipeworx.io/codex-gsfa/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/codex-gsfa/mcp` returns the tools in the table
above **plus the shared Pipeworx meta-tools** — `ask_pipeworx`,
`discover_tools`, `search_within`, `remember`/`recall` and the rest of the
gateway-wide set. So the tool count you see is larger than this table: a
single-pack endpoint currently lists roughly 30 shared tools alongside the
pack's own. The connection's `initialize` response states its exact scope, and
is the authoritative answer for a given day.

This is deliberate, not multiplexing by accident. The meta-tools are what let a
scoped connection answer a question this pack does not cover — via
`ask_pipeworx`, which routes across the whole catalog — without you adding a
second MCP server. There is currently no way to mount a pack endpoint without
them; if the extra schemas cost you more context than the routing is worth,
connect to the full gateway once rather than to several pack endpoints.

Or connect to the full Pipeworx gateway to get every pack's tools listed
directly, instead of just this one's:

```json
{
  "mcpServers": {
    "pipeworx": {
      "url": "https://gateway.pipeworx.io/mcp"
    }
  }
}
```

Both URLs reach the same gateway and the same 1576+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "codex-gsfa": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-codex-gsfa"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-codex-gsfa
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Codex Gsfa data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
