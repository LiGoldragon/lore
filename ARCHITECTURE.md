# lore — architecture

*The canonical agent-discipline repo and the workspace's curated
upstream-documentation index. The cross-workspace `AGENTS.md`
contract lives here; every workspace's per-repo `AGENTS.md` is a
thin shim that points to it.*

---

## 0 · TL;DR

`lore` holds two related things at the same root:

1. The **canonical cross-workspace agent contract**
   (`AGENTS.md`) — the rules every autonomous agent follows in any
   workspace, regardless of which repo it's editing. Per-workspace
   `AGENTS.md` files name workspace-specific carve-outs; the rest
   lives here.
2. **Curated upstream documentation** for the tools and libraries
   the workspace uses daily — one topic per file, small enough for
   agents to do targeted partial reads instead of loading giant
   references. Each file carries frontmatter with source URL and
   fetch date so staleness is visible.

Workspaces consume `lore` read-only most of the time. Edits happen
when upstream changes, when a discipline pattern crystallizes into
a repeatable rule, or when a new tool joins the workspace's daily
surface.

---

## 1 · What lives here

```text
lore/
├── AGENTS.md            canonical cross-workspace agent contract
├── CLAUDE.md            Claude-flavored shim → AGENTS.md
├── README.md            layout + reader's guide
├── LICENSE.md
│
├── annas/               Anna's Archive
├── bd/                  beads (bd) work-tracker
├── dolt/                Dolt + DoltHub
├── jj/                  Jujutsu (jj) version control
├── linkup/              linkup local-dev networking
├── nix/                 Nix, flakes, home-manager
├── rust/                Rust style + tool-specific notes
│                        (kameo, rkyv, testing, nix-packaging)
└── substack/            Substack publishing
```

Each subdirectory holds short topical files. Most are
`basic-usage.md` (the lightest read). The `nix/` and `rust/` trees
hold more — flake patterns, integration-test idioms, packaging
recipes, library-specific cookbooks (kameo actors, rkyv archive
layout, testing patterns).

---

## 2 · Boundaries

This repo owns:

- The canonical cross-workspace `AGENTS.md` contract.
- Curated upstream documentation: one topic per file, small enough
  for partial reads, each with source/date frontmatter.
- Workspace-wide patterns that have crystallized into stable rules
  (the "discipline" half of agent instructions).

It does not own:

- Workspace intent. That lives in each workspace's `ESSENCE.md`
  (e.g. `~/primary/ESSENCE.md`).
- Workspace coordination state. Locks, role lanes, and active-repo
  maps belong to each workspace (e.g. `~/primary/protocols/`).
- Per-role discipline. Role skill files (`skills/<role>.md`) live
  in their workspace, not here.
- Per-repo discipline. Each repo's own `AGENTS.md`, `skills.md`,
  and `ARCHITECTURE.md` are the per-repo surface.
- Implementation code. lore is documentation; software ships from
  other repos.

---

## 3 · Conventions

- One topic per file. Each file under ~100 lines where the source
  permits.
- Frontmatter for upstream-derived files:
  ```
  ---
  source: <canonical-url>
  fetched: <YYYY-MM-DD>
  trimmed: <what-we-stripped>
  ---
  ```
- The `AGENTS.md` / `CLAUDE.md` pattern: per-repo `AGENTS.md` files
  in other repos are thin shims pointing at this one; carve-outs
  live there, common rules live here.

---

## 4 · Constraints

- `AGENTS.md` is the canonical agent contract; per-workspace and
  per-repo `AGENTS.md` files are shims that link back to it.
- Upstream-derived files carry source-URL + fetch-date frontmatter
  so staleness is visible.
- Files are short enough for targeted partial reads (typically
  under ~100 lines per topic).
- Architecture and skill documentation in this repo describes
  stable patterns, not in-flight design.
- Workspace coordination state (locks, roles, reports) is not
  written here; that's per-workspace.

---

## 5 · Invariants

- `lore` is upstream of any per-workspace agent file. A
  workspace-level rule that conflicts with `lore`'s `AGENTS.md`
  loses unless the workspace's `AGENTS.md` explicitly carves out
  an exception.
- `lore` describes patterns and tools, not the workspace's
  current work. Decision records and audits live in each
  workspace's `reports/`.

---

## See also

- `AGENTS.md` — the canonical cross-workspace agent contract.
- `README.md` — reader's guide to the curated upstream docs.
- `~/primary/ARCHITECTURE.md` — the workspace this lore most
  often serves; describes how workspace coordination consumes
  these rules.
- `~/primary/skills/architecture-editor.md` — the rules this file
  follows.
