# ARCHITECTURE — lore

## Purpose

`lore` is the canonical cross-workspace agent-discipline repo and the
workspace's curated upstream-documentation index. It holds two related things at
one root: the canonical cross-workspace agent contract (`AGENTS.md`), and
curated upstream documentation for the tools and libraries the workspace uses
daily.

This is a documentation and discipline repo, not a stack software component:
no daemon, no contract crate, no wire vocabulary. Software ships from other
repos; `lore` is read-only most of the time.

## Two halves at one root

`AGENTS.md` is the canonical cross-workspace agent contract — the rules every
autonomous agent follows in any workspace, regardless of which repo it is
editing. Per-workspace and per-repo `AGENTS.md` files are thin shims that point
back here: common rules live here, carve-outs live in the downstream file.

The curated upstream documentation is one topic per file, each carrying
source-URL and fetch-date frontmatter so staleness is visible. The subdirectory
layout groups topics by tool family:

```text
jj/            Jujutsu (jj)
bd/            beads (bd)
dolt/          Dolt + DoltHub
nix/           Nix, flakes, home-manager
rust/          Rust style + nix-packaging
programming/   cross-language discipline (abstractions, …)
annas/         Anna's Archive CLI
linkup/        Linkup web-search CLI
substack/      Publish Markdown to Substack
```

## Boundaries

What lives here:

- the canonical cross-workspace agent contract (`AGENTS.md`);
- curated upstream tool and library documentation, one topic per file;
- original discoveries about those tools that do not live anywhere else,
  marked `source: (our own — <context>)`.

What does not live here:

- workspace intent (each workspace's `ARCHITECTURE.md` §"Workspace vision and intent");
- workspace coordination state — locks, role lanes, active-repo maps belong to
  each workspace's `protocols/`;
- per-role discipline (`skills/<role>.md`) and per-repo discipline (each repo's
  own `AGENTS.md`, `skills.md`, `ARCHITECTURE.md`);
- implementation code;
- decision records, audits, and in-flight design, which live in each
  workspace's `reports/`.

## Invariants

- `AGENTS.md` here is canonical; everything downstream is a shim. A
  workspace-level or per-repo rule that conflicts with `lore`'s `AGENTS.md`
  loses unless the conflicting `AGENTS.md` explicitly carves out an exception.
  `lore` is upstream of any per-workspace agent file.
- One topic per file, small enough for partial reads. Each upstream-doc file
  stays under roughly 100 lines where the source permits, so agents can
  `Read --offset --limit` the relevant section instead of loading a giant
  reference.
- Prefer verbatim upstream excerpts over paraphrase, so the fidelity a
  summarizer would lose is preserved.
- Source/date frontmatter on every upstream-derived file. Each carries
  `source:`, `fetched:`, and `trimmed:` frontmatter so it is visible when to
  refresh against upstream. Original discoveries are marked
  `source: (our own — <context>)`.
- `lore` describes stable patterns, not in-flight work. It carries
  crystallized, repeatable discipline and tool documentation — not the
  workspace's current work.

## See also

- `AGENTS.md` — the canonical cross-workspace agent contract.
- `README.md` — reader's guide to the curated upstream docs and the frontmatter
  convention.
