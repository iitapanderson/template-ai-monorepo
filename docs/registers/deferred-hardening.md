---
id: REF-DEFERRED-HARDENING-001
type: REFERENCE
status: VERIFIED
gist: Durable register of consciously-deferred hardening and known gaps for template-ai-monorepo — the one place "not yet done" work lives, per repo-hygiene §3.
---

# Deferred-Hardening Register

The single durable home for **consciously-deferred** work and known gaps in
`template-ai-monorepo`. Per `repo-hygiene` §3, known gaps live in ONE register —
not in code comments (which no one re-reads) and not in an ADR (ADRs record
*decisions*, this records *outstanding work*). The structure-lint surfaces this
file via the `docs/**` frontmatter gate.

Each row states **what** is deferred, **why**, and the **trigger** that should bring
it back. Layout and frontmatter follow `repo-structure-standard` and `repo-hygiene`;
this register references those standards by name and does not restate them.

## Open

| # | Item | Why deferred | Trigger to revisit |
|---|------|--------------|--------------------|
| D-1 | **Full file-TYPE placement allowlist** in `tests/test_structure.py` (beyond the current `.py`/`.md` rules). | Phase-2 scope — the current gate polices the two highest-churn file types; a complete per-extension allowlist is broader than the spine needed. | A new file type needs placement rules, or the structure-standard adds a binding per-type rule. |
| D-2 | **Content-hash over the structure-lint rule set** so "rules changed ⇒ `CONFORMS_TO_STRUCTURE_VERSION` bump" is *enforced*, not discipline. Today the gate only asserts the in-test marker equals `[tool.structure_lint].version`. | Phase-2 scope — the cross-assert catches the common "bumped one integer, forgot the other" error; hashing the rule body is a stronger, later control. | The version-marker discipline proves insufficient (a rule changes without a bump), or the standard mandates the hash. |
| D-3 | **Declare `fastmcp` as an explicit direct dependency** of `platform-core`. `agent.py` and the integration test `import fastmcp`, but it is satisfied only transitively (via `pydantic-ai-slim[mcp]` → `fastmcp-slim`). Decision recorded in `docs/adr/0002-mcp-client-server-libraries.md`. | Held to the next code/config fix pass (this engagement was record-only). | The next dependency-touching change, or before a fork relies on the agent in production. "Depend on what you import" — a backend swap in pydantic-ai would otherwise break the import silently. |
| D-4 | **Correct `docs/folders-and-naming.md` drift.** Its "Root furniture" section says the root "contains *exactly* the following" then lists a subset — it omits `AGENTS.md`, `CLAUDE.md`, `CHANGELOG.md`, `CODE_OF_CONDUCT.md`, `CONTRIBUTING.md`, `SECURITY.md`, `.editorconfig` that ARE in the enforced 15-entry `ROOT_REQUIRED`; and it shows `conftest.py` under `tests/fixtures/` when it lives at `tests/conftest.py`. The doc's own rule is "the test wins." | Held to the next docs fix pass (record-only engagement). | Next docs pass, or whenever `ROOT_REQUIRED` changes. |

## Low / watch

| # | Item | Why deferred | Trigger to revisit |
|---|------|--------------|--------------------|
| D-5 | **`_USES_REF` in `test_structure.py` matches `uses:` anywhere on a line, including comments.** A commented `# uses: actions/x@v4` would false-positive as unpinned. Direction is fail-closed (safe), so low priority. | Cosmetic robustness; no live impact today (no such comment exists). | A legitimate comment ever trips the gate; then skip comment text before matching. |
| D-6 | **ruff version drift.** The pre-commit hook runs ruff from its own isolated env (`rev: v0.13.2`); CI and the `justfile` run `uv run ruff` (lock-pinned). The two can diverge on format/lint output. | Low — both are recent; divergence is latent, not active. | First format/lint disagreement between the local hook and CI; align the versions (e.g. run ruff via `uv run` in the hook, or match the pins). |
| D-7 | **Dependabot covers `github-actions` + `uv` only.** Docker base images (`python:3.12.10-slim`, `uv:0.11.18`) get no update PRs, and `pre-commit` is not a Dependabot-supported ecosystem — both are manual. Coupled to the pinning-scope decision in `docs/adr/0003-supply-chain-pinning-scope.md`. | Deliberate — see ADR-0003; these surfaces are intentionally tag-pinned, not in the automated-update loop. | Reassess on going public, or if a base-image / hook CVE needs a managed bump. |

## Closed

<!-- Move an item here with a completion date (dd/mm/yyyy) and the proof when it ships. -->

_None yet._

---

Author: Phillip Anderson | Integrate-IT Australia
