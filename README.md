# @pipeworx/icypeas

Find a work email by name + company domain/name, via Icypeas (icypeas.com).
Part of the `enrich-waterfall` finder rotation (fleet #2860).

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1856+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `icypeas_find_email(first_name?, last_name?, domain_or_company, _apiKey)` —
  work email with Icypeas's own certainty label.

## Auth

BYO only. Pass your Icypeas API key as `_apiKey` (header
`Authorization: Bearer <key>`). Enable API access and grab the key from the
API section of your icypeas.com dashboard. No platform key (D4).

## Data sources

- `https://app.icypeas.com/api/email-search` — email search.

## Verification notes (fleet #2860, 2026-10-08, no vendor key held) — READ BEFORE TRUSTING THE PARSE

- **Icypeas's documented model is ASYNC**: the vendor's own docs
  (https://api-doc.icypeas.com/find-emails/email-discovery/) describe
  `/email-search` as enqueuing a search, with an optional webhook fired "when
  it is done" — implying the bare enqueue response does not itself carry the
  email. We hold no Icypeas key, so we could not make a real search end to
  end to see the actual success shape, and an unauthenticated probe cannot
  get past the 401.
- **What IS confirmed by edge probe:** the endpoint is live, and both no
  `Authorization` header and `Authorization: Bearer invalidkey123` return the
  identical HTTP 401
  `{"message":"The user does not exist","error":"UserNotFoundError","status":401,
  "code":"user_not_found_error"}` — so the header name is right and a bad key
  surfaces honestly (as "no such user") rather than silently.
- **The parser is defensive, not confirmed**: it reads a `results` array at
  either the top level or under `item.results` and takes the first entry with
  a non-empty `email`. This is a best-effort shape inferred from Icypeas's
  Make/n8n no-code integrations (which present single-search results inline),
  NOT a field-by-field confirmed response. **If this pack returns
  `found: false` on every call even with a valid key, the first thing to
  check is whether `/email-search` actually hands back an `item._id` that
  must be polled via a separate results endpoint** — that would mean this
  pack needs a second poll step added, not that the person has no email.
  Flagging this explicitly rather than letting a parse gap read as a mundane
  "not found."
- **Not verified (no key held):** the exact success response shape; whether
  a poll step is required; the `certainty` field's possible values.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "icypeas": {
      "url": "https://gateway.pipeworx.io/icypeas/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/icypeas/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1856+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

This pack takes your own API key (`_apiKey`) — we don't front one for it, so there's no curl here that would run without it. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/icypeas_find_email`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-icypeas` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/icypeas/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "icypeas": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-icypeas"]
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
ask_pipeworx({ question: "your question about Icypeas data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
