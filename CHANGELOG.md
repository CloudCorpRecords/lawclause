# Changelog

## 0.1.0 (2026-10-28)

- First release: a forwarding client for the hosted dpolens MCP server. All four captured tools are answered locally for tools/list, and every tools/call is forwarded upstream over a streamable-HTTP JSON-RPC session (initialize, notifications/initialized, tools/call, honouring the Mcp-Session-Id header when the upstream issues one).
- Tools: `search_policies`, `get_clause`, `list_documents`, `get_document` — search the privacy-law corpus, read one clause by key, list the citable documents and their in-force versions, and read one document's outline (GDPR and UAE PDPL).
- Upstream results pass through verbatim, including the isError flag; upstream failures surface as `lawclause error:` text results.
- One bin serves the CLI (one subcommand per tool, plus `--version`) and the stdio MCP server (`mcp` subcommand). The upstream endpoint can be overridden via `DPOLENS_MCP_URL`. The upstream record declares no headers, so no auth header is ever sent.
