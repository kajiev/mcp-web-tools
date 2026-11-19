# Plan: FastMCP major-version migration (superseded)

> Status: **superseded**. This plan described a deliberate FastMCP 2.x → 3.x
> migration. It is no longer needed: `remaining-modernization.md` pinned FastMCP
> to `fastmcp==4.2.2` directly (verified green on 4.2.2) instead of the 2.x pin,
> because the code already depended on ≥3.x behavior and a fresh
> `pip install` had been resolving 3.x/4.x for some time. See
> `remaining-modernization.md` for that decision.

Key facts that made the 2.x → 3.x migration moot:

- `@mcp.tool` per-parameter descriptions (from the `Args:` docstrings) only
  materialize in the MCP JSON schema on FastMCP **≥3.x**. On 2.x they are
  absent, so the 2.x pin would have regressed the agent-facing tool schema.
- The code surface is small and already works on 4.x: `FastMCP(instructions=)`,
  `@mcp.tool`, `mcp.http_app(path="/")`, `api.create_app` lifespan wiring, and
  `from fastmcp import Client` (CI e2e) were all verified against 4.2.2.

## When FastMCP 5.x ships

Re-apply the verification below against 5.x, then decide whether to bump the
`fastmcp==4.2.2` pin:

- `pytest -q` green (incl. `test_mcp_schema.py` and the `test_api.py` parity
  test — both are written version-agnostic so they survive a major bump).
- `make up && make smoke` green.
- CI e2e (`from fastmcp import Client`) green.
