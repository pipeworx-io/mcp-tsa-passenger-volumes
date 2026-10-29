# @pipeworx/tsa-passenger-volumes

Daily U.S. airport checkpoint screening counts — how many travelers TSA
screened nationwide each day — plus busiest/quietest-day and average/total
stats over a range.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1857+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `tsa_checkpoint_volumes(date? | start_date?/end_date? | year?)` — daily
  figures for a single day, a date range, or a whole year. Defaults to the
  current year to date.
- `tsa_checkpoint_stats(start_date?/end_date? | year?)` — max/min/average/total
  across a range or a whole year, e.g. "busiest travel day of 2026." Defaults
  to the current year to date.

Both return `found: false` (never an error, never an empty payload) when the
requested period falls outside the published range.

## Auth

Keyless.

## Data sources

- Transportation Security Administration (TSA), checkpoint travel numbers —
  the daily total of travelers screened at TSA checkpoints nationwide,
  published since 2019. Name the publisher and dataset only — the route lives
  in `src/index.ts`.

## Notes

- The publisher has no API: the figures are an HTML table, the current year
  on the landing page (descending, through roughly yesterday) and each prior
  year on its own page (ascending, full year). The pack fetches live per
  request — at most one page per calendar year a query touches — and relies
  on the gateway's edge cache (`cf.cacheTtl`) rather than a baked copy: a
  closed-out past year is cached for a week, the current (still-updating)
  year for 30 minutes.
- Published range starts at 2019; a year outside `[2019, current year]`
  resolves to `found:false`, not an error.
- An unexpected non-200 or an HTML page that no longer matches the expected
  row shape throws a loud error naming the upstream and status — never a
  silent empty result.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "tsa-passenger-volumes": {
      "url": "https://gateway.pipeworx.io/tsa-passenger-volumes/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/tsa-passenger-volumes/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1857+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/tsa_checkpoint_volumes \
  -H 'Content-Type: application/json' \
  -d '{"date":"2026-06-18"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/tsa_checkpoint_volumes`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-tsa-passenger-volumes` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/tsa-passenger-volumes/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "tsa-passenger-volumes": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-tsa-passenger-volumes"]
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
ask_pipeworx({ question: "your question about Tsa Passenger Volumes data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
