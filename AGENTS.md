# AGENTS.md

Guidance for AI agents working in this repository.

## What this is

MCP Web Tools — a self-hosted MCP server exposing `web_search` (SearXNG) and
`web_extractor` (Crawl4AI), with an optional SaaS fallback chain
(Tavily → Firecrawl → Exa → local). Thin FastMCP + FastAPI exposers over a
provider-adapter layer, shipped as a 4-service Docker Compose stack
(valkey + searxng + crawl4ai + web-mcp).

## Common commands

```bash
pytest -q                 # unit tests (no docker needed)
python -m compileall mcp  # syntax check
./install.sh --skip-checks  # bootstrap .env + SEARXNG_SECRET (no containers)
make up                   # start the stack
make smoke                # hit all endpoints (SearXNG, Crawl4AI, REST, MCP)
make down                 # stop (keeps valkey cache volume)
```

`make smoke` is an end-to-end check that needs the containers running. Unit
tests are hermetic (they fake the HTTP client) and run without docker.

## Layout

- `mcp/server.py` — MCP tools (FastMCP) + transport wiring.
- `mcp/api.py` — REST API (`/api/v1/search`, `/api/v1/extract`, `/healthz`).
- `mcp/tools.py` — core implementations shared by both exposers.
- `mcp/providers/` — per-backend adapters + YAML fallback-chain orchestration.
- `mcp/url_policy.py` — SSRF boundary for `web_extractor` URLs.
- `config/providers.yaml` — fallback chains (gitignored; see `.example`).
- `searxng/settings.yml`, `docker-compose.yml`, `install.sh`, `Makefile`.

## Conventions

- Tools return errors as values (a `"error"` key), never raise, across MCP and
  REST. Preserve that contract.
- Provider adapters are keyed by name; the YAML chain order (first = primary)
  drives fallback. Only hard failures advance the chain.
- `config/providers.yaml` and `.env` are secrets — never commit; only the
  `.example` files.
- Test files fake the HTTP client (`FakeClient`/`FakeResponse` in
  `tests/test_tools.py`) rather than hitting real services.

## Current work

Modernization is planned but not yet implemented. See `.kilo/plans/` — start
with `README.md` there, then `crawl4ai-modernization.md` (the fix for the broken
Crawl4AI smoke test), `remaining-modernization.md`, and the deferred
`fastmcp-v3-migration.md`.
