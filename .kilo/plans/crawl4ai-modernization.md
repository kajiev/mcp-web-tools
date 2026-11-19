# Plan: Crawl4AI modernization (fix the broken smoke test, adopt 0.9.x)

> Scope: make the documented path actually work, on the current Crawl4AI image.
> This is the fix for the "smoke tests broke for the Crawl4AI part" report.
> Companions: `remaining-modernization.md`, `fastmcp-v3-migration.md`.

## Why

- `docker-compose.yml` defaults `CRAWL4AI_IMAGE` to `unclecode/crawl4ai:latest`.
- Since Crawl4AI **0.9.0** the server is **secure-by-default**:
  - Without `CRAWL4AI_API_TOKEN` it binds loopback-only inside the container, so
    the `web-mcp` container cannot reach it over `web-tool-net`.
  - Every endpoint except `GET /health` requires
    `Authorization: Bearer $CRAWL4AI_API_TOKEN`.
- `install.sh` generates `SEARXNG_SECRET` but **not** `CRAWL4AI_API_TOKEN`;
  `.env.example` leaves it empty and only comments the pin
  (`#CRAWL4AI_IMAGE=unclecode/crawl4ai:0.7.4`).
- `make smoke` (Makefile line 63) POSTs `/md` with **no auth header**.

Result: a fresh `make up` pulls 0.9.x → extraction and smoke both fail. CI only
passes because `ci.yml` hard-pins `unclecode/crawl4ai:0.7.4`.

The `/md` route and `{url, f, q, c}` body still exist in 0.9.x, so the provider
code is mostly fine — what changed is **auth + binding**, plus a minor response
shape (`filter`/`success`/`cache` fields) that `coerce_markdown()` already
tolerates.

## Steps

1. **Pin the image.** Verify the exact 0.9.x tag exists (Docker Hub / GHCR), then:
   - `docker-compose.yml` line 43: `${CRAWL4AI_IMAGE:-unclecode/crawl4ai:0.9.5}`
     (or latest 0.9.x).
   - `.env.example`: uncomment `CRAWL4AI_IMAGE=unclecode/crawl4ai:0.9.x`.
2. **Generate the token** in `install.sh`, mirroring the `SEARXNG_SECRET` block:
   `openssl rand -hex 32`, write `CRAWL4AI_API_TOKEN=` into `.env`. The compose
   file already forwards it to both `crawl4ai` and `web-mcp`.
3. **Document the token** in `.env.example`: `CRAWL4AI_API_TOKEN` is now
   **required** (not optional) — a real bearer token, not a no-op.
4. **Confirm the 0.9.x auth model** before coding `crawl4ai.py`:
   - docs.crawl4ai.com (0.9.x): direct `Authorization: Bearer $CRAWL4AI_API_TOKEN`
     on all non-`/health` endpoints → `crawl4ai.py` already sends this when the
     token is non-empty.
   - Newer `deploy/docker/server.py` (unreleased main): `POST /token` mints a
     JWT, and `/md` requires it. If the pinned tag uses this, add a `/token`
     exchange in `crawl4ai.py` before `/md`. **Verify which applies.**
5. **Fix `make smoke`** (Makefile line 63): include
   `Authorization: Bearer $CRAWL4AI_API_TOKEN` on the `/md` request; keep the
   `{url, f:"fit"}` body.
6. **Healthcheck:** confirm `/health` is still valid (0.9.x returns
   `{status, version}`, no auth) — likely no compose change.
7. **CI** (`ci.yml`): bump `CRAWL4AI_IMAGE` to the pinned 0.9.x and set a test
   `CRAWL4AI_API_TOKEN`; keep/adjust the smoke + e2e steps to send the token.
8. **Docs:** update `docs/install.md` (smoke behavior + token), `docs/config.md`
   (`CRAWL4AI_API_TOKEN` required + image pin), README (first-run token note).
9. **Tests:** add/adjust a `crawl4ai` token-header assertion and a smoke-level
   check that `/md` with/without the token behaves as expected.

## Verification

- `make up && make smoke` green on a clean clone (pulls 0.9.x).
- `make update` green.
- `curl -H "Authorization: Bearer $TOKEN" http://localhost:11235/md` returns
  markdown; without token → 401 / connection reset.
- CI green with the new pin + token.

## Risks / open questions

- Exact 0.9.x tag + auth model (direct bearer vs `/token`+JWT) must be confirmed.
- `llm`/`bm25` modes may need an LLM provider configured server-side in 0.9.x
  (`fit`/`raw` are free and deterministic). Confirm the default path stays free.
