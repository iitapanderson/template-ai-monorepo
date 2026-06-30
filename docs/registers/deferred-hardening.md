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
| D-4 | **Correct `docs/folders-and-naming.md` drift.** Its "Root furniture" section says the root "contains *exactly* the following" then lists a subset — it omits `AGENTS.md`, `CLAUDE.md`, `CHANGELOG.md`, `CODE_OF_CONDUCT.md`, `CONTRIBUTING.md`, `SECURITY.md`, `.editorconfig` that ARE in the enforced 15-entry `ROOT_REQUIRED`; and it shows `conftest.py` under `tests/fixtures/` when it lives at `tests/conftest.py`. The doc's own rule is "the test wins." | Held to the next docs fix pass (record-only engagement). | Next docs pass, or whenever `ROOT_REQUIRED` changes. |
| D-8 | **Composite per-overlay structure-versioning** (spike WS-5). The family uses a single base-owned integer (`CONFORMS_TO_STRUCTURE_VERSION`); once overlays update independently (`copier update` per `.copier-answers.<name>.yml`) one integer cannot express "python-stack bumped, base did not." | Single integer is sufficient with one composition reference today; composite versioning is the standard-ahead-of-exemplar that reference-impl-first warns against. | The first real independent two-overlay `copier update` — codify the composite scheme from that exemplar, not before. |
| D-9 | **`.copier-answers.base.yml` in `root_required` for LINKED repos** (spike WS-12). The structure-lint has no root catch-all allowlist, so a linked repo that loses its answers file (its `copier update` linkage) is not flagged. Adding the answers file to `root_required` only for linked consumers would surface lost linkage. | No repo is linked yet (the base overlay was only just extracted); adding it now would false-flag every non-linked repo. | When this repo (or any consumer) is linked via `.copier-answers.base.yml` (R-B link-back step) and loss-of-linkage detection is wanted. |
| D-11 | **Local gate runs no GitHub-Actions linter** (`actionlint`/`zizmor`), so a workflow defect that only GitHub's expression lexer or workflow schema rejects passes every local check — PyYAML parses it and the render-gate never evaluates `${{ }}`. *Fail-open:* a broken workflow ships green locally and only `startup_failure`s on push. Surfaced when both templates' meta-CI died at startup on their first GitHub push: an empty `${{ }}` inside a `run:`-block **shell comment** is a fatal parse error GitHub evaluates even in comments. The two instances were fixed at source (reworded + actionlint-verified); the *gate gap* that let them through is what's deferred. | Adding `actionlint` is a new toolchain surface (Docker image / binary) plus a hook + structure-lint test across base **and** overlay; this engagement was scoped to the BASE→OVERLAY push chain. | Going public / first external contributor, or the next workflow-touching change — wire `actionlint` into `.pre-commit-config.yaml` and/or a structure-lint test over `.github/workflows/**` so this class fails locally. |

## Low / watch

| # | Item | Why deferred | Trigger to revisit |
|---|------|--------------|--------------------|
| D-5 | **`_USES_REF` in `test_structure.py` matches `uses:` anywhere on a line, including comments.** A commented `# uses: actions/x@v4` would false-positive as unpinned. Direction is fail-closed (safe), so low priority. | Cosmetic robustness; no live impact today (no such comment exists). | A legitimate comment ever trips the gate; then skip comment text before matching. |
| D-6 | **ruff version drift.** The pre-commit hook runs ruff from its own isolated env (`rev: v0.13.2`); CI and the `justfile` run `uv run ruff` (lock-pinned). The two can diverge on format/lint output. | Low — both are recent; divergence is latent, not active. | First format/lint disagreement between the local hook and CI; align the versions (e.g. run ruff via `uv run` in the hook, or match the pins). |
| D-7 | **Dependabot covers `github-actions` + `uv` only.** Docker base images (`python:3.12.10-slim`, `uv:0.11.18`) get no update PRs, and `pre-commit` is not a Dependabot-supported ecosystem — both are manual. Coupled to the pinning-scope decision in `docs/adr/0003-supply-chain-pinning-scope.md`. | Deliberate — see ADR-0003; these surfaces are intentionally tag-pinned, not in the automated-update loop. | Reassess on going public, or if a base-image / hook CVE needs a managed bump. |
| D-10 | **SHA-pin gate accepts a 40-hex git *tag* as if it were a commit.** `_SHA_PIN = @[0-9a-f]{40}$` (`tests/test_structure.py`, and the base `tools/governance_checks.py`) is a *textual* check — it cannot tell a 40-hex commit from a crafted 40-hex tag (e.g. `@aaaa…aaaa`); only API resolution (pinact `--verify` / zizmor) can. Surfaced by the BUILD-STEP-2 closing red-team. Coupled to `docs/adr/0003-supply-chain-pinning-scope.md` (which documents surface-scoping but not this textual-vs-resolved limit). | Low real-world exploitability — the attacker must control the referenced Action repo to create the tag — and the gate stays fail-closed for the common floating-tag case. | Going public, a resolving tool (pinact/zizmor) is adopted, or the supply-chain ADR is revised; add a residual-risk line to ADR-0003 then. |

## Closed

<!-- Move an item here with a completion date (dd/mm/yyyy) and the proof when it ships. -->

| # | Item | Closed | Proof |
|---|------|--------|-------|
| D-3 | **Declare the `fastmcp` client as an explicit direct dependency of `platform-core`** — corrected during closure from the package name `fastmcp` to **`fastmcp-slim[client]`** (the distribution that actually provides the `fastmcp` import namespace; the full `fastmcp` package would clash on it). | 30/06/2026 | Shipped in the `template-mcp-capability` overlay: `template/packages/platform-core/pyproject.toml` declares `fastmcp-slim[client]>=3.4`, asserted by the overlay render-gate's closure check. `docs/adr/0002-mcp-client-server-libraries.md` updated to the corrected package name + "shipped". (The pre-partition exemplar's own `packages/platform-core` stays transitive-only — it is the extraction source, superseded by the rendered overlay.) |

---

Author: Phillip Anderson | Integrate-IT Australia
