# @pipeworx/mymemory

Keyless machine + translation-memory translation between languages — the one
translation pack in the catalog that needs no API key, no self-hosted
instance, and no registration of any kind.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1857+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `mymemory_translate(text, target, source?)` — translates `text` into
  `target` (ISO 639-1, e.g. `"de"`). `source` is optional and defaults to
  automatic language detection; when it auto-detects, the detected code comes
  back in `detected_source_language`. Also returns a match-quality score
  (1.0 = an exact translation-memory match, lower = machine-translated) and
  up to 3 alternate candidate translations.

## Auth

Keyless. Every call carries a courtesy contact address (not a login, not a
key) that raises the shared anonymous allowance under the publisher's own
usage policy — see "Quota" below.

## Data sources

- MyMemory (Translated S.r.l.), a community and machine translation memory —
  translations contributed by professional translators, public corpora and
  machine translation, blended and ranked by match quality.

## Quota

Free, anonymous use is limited by the publisher to 5,000 characters/day,
raised to 50,000 characters/day by supplying a contact address (which this
pack does on every call). All Pipeworx gateway traffic shares one egress
address, so this allowance is shared across every caller of this pack —
exactly like the SEC.gov egress note elsewhere in this repo. When the shared
daily allowance is spent, the tool returns a clear `{found: false,
reason: "rate_limit"}` refusal rather than retrying or rotating identity; it
resets on the publisher's own rolling daily window, not ours.

## Why this exists

`deepl` needs a funded platform key (unfunded by choice) and its shared key
also runs on a monthly character allowance that has sat exhausted for
extended stretches. `translate` and `libretranslate` both wrap LibreTranslate,
which needs either a key or a self-hosted endpoint since the free public
mirrors shut down in 2025-26. That left no translation tool anyone without a
key of their own could actually use — this pack is that tool (fleet #3862).

Both of those packs' refusals now name `mymemory_translate` as the keyless
alternative, and the gateway's sibling-retry (`NAMED_SERVICE_SIBLING_OVERRIDE`
in `workers/gateway/src/retry-policy.ts`) falls over to it automatically when
`deepl_translate` fails on a quota-exhausted shared key — the
`source`/`source_lang` and `target`/`target_lang` field-alias families in
`shared/src/field-aliases.ts` are what let the retry hand this pack the same
arguments the failed call received.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "mymemory": {
      "url": "https://gateway.pipeworx.io/mymemory/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/mymemory/mcp` returns the tools in the table
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
curl -X POST https://gateway.pipeworx.io/v1/tools/mymemory_translate \
  -H 'Content-Type: application/json' \
  -d '{"text":"The meeting has been postponed until next week","target":"de"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/mymemory_translate`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-mymemory` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/mymemory/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "mymemory": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-mymemory"]
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
ask_pipeworx({ question: "your question about Mymemory data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
