# Plan: FastMCP v3 migration (deferred)

> Status: **deferred**. Do after `crawl4ai-modernization.md` and
> `remaining-modernization.md` land and are verified. Until then the pin is
> `fastmcp>=2.3,<3`.

## Why defer

FastMCP moved to `PrefectHQ/fastmcp` and shipped v3 (v4 is arriving). This
code's FastMCP surface is small, so most v3 breaking changes do not apply — but
that must be verified in isolation rather than bundled into the Crawl4AI auth
change.

## Scope (from the FastMCP 2→3 upgrade guide)

Verify each against the actual v3 release:

1. `FastMCP("web-tool", instructions=...)` — `instructions` is not in the removed
   constructor-kwarg list, so it should survive; confirm.
2. `mcp.http_app(path="/")` — confirm the `path` kwarg survives v3. If not, use
   `mcp.run(transport="http", ...)` or the new mount semantics.
3. `@mcp.tool` decorators now return the original function. This code already
   ignores decorated-object attributes, so no change expected.
4. `from fastmcp import Client` (CI e2e) — still present in v3.
5. `api.create_app` mounts the http app with
   `lifespan=getattr(mcp_app, "lifespan", None)` — confirm lifespan wiring is
   still correct in v3.
6. Not applicable here (verify none are used): prompts (`mcp.types.PromptMessage`),
   auth providers, `WSTransport`, OpenAPI provider, `get_tools()`/`list_tools()`,
   `ctx.set_state/get_state`, background tasks.

Then bump the pin to `fastmcp>=3,<4` and re-run the full check below.

## Verification

- `pytest -q` green (incl. `test_mcp_schema.py` — `mcp.list_tools()` returns both
  tools with provider-agnostic descriptions).
- `make up && make smoke` green.
- CI e2e (`from fastmcp import Client`) green.

## Risks / open questions

- Decide whether to jump straight to v3 or wait for v4 to settle, given v4
  removes the v3 `FASTMCP_DECORATOR_MODE=object` escape hatch (unused here).
