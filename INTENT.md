# INTENT — lore

*What the psyche wants this project to be, and its most important
design constraints. Synthesised from workspace-backed intent that
applies to this repo plus the repo's specific purpose. Backed by the
repo's real purpose and applicable workspace constraints; not
embellished. Companion to `ARCHITECTURE.md`. Maintenance:
`primary/skills/repo-intent.md`.*

## Why this repo exists

`lore` is the **canonical cross-workspace agent-discipline repo and the
workspace's curated upstream-documentation index**. It holds two
related things at one root:

1. The canonical cross-workspace agent contract (`AGENTS.md`) — the
   rules every autonomous agent follows in any workspace, regardless of
   which repo it is editing. Per-workspace and per-repo `AGENTS.md`
   files are thin shims that point back here; carve-outs live there,
   common rules live here.
2. Curated upstream documentation for the tools and libraries the
   workspace uses daily — one topic per file, small enough for targeted
   partial reads, each carrying source-URL and fetch-date frontmatter so
   staleness is visible.

This is a documentation and discipline repo, not a stack software
component: no daemon, no contract crate, no wire vocabulary. Software
ships from other repos; `lore` is read-only most of the time.

## Load-bearing constraints

*`AGENTS.md` here is canonical; everything downstream is a shim.* A
workspace-level or per-repo rule that conflicts with `lore`'s
`AGENTS.md` loses unless the conflicting `AGENTS.md` explicitly carves
out an exception. `lore` is upstream of any per-workspace agent file.

*One topic per file, small enough for partial reads.* Each upstream-doc
file stays under roughly 100 lines where the source permits, so agents
can `Read --offset --limit` the relevant section instead of loading a
giant reference. Prefer verbatim upstream excerpts over paraphrase, so
the fidelity a summarizer would lose is preserved.

*Source/date frontmatter on every upstream-derived file.* Each carries
`source:`, `fetched:`, and `trimmed:` frontmatter so it is visible when
to refresh against upstream. Original discoveries are marked
`source: (our own — <context>)`.

*`lore` describes stable patterns, not in-flight work.* It carries
crystallized, repeatable discipline and tool documentation — not the
workspace's current work. Decision records, audits, and in-flight
design live in each workspace's `reports/`, not here.

## What does not live here

- Workspace intent (each workspace's `ESSENCE.md` / `INTENT.md`).
- Workspace coordination state — locks, role lanes, active-repo maps
  belong to each workspace's `protocols/`.
- Per-role discipline (`skills/<role>.md`) and per-repo discipline
  (each repo's own `AGENTS.md`, `skills.md`, `ARCHITECTURE.md`).
- Implementation code.

## See also

- `AGENTS.md` — the canonical cross-workspace agent contract.
- `ARCHITECTURE.md` — the repo's two-halves shape, boundaries, and
  invariants.
- `README.md` — reader's guide to the curated upstream docs.
- `primary/skills/repo-intent.md` — the discipline this file follows.
