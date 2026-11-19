# Modernization plans

Plans for bringing MCP Web Tools up to date (last release v1.2.1, 2026-05-21).
None of these are implemented yet — this is the plan set for review.

Split so each is independently reviewable and mergeable:

1. **crawl4ai-modernization.md** — the actual "is it broke" fix: adopt
   Crawl4AI 0.9.x (secure-by-default), generate `CRAWL4AI_API_TOKEN`, fix
   `make smoke`. This is the only thing broken today for a fresh install.
2. **remaining-modernization.md** — pin FastMCP to 2.x, pin `searxng` image,
   Python 3.13 bump, doc-drift cleanup, and optional cruft removal.
3. **fastmcp-v3-migration.md** — superseded: the FastMCP 2.x → 3.x migration was
   folded into `remaining-modernization.md`, which pinned `fastmcp==4.2.2`
   directly.

Suggested order: 1 → 2 → 3. Each plan lists its own verification.
