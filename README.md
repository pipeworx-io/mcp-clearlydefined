# @pipeworx/clearlydefined

ClearlyDefined — the licence, copyright attribution and provenance actually
found inside an open-source package, from the Linux Foundation's curated
dataset.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1863+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

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

Both URLs reach the same gateway and the same 1863+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/clearlydefined_search \
  -H 'Content-Type: application/json' \
  -d '{"pattern":"express","limit":5}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/clearlydefined_search`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-clearlydefined` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/clearlydefined/mcp` endpoint shown above. It needs network access;
there is no offline mode.

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

Without a key it calls anonymously, capped at 50 calls a day. A free account
raises that to 200 a day: get a key at https://pipeworx.io/signup and add
`"env": { "PIPEWORX_API_KEY": "<your key>" }` to the server entry.
The key is sent as a Bearer token, the same way a hosted connection sends it,
so calls are metered and billed exactly as they are there.

It lists **only** this pack's tools by default. Set `PIPEWORX_META_TOOLS=1` to
also list `ask_pipeworx` and the other gateway-wide tools.

Behind an HTTPS proxy: Node's built-in `fetch` does not read `HTTPS_PROXY` by
default. Newer Node releases honour it when `NODE_USE_ENV_PROXY=1` is set; check
your Node version's documentation.

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
