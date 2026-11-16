# BUILD REPORT — lawclause

Clean-room rebuild of `com.dpolens/dpolens` v0.1.0
(remote-only MCP server at `https://mcp.dpolens.com/`), built
2026-10-28 as a forwarding client. Package: `@reneturcios/lawclause`
0.1.0, registry name `io.github.CloudCorpRecords/lawclause`.

## Surface built

Full surface: **4 of 4 captured tools built**, no omissions.

| Tool | Required fields as served |
|------|---------------------------|
| `search_policies` | `query` |
| `get_clause` | `key` |
| `list_documents` | NO `required` key in the capture — served verbatim, none invented |
| `get_document` | `slug` |

Each served tool carries exactly the keys `name`, `title`,
`description`, `inputSchema`, exactly as captured (the capture's
`annotations` and `outputSchema` were dropped at slimming and are
NOT part of the served literals). The tool literals were generated
programmatically from
`harness/fixtures/dpolens-tools-original.json` by the script at
`/tmp/b44-lawclause-scratch/gen.py` (parse + re-serialise,
placeholder substitution into `src/forward.ts`) — never retyped —
and a post-generation check (the registration-integrity test suite,
plus the parity harness) confirmed the served literals deep-equal
the fixture's 4 tool objects, in order. Tools are registered via
the MCP SDK low-level Server API (`ListToolsRequestSchema` /
`CallToolRequestSchema` handlers), not McpServer+zod.

## Fixture conflicts

**None.** The spec and the fixture agree on the tools' names,
titles, descriptions, required fields (`query` for search_policies,
`key` for get_clause; no `required` key for list_documents;
`slug` for get_document) and property shapes.

## Labelled reconstructions (from the spec's Wire section)

- Upstream endpoint pinned: `https://mcp.dpolens.com/` (note the
  trailing slash).
- Env override `DPOLENS_MCP_URL` — labelled reconstruction.
- The registry record declares NO headers on the remote: no auth
  header is ever sent. Tests assert that no `Authorization` header
  is sent under any environment — including with key-like env vars
  (`DPOLENS_API_KEY`, `API_KEY`, `AUTHORIZATION`) set — and that no
  `X-API-Key` header is ever sent; upstream requests carry only
  `Content-Type`, `Accept` and, when issued, `Mcp-Session-Id`.
- Error rendering `lawclause error:` text for upstream failures —
  labelled reconstruction.
- Upstream results pass through verbatim, including `isError` and
  extra fields. tools/call is forwarded as streamable-HTTP JSON-RPC:
  initialize (protocolVersion `2025-03-26`) →
  notifications/initialized → tools/call, honouring `Mcp-Session-Id`
  when issued; plain-JSON and SSE replies are both parsed. The
  forwarding core takes an injectable fetch; tests stub the
  upstream — no live network calls in tests.

## Deviations

None.

## Lockfile

- Route: generated clean from birth in the build dir with no
  `node_modules` and no pre-existing lockfile:
  `npm install --package-lock-only --ignore-scripts --offline`
  (Route 3).
- Parses as JSON; `name` == `@reneturcios/lawclause`.
- Package entries: **128**.
- sha256: `fc3027a7a9b8151329f74b8f0ef81051ad7bd43896a0006edaef3213f16cabe4`
  (re-verified unchanged after the subsequent offline `npm install`).

## server.json

- Description exactly: "Search and cite privacy law clauses: GDPR
  and UAE PDPL texts, documents and versions." — **85 characters**.
- npm package entry carries `"version": "0.1.0"`; the OCI entry
  (`ghcr.io/cloudcorprecords/lawclause:0.1.0`) carries **no**
  `version` field. Both entries carry packageArguments
  `[{type: positional, value: mcp}]` and transport stdio. Dockerfile
  (node:20-alpine, both stages) carries the label
  `io.modelcontextprotocol.server.name="io.github.CloudCorpRecords/lawclause"`.
  Container workflow `IMAGE` is `ghcr.io/cloudcorprecords/lawclause`
  and its smoke test asserts the server name `lawclause`.

## Verification (all run 2026-10-28)

1. `npx tsc --noEmit` → exit 0.
2. `npm run build` → exit 0; real `dist/` produced (cli, forward,
   index, mcp, types — .js, .d.ts, .js.map each).
3. `node dist/cli.js --version` → `0.1.0`.
4. `npm test` → **54 tests, 54 pass, 0 fail** (registration-integrity
   suite deep-equals the served tools/list against the fixture's 4
   tools, plus session handling/reuse, env override, the
   no-Authorization-header assertions under any environment,
   verbatim argument forwarding and result pass-through including
   upstream isError, SSE parsing, error rendering, MCP-layer
   validation, and CLI behaviour).
5. Parity: `python3 /home/hatch/workspace/mcp-rebuild-100/harness/parity.py --original /home/hatch/workspace/mcp-rebuild-100/harness/fixtures/dpolens-tools-original.json --rebuild "stdio:node /home/hatch/workspace/mcp-rebuild-100/builds/lawclause/dist/cli.js mcp" --out /home/hatch/workspace/mcp-rebuild-100/builds/lawclause/parity-out` → **verdict PASS, parity 100.0**
   (4/4 identical, 0 missing, 0 extra, no schema diffs). Output in
   `parity-out/`.
