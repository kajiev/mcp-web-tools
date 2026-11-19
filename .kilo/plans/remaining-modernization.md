# Plan: Remaining modernization (non-Crawl4AI)

> Scope: pin FastMCP to 2.x now, pin the `searxng` image, Python 3.13, doc-drift
> cleanup, optional cruft removal. FastMCP 3.x is a separate, deferred plan
> (`fastmcp-v3-migration.md`).

## Steps

### Pin FastMCP to 2.x (now)

1. `mcp/requirements.txt`: `fastmcp>=2.3` → `fastmcp>=2.3,<3`.
   - Reason: fresh builds currently resolve FastMCP **3.x** (repo moved
     `jlowin/fastmcp` → `PrefectHQ/fastmcp`) which has breaking changes. Pinning
     2.x keeps the documented path stable until the v3 migration is done
     deliberately.
2. (Lower priority) add upper bounds to other fast-moving deps (`httpx`,
   `fastapi`, `pydantic`) if reproducible builds matter.

### Pin remaining images

3. `docker-compose.yml`: pin `searxng/searxng:latest` → a known-good tag (same
   rationale as Crawl4AI; `latest` drifts).

### Python bump

4. `mcp/Dockerfile`: `python:3.12-slim` → `python:3.13-slim`.
5. CI: `ci.yml` `python-version: 3.12` → `3.13`; `pages.yml` `3.11` → `3.13`.
6. Verify FastMCP 2.x + FastAPI + pydantic on 3.13 (they support it).

### Doc-drift cleanup

7. `.env.example` line 13: `Provider fallback chains (Brave → Tavily → local)`
   → `(tavily → firecrawl → exa → local)`.
8. `docs/config.md` line 15: `prompts for Brave and Tavily keys` → `prompts for
   Tavily, Firecrawl, Exa keys`.
9. README: remove any remaining stale "Brave" provider references.

### Cleanup (decide before doing)

10. Untrack + remove `.jules/` (leftover agent state; its SSRF learnings are
    already reflected in `mcp/url_policy.py`). Add `.jules/` to `.gitignore`.
11. Container-name prefix `web-tool-*` → `mcp-web-tools-*` (breaking for existing
    deploys; CHANGELOG flags it as "future major release"). Recommend **defer**
    to a major bump unless you want it now.

### SSRF follow-up (deferred hardening, not a blocker)

12. `mcp/url_policy.py` is already solid (`getaddrinfo`, trailing-dot,
    unspecified/reserved/multicast). Known ceilings for later: fail-open on DNS
    failure, and DNS-rebinding TOCTOU (validated IP ≠ the IP the downstream
    fetcher actually dials).

## Verification

- `pytest -q` green.
- `make up && make smoke` green with pinned images + FastMCP 2.x.
- Fresh `pip install -r requirements.txt` resolves FastMCP **2.x** (not 3.x).

## Risks / open questions

- Whether to include the breaking container-name rename now (recommend defer).
