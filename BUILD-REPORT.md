# BUILD REPORT — priceprobe

Clean-room rebuild of `com.dropwatchhq/price-check`
v1.0.0 (remote-only MCP server at `https://price.dropwatchhq.com/mcp`),
built 2026-10-28 as a forwarding client. Package:
`@reneturcios/priceprobe` 0.1.0, registry name
`io.github.CloudCorpRecords/priceprobe`.

## Surface built

Full surface: **all 2 captured tools built**, no omissions.

| Tool | Required fields as served |
|------|---------------------------|
| `list_niches` | NO `required` key in the capture — served verbatim, none invented |
| `price_check` | `niche`, `query` |

Each served tool carries exactly the keys `name`, `description`,
`inputSchema`, exactly as captured: the capture carries NO `title`
keys, so none are served. The tool literals were generated
programmatically from
`harness/fixtures/pricecheck-tools-original.json` by the script at
`/tmp/b47-priceprobe-scratch/gen_forward.py` (parse + re-serialise
into `src/forward.ts`) — never retyped — and a post-generation check
(the generator's own round-trip assertion, the registration-integrity
test suite, plus the parity harness) confirmed the served literals
deep-equal the fixture's 2 tool objects, in order. Tools are
registered via the MCP SDK low-level Server API
(`ListToolsRequestSchema` / `CallToolRequestSchema` handlers), not
McpServer+zod.

## Fixture conflicts

**None.** The spec and the fixture agree on both tools' names,
descriptions, required fields (including `list_niches` carrying no
`required` key) and property shapes. The spec's note that the capture
carries no `title` keys matches the fixture exactly.

## Labelled reconstructions (from the spec's Wire section)

- Upstream endpoint pinned: `https://price.dropwatchhq.com/mcp`.
- Env override `PRICECHECK_MCP_URL` — labelled reconstruction.
- The registry record declares NO headers on the remote: no auth
  header is ever sent. Tests assert that no `Authorization` header
  is sent under any environment — including with key-like env vars
  (`PRICECHECK_API_KEY`, `API_KEY`, `AUTHORIZATION`) set — and
  that no `X-API-Key` header is ever sent; upstream requests carry
  only `Content-Type`, `Accept` and, when issued, `Mcp-Session-Id`.
- Error rendering `priceprobe error:` text for upstream failures —
  labelled reconstruction.
- Upstream results pass through verbatim, including `isError` and
  extra fields. tools/call is forwarded as streamable-HTTP
  JSON-RPC: initialize (protocolVersion `2025-03-26`) →
  notifications/initialized → tools/call, honouring
  `Mcp-Session-Id` when issued; plain-JSON and SSE replies are
  both parsed. The forwarding core takes an injectable fetch;
  tests stub the upstream — no live network calls in tests.
- Price-verdict judgment (from the spec, recorded by the
  coordinator on the firsthand capture read): `price_check`
  returns the retail min/max/median across polled shops, the shop
  count, the price multiple, and a FAIR / OVER / SCALPED verdict
  about a collectible PRODUCT's price — a verdict about a product,
  not a person or a counterparty (the restockradar
  `will_it_restock` / oracle `check_scalper_price` shape).
  `list_niches` is the covered-niche catalogue. No transaction is
  executed; no payment, metering, contact, signup or send tool
  exists in the surface.

## Deviations

None.

## Lockfile

- Route: generated clean from birth in the build dir with no
  `node_modules` and no pre-existing lockfile:
  `npm install --package-lock-only --ignore-scripts --offline`
  (Route 3).
- Parses as JSON; `name` == `@reneturcios/priceprobe`.
- Package entries: **128**.
- sha256: `849ab056f65cdb0c52e4284d96ec0615c9bcce86b8e60e044eef047e87c7b9b0`
  (re-verified unchanged after the subsequent offline `npm ci`).

## server.json

- Description exactly: "Collectible price check: fair, over, or
  scalped versus the current retail median." — **81 characters**.
- npm package entry carries `"version": "0.1.0"`; the OCI entry
  (`ghcr.io/cloudcorprecords/priceprobe:0.1.0`) carries **no**
  `version` field. Both entries carry packageArguments
  `[{type: positional, value: mcp}]` and transport stdio.
  Dockerfile (node:20-alpine, both stages) carries the label
  `io.modelcontextprotocol.server.name="io.github.CloudCorpRecords/priceprobe"`.
  Container workflow `IMAGE` is `ghcr.io/cloudcorprecords/priceprobe`
  and its smoke test asserts the server name `priceprobe`.

## Verification (all run 2026-10-28)

1. `npx tsc --noEmit` → exit 0.
2. `npm run build` → exit 0; real `dist/` produced (cli, forward,
   index, mcp, types — .js, .d.ts, .js.map each).
3. `node dist/cli.js --version` → `0.1.0`.
4. `npm test` → **55 tests, 55 pass, 0 fail** (registration-integrity
   suite deep-equals the served tools/list against the fixture's 2
   tools, including the no-title and no-required assertions, plus
   session handling/reuse, env override, the
   no-Authorization-header assertions under any environment,
   verbatim argument forwarding and result pass-through including
   upstream isError, SSE parsing, error rendering, MCP-layer
   validation, and CLI behaviour — including a per-tool CLI call
   against a throwaway stub upstream on 127.0.0.1 reached through
   the `PRICECHECK_MCP_URL` override).
5. Parity: `python3 /home/hatch/workspace/mcp-rebuild-100/harness/parity.py --original /home/hatch/workspace/mcp-rebuild-100/harness/fixtures/pricecheck-tools-original.json --rebuild "stdio:node /home/hatch/workspace/mcp-rebuild-100/builds/priceprobe/dist/cli.js mcp" --out /home/hatch/workspace/mcp-rebuild-100/builds/priceprobe/parity-out` → **verdict PASS, parity 100.0**
   (2/2 identical, 0 missing, 0 extra, no schema diffs). Output in
   `parity-out/`.
