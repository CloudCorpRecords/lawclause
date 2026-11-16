# lawclause

Look up the privacy law itself, word for word, from your own machine's tool list. lawclause is a local forwarding client for the hosted dpolens MCP service: it answers `tools/list` locally from the captured tool definitions and forwards every lookup to dpolens, returning the answers exactly as the service sends them back.

The corpus is a published community version covering GDPR and UAE PDPL. Every clause comes back verbatim, with the key, document and version to cite it by. Nothing is cached, stored or interpreted locally — the package is a pipe to the source texts, not a legal adviser.

## The tools

- **`search_policies`** — Search the privacy law this community version holds. `query` is required. Call it before writing or changing code that collects, stores, logs, shares or deletes personal data, and before answering a question about what the law requires. Optional filters narrow the result by language, date in force, explanatory material and selected laws.
- **`get_clause`** — Read one clause by its canonical key, such as `gdpr:art-17:para-1`, with where it sits and what sits beneath it. `key` is required; `whole_subtree` returns the clause with everything beneath it in reading order.
- **`list_documents`** — List the laws this community version can cite, and which version of each is in force. No required parameters; `limit`, `offset` and `as_of` page and date-filter the listing.
- **`get_document`** — Read one document and its top-level structure, without the text. `slug` is required, such as `gdpr`. The outline shows where to find a clause key; read the words themselves with `get_clause`.

## Install and run

As an MCP server via npm:

```bash
npx -y @reneturcios/lawclause mcp
```

MCP client configuration:

```json
{
  "mcpServers": {
    "lawclause": {
      "command": "npx",
      "args": ["-y", "@reneturcios/lawclause", "mcp"]
    }
  }
}
```

As a container:

```bash
docker run -i ghcr.io/cloudcorprecords/lawclause
docker run --rm ghcr.io/cloudcorprecords/lawclause get_document '{"slug":"gdpr"}'
```

## CLI

One binary serves both the command line and the MCP server. Each tool is a subcommand taking a single JSON string with that tool's arguments:

```bash
lawclause search_policies '{"query":"right to erasure"}'
lawclause get_clause '{"key":"gdpr:art-17:para-1"}'
lawclause list_documents '{}'
lawclause get_document '{"slug":"gdpr"}'
lawclause --version
```

Exit codes: `0` when the forwarded call succeeds, `1` when the upstream result is an error, and `2` for usage problems such as an unknown subcommand or a malformed JSON argument.

## Configuration

- `DPOLENS_MCP_URL` — Overrides the upstream endpoint. The default is the pinned hosted endpoint, `https://mcp.dpolens.com/`. (The override variable is a reconstruction added by this rebuild.)

The upstream registry record declares no headers, so no authentication header is ever sent; the tool surface was captured keyless.

Upstream failures — a JSON-RPC error from the service, a non-2xx response, an unreadable body, or a request that fails outright — are reported to the caller as an error result whose text starts with `lawclause error:`.

## How forwarding works

The first forwarded call opens a session: an `initialize` request (protocol version `2025-03-26`), followed by a `notifications/initialized` notification, then the `tools/call` itself. If the service answers the initialize request with an `Mcp-Session-Id` header, that value is carried on every later request, and the open session is reused for the rest of the process. Replies are accepted either as plain JSON or as server-sent events. Successful results pass through verbatim, including any `isError` flag and any extra fields the service includes.

## Development

```bash
npm install
npm run build   # tsc -> dist/ (dist/cli.js is the bin)
npm test        # tsx --test test/*.test.ts — stubbed upstream, no network
npm run dev     # run the CLI from source
```

The test suite checks the served tool list against the captured surface, exercises the session handshake and its reuse, verifies that no `Authorization` header is ever sent, and verifies that arguments go upstream untouched and results come back untouched.

## License

MIT — see `LICENSE`. Copyright (c) 2026 Rene Turcios.
