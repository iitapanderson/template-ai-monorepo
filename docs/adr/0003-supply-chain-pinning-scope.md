---
id: ADR-0003
type: ADR
status: VERIFIED
gist: Scope the SHA/digest-pin supply-chain mandate to CI Actions (the trusted build/release path); keep pre-commit hook revs and Dockerfile base images on readable tags, since this repo is a teaching template, not a deployed artifact.
---

# ADR-0003: Supply-chain pinning is scoped to CI Actions, not every ref

## Status

Accepted.

Durable ADR recording a deliberate asymmetry that, before this record, read as a possible oversight — exactly the kind of "deviation not recorded as an ADR" the repository audit looks for. Frontmatter follows the `repo-hygiene` `docs/**` rule.

## Context

`AGENTS.md` states a SHA-pinning mandate ("Supply-chain integrity over convenience"), and `tests/test_structure.py::test_all_workflow_actions_are_sha_pinned` enforces it: every `uses:` in `.github/workflows/` must pin a 40-hex commit SHA (or, for `docker://` actions, an `@sha256:` digest). Two other supply-chain surfaces are **not** held to that bar:

- **`.pre-commit-config.yaml`** pins hook repos to mutable version tags (`rev: v5.0.0`, `v0.13.2`, `v8.21.2`).
- **The service `Dockerfile`** pins base images to tags (`python:3.12.10-slim`, `ghcr.io/astral-sh/uv:0.11.18`), not digests.

The framing question is decisive: **this repository is a reference template whose product is the standard it propagates to clones** — it is not a deployed application. Nothing here ships to production; the example image is never run in anger. So the question is not "does this repo need the protection" but "what posture should every clone inherit."

## Decision

**Scope the SHA/digest-pin mandate to CI Actions only.** Pre-commit hook revs and Dockerfile base images stay on readable, specific tags.

Rationale, surface by surface:

- **CI Actions — pin, and a clone inherits it.** Actions run in the *trusted build/release path*: they hold write tokens and produce the signed release artifacts. This is the highest-value supply-chain surface and the one where a moved tag does real damage. The mandate earns its keep here and a clone should carry it forward.
- **pre-commit hooks — tag-pinned.** Hooks run in the *local dev path* (a much weaker threat model — no release tokens, no published artifact). The ecosystem norm is tag pins managed by `pre-commit autoupdate`, and Dependabot cannot update them at all. SHA-pinning them fights the tool's grain for little gain.
- **Docker bases — tag-pinned, with a cloner note.** For a *teaching* template, a readable `python:3.12.10-slim` instructs a cloner better than an opaque `@sha256:` blob, and the example image is never deployed. A cloner productionising a real service **should digest-pin its bases** — that guidance belongs in a comment, not in an enforced gate on an undeployed example.

## Consequences

Pros:

- **Coherent, defensible posture** — the strongest control sits on the highest-risk surface (the build/release path), and the asymmetry is now a documented decision, not an apparent gap.
- **The template stays legible** — clones inherit SHA-pinned Actions (correct) without inheriting digest noise on local-dev and example surfaces where it teaches poorly.

Cons:

- **pre-commit and Docker refs remain mutable** — a moved tag could alter a local hook run or a base layer. Mitigated by specific patch tags and by confining the unpinned surfaces to the local-dev / undeployed-example paths. Tracked as **D-7** in `docs/registers/deferred-hardening.md` (no automated update path for these ecosystems).

## Alternatives considered

- **Extend SHA/digest pinning to all surfaces.** Rejected: it over-applies a build-path control to a local-dev tool against its ecosystem grain, breaks `pre-commit autoupdate`, and trades a teaching template's legibility for theoretical hardening on an example image that is never deployed.
- **Leave the asymmetry undocumented.** Rejected: in a reference repo that exists to model a standard, an unexplained inconsistency reads as an oversight and risks cloners cargo-culting either the pinning or the gaps without understanding why.

---

Author: Phillip Anderson | Integrate-IT Australia
