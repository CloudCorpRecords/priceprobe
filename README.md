# priceprobe

Is that listing for a Charizard card or a LEGO set actually a bargain, or is it marked up past what shops charge? priceprobe is a local forwarding client for the hosted collectible price-check MCP service: it answers `tools/list` locally from the captured tool definitions and forwards every call to the service, returning the answers exactly as the service sends them back.

The headline read, `price_check`, compares a collectible product's price with the current retail picture across the shops the service polls — the retail minimum, maximum and median, how many shops carry the product, the price multiple, and a FAIR / OVER / SCALPED verdict. That verdict is about a product's price, never about a person, and nothing is bought or sold through these tools. The companion tool, `list_niches`, lists the collectible niches the checker covers so callers can pick a valid niche before checking a price.

Both captured tools are built; there are no omissions.

## The tools

- **`list_niches`** — The catalogue of collectible niches the checker covers (TCG, LEGO, Funko, mechanical keyboards, vinyl, aquarium). Takes no parameters and has no required fields; call it first to pick a valid `niche` value for `price_check`.
- **`price_check`** — The verdict read for one product: retail min/max/median across the polled shops, the number of shops carrying it, the price multiple, and a FAIR / OVER / SCALPED verdict. `niche` and `query` are required; pass `price` as well to have the verdict compare against a specific asking price you are seeing.

## Install and run

As an MCP server via npm:

```bash
npx -y @reneturcios/priceprobe mcp
```

MCP client configuration:

```json
{
  "mcpServers": {
    "priceprobe": {
      "command": "npx",
      "args": ["-y", "@reneturcios/priceprobe", "mcp"]
    }
  }
}
```

As a container:

```bash
docker run -i ghcr.io/cloudcorprecords/priceprobe
docker run --rm ghcr.io/cloudcorprecords/priceprobe list_niches '{}'
```

## CLI

One binary serves both the command line and the MCP server. Each tool is a subcommand taking a single JSON string with that tool's arguments:

```bash
priceprobe list_niches '{}'
priceprobe price_check '{"niche":"tcg","query":"Charizard V"}'
priceprobe price_check '{"niche":"lego","query":"LEGO 10307","price":249.99}'
priceprobe --version
```

Exit codes: `0` when the forwarded call succeeds, `1` when the upstream result is an error, and `2` for usage problems such as an unknown subcommand or a malformed JSON argument.

## Configuration

- `PRICECHECK_MCP_URL` — Overrides the upstream endpoint. The default is the pinned hosted endpoint, `https://price.dropwatchhq.com/mcp`. (The override variable is a reconstruction added by this rebuild.)

The upstream registry record declares no headers, so no authentication header is ever sent; the tool surface was captured keyless.

Upstream failures — a JSON-RPC error from the service, a non-2xx response, an unreadable body, or a request that fails outright — are reported to the caller as an error result whose text starts with `priceprobe error:`.

## How forwarding works

The first forwarded call opens a session: an `initialize` request (protocol version `2025-03-26`), followed by a `notifications/initialized` notification, then the `tools/call` itself. If the service answers the initialize request with an `Mcp-Session-Id` header, that value is carried on every later request, and the open session is reused for the rest of the process. Replies are accepted either as plain JSON or as server-sent events. Successful results pass through verbatim, including any `isError` flag and any extra fields the service includes.

## Development

```bash
npm install
npm run build   # tsc -> dist/ (dist/cli.js is the bin)
npm test        # tsx --test test/*.test.ts — stubbed upstream, no network
npm run dev     # run the CLI from source
```

The test suite checks the served tool list against the captured surface (both tools, deep-equal, in order, with no titles invented and no `required` key added to the tool that carries none), exercises the session handshake and its reuse, verifies that no `Authorization` header is ever sent under any environment, and verifies that arguments go upstream untouched and results come back untouched.

## License

MIT — see `LICENSE`. Copyright (c) 2026 Rene Turcios.
