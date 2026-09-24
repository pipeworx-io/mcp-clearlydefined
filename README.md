# @pipeworx/clearlydefined

ClearlyDefined — the licence, copyright attribution and provenance actually
found inside an open-source package, from the Linux Foundation's curated
dataset.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1679+ live data sources.

## Tools

- `clearlydefined_search(pattern, limit?, type?)` — find component coordinates
  by name fragment across npm, Maven, PyPI, RubyGems, NuGet, crates.io,
  Packagist, Go and GitHub.
- `clearlydefined_definition(type, provider, namespace?, name, revision, include_files?, file_limit?)` —
  the curated record for one exact version: declared SPDX licence, every licence
  expression discovered in the files, the copyright holders to attribute, the
  upstream source repo + commit, release date, hashes, and 0-100 scores.
- `clearlydefined_harvest_status(type, provider, namespace?, name, revision)` —
  which scanners (ScanCode, Licensee, REUSE, FOSSology) analysed that revision,
  and at which versions — i.e. how much evidence sits behind the definition.

## Auth

Keyless.

## Data sources

- <https://api.clearlydefined.io/definitions?pattern=> — coordinate search.
- <https://api.clearlydefined.io/definitions/{type}/{provider}/{namespace}/{name}/{revision}> — one definition.
- <https://api.clearlydefined.io/harvest/{coordinates}?form=list> — harvest records.

## Things the next person would otherwise rediscover

- **Coordinates are five segments and the namespace is never omitted** — an
  absent namespace is the literal `-`, as in `npm/npmjs/-/express/4.18.2`.
- **`?pattern=` is a prefix scan and does not always answer.** Measured
  2026-09-17: `lodash`, `express` and `requests` returned in seconds;
  `log4j-core` never returned inside 55 seconds. The pack bounds it at 30s and
  the error tells the caller to use exact coordinates instead. Other query
  shapes on `/definitions` (`?type=&provider=&name=&sort=`) answered 502.
- **A definition can exist with no licence evidence at all**, which is a
  different answer from MIT. The payload carries an explicit
  `license_evidence: "present" | "none_found"` so a null `declared_license` is
  never read as "unlicensed" by accident.
- `/definitions/.../revisions` is NOT a revision list — the API treats
  `revisions` as a literal version string and returns an empty definition for
  it.

## Related packs

`osv` answers the vulnerability question and `deps-dev` the dependency question
about the same components. This one answers the licence-compliance question.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "clearlydefined": {
      "url": "https://gateway.pipeworx.io/clearlydefined/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/clearlydefined/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1679+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/clearlydefined_search \
  -H 'Content-Type: application/json' \
  -d '{"pattern":"express","limit":5}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/clearlydefined_search`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "clearlydefined": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-clearlydefined"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-clearlydefined
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Clearlydefined data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
