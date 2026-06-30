---
id: ADR-0002
type: ADR
status: VERIFIED
gist: Use the fastmcp client (the fastmcp-slim[client] distribution) on the agent side and the official mcp SDK's FastMCP on the server side — each role uses its idiomatic library — and declare fastmcp-slim[client] as an explicit dependency.
---

# ADR-0002: MCP client (fastmcp) vs server (official mcp SDK) libraries

## Status

Accepted.

This record lives under `docs/adr/` because it is a durable Architecture Decision Record. It documents an as-built choice that was implicit in the code until now (only ADR-0001 existed), closing the "deliberate trade-off with no ADR" gap surfaced by the repository audit. Frontmatter follows the `repo-hygiene` rule that every file under `docs/` carries enum-valid YAML.

## Context

The worked example crosses the Model Context Protocol twice, and the two sides currently use two different libraries:

- **Client side** — `platform-core`'s `agent.py` reaches the tool through pydantic-ai's `MCPToolset`, constructing the stdio transport with the **standalone `fastmcp`** package (`fastmcp.client.transports.StdioTransport`, `fastmcp.Client`). The no-key fallback round-trip uses the same `fastmcp` client.
- **Server side** — `example-mcp-server`'s `server.py` exposes the tool with the **official `mcp` SDK's bundled FastMCP** (`mcp.server.fastmcp.FastMCP`), declared via `mcp>=1.2`.

This was not a recorded decision, and it carries a latent hygiene defect: `platform-core` **imports `fastmcp` directly but does not declare it**. `fastmcp` is resolved only transitively — `pydantic-ai-slim[mcp]` pulls `fastmcp-slim[client]`, which provides the `fastmcp` import namespace. Nothing in `platform-core`'s manifest guards the import.

## Decision

Keep both libraries, each on the side where it is idiomatic, **and** make the client-side dependency explicit:

- **Client = `fastmcp` (the `fastmcp-slim[client]` distribution).** pydantic-ai's MCP toolset is built on the `fastmcp` client; using its transport is the supported, lowest-friction integration. The `fastmcp` import namespace is provided by **`fastmcp-slim[client]`** (what `pydantic-ai-slim[mcp]` resolves), **not** the full `fastmcp` package — which would clash on the same namespace or drag the server tree into `platform-core`. Fighting that to force a single library buys nothing for a template.
- **Server = official `mcp` SDK FastMCP.** The first-party SDK is the canonical way to author an MCP server, and it is what a cloner should copy when writing their own.
- **Declare `fastmcp-slim[client]` as a direct dependency of `platform-core`** ("depend on what you import" — it provides the `fastmcp` namespace the client imports). Tracked as **D-3** in `docs/registers/deferred-hardening.md`; **shipped in the `template-mcp-capability` overlay**.

## Consequences

Pros:

- **Each side uses the idiomatic library for its role** — minimal friction with pydantic-ai on the client, first-party canonical authoring on the server.
- **The example is what a cloner should copy** — a maker writing a new agent stays on pydantic-ai + fastmcp; a maker writing a new server stays on the official SDK.

Cons:

- **Two MCP libraries resolve into one venv**, enlarging the dependency graph. Acceptable: both share the same protocol and the workspace resolves one coherent set (per ADR-0001).
- **A future divergence** between `fastmcp` and the official SDK's protocol surface is a (low) risk the template would have to track.
- **D-3 (shipped in `template-mcp-capability`) declares `fastmcp-slim[client]` directly**, so a backend swap in pydantic-ai can no longer silently break the `fastmcp` import.

## Alternatives considered

- **Unify on the official `mcp` client (drop `fastmcp`).** One library both sides. Rejected for now: pydantic-ai's toolset expects `fastmcp`, so this means re-implementing the transport and re-testing the round-trip for no functional gain on a template — and it would diverge the example from the supported pydantic-ai integration a cloner is most likely to follow.
- **Unify on `fastmcp` for the server too.** Rejected: the official SDK's FastMCP is the first-party, canonical server-authoring path; the template should demonstrate that, not the standalone fork.

---

Author: Phillip Anderson | Integrate-IT Australia
