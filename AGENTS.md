# Agent instructions — workspace-wide canonical contract

This file is the **canonical workspace contract**. Every per-repo
`AGENTS.md` is a thin shim pointing here; the rules below apply
to every repo under `~/git/`. Repo-specific carve-outs live in
that repo's own `AGENTS.md`.

---

## 🚨 Required reading, in order 🚨

1. [**INTENTION.md**](INTENTION.md) — what the project is for.
   Priorities, the deepest value, what we are not. Upstream of
   every other doc; if a downstream doc conflicts with INTENTION,
   the downstream doc is wrong.
2. [**criome/ARCHITECTURE.md**](https://github.com/LiGoldragon/criome/blob/main/ARCHITECTURE.md) — what the engine IS.
   Project-wide invariants, the request flow, the daemon shape,
   the rejected framings. Required before touching any
   sema-ecosystem repo.
3. **This file** (`lore/AGENTS.md`) — how agents work in the
   workspace.
4. **The repo-specific `AGENTS.md`** of whatever repo you're
   editing — repo-role + carve-outs only.

If you are entering this workspace as a new agent, do not skip
(1) and (2). They are upstream of every decision below.

---

## AGENTS.md / CLAUDE.md pattern

Across all canonical repos: **`AGENTS.md` holds the real content**;
`CLAUDE.md` is a one-line shim reading
**"You MUST read [AGENTS.md](AGENTS.md)."**
This way Codex (which reads `AGENTS.md`) and Claude Code (which
reads `CLAUDE.md`) converge on a single source of truth. When
creating or restructuring a repo, keep this pattern.

Per-repo `AGENTS.md` itself opens with:
**"You MUST read [lore/AGENTS.md](path) — the workspace-wide
contract."** plus the repo's role and carve-outs.

---

## Per-repo `ARCHITECTURE.md` at root

Every canonical repo carries an `ARCHITECTURE.md` at its root
(matklad convention). The file is short — typically 50-150
lines — and answers: *what does this repo do, where do things
live in it, and how does it fit into the wider sema-ecosystem.*
Standard sections: role, boundaries (owns / does not own), code
map, invariants, status, cross-cutting context (link to
[criome's `ARCHITECTURE.md`](https://github.com/LiGoldragon/criome/blob/main/ARCHITECTURE.md) and any relevant report).

Per-repo `ARCHITECTURE.md` does **not** duplicate criome's
`ARCHITECTURE.md`. It points. Project-wide invariants live once,
in criome; per-repo files describe their own niche only. When a
repo's role changes, edit that repo's `ARCHITECTURE.md` and (if
the change is system-level) criome's `ARCHITECTURE.md`.

When creating a new canonical repo: write `ARCHITECTURE.md` at
root before the first commit.

---

## Inclusion/exclusion rule — HARD

**If a repo is not listed as CANON or TRANSITIONAL in
[mentci/docs/workspace-manifest.md](https://github.com/LiGoldragon/mentci/blob/main/docs/workspace-manifest.md),
do not edit its source or docs.** Agents that drift outside the
manifest corrupt repos that are either superseded, archived, or
outside scope. To add a new canonical repo: update the manifest
and `mentci/devshell.nix`, write a report, commit.

---

## Documentation layers — strict separation

| Where | What |
|---|---|
| [criome/ARCHITECTURE.md](https://github.com/LiGoldragon/criome/blob/main/ARCHITECTURE.md) | **Project-wide canonical.** Prose + diagrams only. No code. High-level shape, invariants, relationships, rules of the engine being built. |
| [mentci/ARCHITECTURE.md](https://github.com/LiGoldragon/mentci/blob/main/ARCHITECTURE.md) | **Dev environment.** Workspace conventions, role, layout. Points at criome for the project itself. |
| `<repo>/ARCHITECTURE.md` | **Per-repo bird's-eye view.** Repo's role, boundaries, code map, status. Points at criome for cross-cutting context — does *not* duplicate it. |
| [mentci/reports/NNN-*.md](https://github.com/LiGoldragon/mentci/blob/main/reports) | **Decision records + design syntheses.** Prose + visuals. No implementation code. |
| The repos themselves | **Implementation.** Rust code, tests, flakes, `Cargo.toml`. Type sketches as compiler-checked skeletons live here. |

If a layer rule is violated, rewrite: move type sketches out of
architecture docs into a report or skeleton code; move runnable
code out of reports into the appropriate repo. Architecture stays
slim so it remains readable in one pass.

**No report links inside `criome/ARCHITECTURE.md`.** Cross-references
go *into* architecture from reports, not *out of* architecture
to reports. Reading lists, decision histories, type-spec details
all live in reports or in `mentci/docs/workspace-manifest.md` —
never inline in criome's architecture.

---

## No stop-gaps — INTENTION applied

[INTENTION.md](INTENTION.md) explicitly rejects "minimum viable,"
"ship fast," "iterate later," and "time to market" as
design-shaping framings. **The right shape now is worth more than
a wrong shape sooner; unbuilding a wrong shape costs more than
the speed it bought.**

This rules out a specific kind of agent suggestion: *"ship X as a
stop-gap; we'll replace it with Y when Y lands."* That suggestion
is the failure mode INTENTION names directly — it ships a
known-wrong shape and accepts the cost of unbuilding it later in
exchange for an immediate appearance of progress. Both the wrong
shape AND the unbuild cost survive. The "speed" it bought is
illusory.

**The recognition heuristic.** When you find yourself proposing
two paths where one is "small / hand-written / quick" and the
other is "the durable shape," that pairing IS the antipattern.
The durable shape is the answer; the small/quick path is what to
*not* propose. If the durable shape isn't ready yet, the work
that's gated by it waits — that's a feature of the system, not a
bug to route around.

When this rule conflicts with a deadline pressure, the deadline
pressure is wrong, not the rule (per INTENTION §"What we are
not"). When this rule conflicts with another rule, INTENTION
wins (per INTENTION §"The relationship with rules").

---

## Architectural defaults agents keep getting wrong

Three frames that agents repeatedly slip into when designing
schema, bootstrap, or localization. All three are rejected — see
[criome/ARCHITECTURE.md §10.1](https://github.com/LiGoldragon/criome/blob/main/ARCHITECTURE.md)
for the canonical statement. Restated here so the failure mode
is visible during design:

- **Don't propose nexus files as a storage or seed mechanism.**
  Nexus is the text request language; its parser produces signal
  envelopes for criome. Files of nexus assertions
  (`bootstrap.nexus`, `genesis.nexus`, `kinds.nexus`, …) are a
  hack — a human typing assertions into a file and feeding them
  through a tool. Don't repeat the pattern for new concerns.
  Schema bootstrap is hand-coded in criome's init; domain data
  enters sema through Assert frames at runtime, never by criome
  reading a `.nexus` file off disk.
- **Don't put schema-identity strings in sema.** Kind, field, and
  variant identifiers are slot ids in per-kind indexes — never
  strings. Display names, per-language translations, and other
  human-facing text labels for schema records live in a separate
  localization store distinct from sema and arca.
- **Don't default to "compile-time baked" for data.** When the
  architecture treats X as data (schema is data, kinds are data,
  localization is data), proposing "X lives as compile-time
  tables in the binary" regresses to a config-style pattern that
  contradicts the principle. Data lives in a database; not in a
  const array or generated source file.

If the design you are sketching pulls toward any of these,
question it before proceeding. The defaults exist because they're
agent-natural; the architecture rules out all three.

---

## Components, not monoliths

The workspace is composed of **micro-components** — one capability
per crate, per repo, per protocol. Each lives in its own repo
with its own `Cargo.toml`, `flake.nix`, and tests; each fits in a
single LLM context window; each speaks to its neighbors only
through typed protocols.

**Adding a feature defaults to a new crate, not editing an existing
one.** The burden of proof is on the contributor (human or agent)
who wants to grow a crate. They must justify why the new behavior
is part of the *same capability* — not a new one. The default
answer is "new crate."

**criome communicates; it never runs.** It does not spawn
subprocesses, write files outside sema, invoke external tools, or
link code-emission libraries. Effect-bearing work is dispatched
as typed verbs to dedicated components. Bundling executor work
into criome — or any communicator component — is the failure mode
this rule closes.

Full case (historical canon, the catastrophic record of monolith
collapse, the modern LLM-context argument):
[programming/micro-components.md](programming/micro-components.md).

---

## Beauty is the criterion

Read [programming/beauty.md](programming/beauty.md) before pushing
back on any rule below as "verbose" or "ceremonial." Beauty is
not a luxury — it is the test of correctness. **Ugly code is
evidence that the underlying problem is unsolved.**

The aesthetic discomfort *is* the diagnostic reading. When
something feels ugly, slow down and find the structure that makes
it beautiful — that structure is the one you were missing.

---

## Verb belongs to noun (thinking discipline)

Read [programming/abstractions.md](programming/abstractions.md)
before writing free functions. The discipline applies to any
language with method dispatch: behavior lives on types, not as
floating verbs. The rule's purpose is to force the question
"what type owns this verb?" — when the answer isn't obvious, the
model of the problem isn't fully formed yet, and slowing down to
find the noun is the load-bearing cognitive event.

The rule has a sharper version: the verb belongs to **the right**
noun, not just any nearby noun. Adjacency of *types* is not the
same as adjacency of *concerns* — see
[programming/abstractions.md](programming/abstractions.md)
§"The wrong-noun trap."

---

## Push, don't poll

Read [programming/push-not-pull.md](programming/push-not-pull.md).
**Polling is wrong. Always.** Producers push, consumers subscribe.
The subscription primitive is the contract; if it doesn't exist
yet, defer the real-time feature rather than poll "for now."

---

## Naming — full English words

Read [programming/naming.md](programming/naming.md). Identifiers
are read far more than they are written. Cryptic abbreviations
optimize for the writer (a few keystrokes saved) at the reader's
expense (one mental lookup per occurrence).

**Default: spell every identifier as full English words.**

The canonical rule + the full table of common offenders + the
six permitted exception classes + the "feels too verbose"
anti-pattern all live in [programming/naming.md](programming/naming.md).

---

## Binary naming — `-daemon` suffix, full English

Long-running daemon binaries carry the `-daemon` suffix:
`nexus-daemon`, `criome-daemon`, `forge-daemon`. The library half
of the same crate keeps the bare name (`nexus`, `criome`, `forge`)
— `[lib] name = "nexus"` and `[[bin]] name = "nexus-daemon"` in
the same `Cargo.toml`. CLI binaries that the user invokes
interactively keep bare names (`nexus`, `criome`).

**Why:** `nexusd` / `criomed` are unix-folklore abbreviations
carrying no information beyond the suffix. `nexus-daemon` reads
as English; `nexusd` requires expansion. It also disambiguates
the daemon binary from the CLI at PATH level. Per-process stderr
log tags also use the suffix (`nexus-daemon: ready`).

---

## One-shot binaries — `<crate>-<verb>`

Several crates ship thin one-shot binaries that wrap a single
verb of the library code as a stdin/stdout filter — useful for
test pipelines, agent harnesses, debug scripts.

Naming convention: `<crate>-<verb>`, ~30 LoC each.

| Binary | What it does |
|---|---|
| `nexus-parse` | nexus text from stdin → length-prefixed signal Frames to stdout |
| `nexus-render` | length-prefixed Frames from stdin → rendered text to stdout |
| `criome-handle-frame` | reads a Frame, dispatches via `Daemon::handle_frame` against `$SEMA_PATH`, writes reply Frame |

These expose useful affordances of the library code as standalone
tools — same shape as `gcc -E` shipping alongside `gcc`. They
live as additional `[[bin]]` entries in the existing crate, not
in a separate test-tools crate.

When to add a new one: when a verb of the library code is
genuinely useful as a filter primitive. Don't create trivial
passthrough binaries.

---

## Design-doc hygiene — state criteria positively

Avoid polluting design context with "do not use X" patterns. When
a candidate is excluded — by constraint, preference, or past
decision — **omit it silently** from forward-going context. Don't
leave "X is bad because Y," "we ruled out X," or "not X"
breadcrumbs across files.

**Why:** design docs and these instructions get loaded into every
future agent's context. Every negative statement ("don't use X")
costs context budget forever for zero ongoing value — X isn't
going to be used; naming it just keeps it alive as a dead option.

**How to apply:**
- State criteria **positively**: "must be Rust," not "Go is
  excluded because …"
- List candidates that **satisfy** the criteria. Silently drop
  the rest.
- When correcting an earlier wrong direction, the retraction is a
  one-time edit. Do not leave the retracted direction in the doc
  "for historical record" — git/jj log preserves it.
- If an excluded option keeps getting re-proposed, that's a
  signal to **add a positive criterion that silently excludes
  it**, not to add a rule naming it.

The **rejected-framings** list in
[criome/ARCHITECTURE.md §10](https://github.com/LiGoldragon/criome/blob/main/ARCHITECTURE.md)
is the *only* place wrong frames are named, and only as one-line
entries. Forensic narratives ("here's how this contamination
crept in") do not become reports — their lessons land in §10 as
one-liners.

---

## No version history in vision / design / architecture docs

Vision, design, and architecture docs describe what the system IS
and what it's heading toward. They do NOT narrate prior abandoned
approaches, scaffold history, or "why we restarted." That framing
is a self-deprecation reflex that pollutes the doc and dates
poorly.

A doc that opens with "this is the third try" is already weaker
than a doc that just describes the system. Future readers don't
need the lineage; current readers don't either. If a piece of
context truly matters (e.g. "the validator pipeline is
Datomic-inspired"), state the fact and cite the inspiration —
don't frame it as "we abandoned X to get here."

How to apply:
- When rewriting a stale doc, extract the durable ideas + recast
  in current terms. Drop *all* meta-commentary about the rewrite
  itself.
- The same rule applies to commit messages of substance —
  describe the change, not the history of failed attempts.
- Project framing inside docs is "criome" — there is no
  `criomev3` / `v1` / `v2` distinction in published docs. Version
  markers are bookkeeping for agents, never for readers.

---

## Design reports — visuals, not code

Reports in `mentci/reports/` explain, propose, analyse, or
summarise. Their medium is **prose + visuals** — ASCII diagrams,
swimlanes, flowcharts, tables, dependency graphs. Implementation
code (Rust `impl` blocks, function bodies, struct definitions
with methods, full Nix derivations) belongs in skeleton code in
the relevant repo, not in a report.

**Why:** code in a design doc goes stale the moment it lands and
the real type drifts; readers can't tell whether the report's
snippet or the repo's actual type is authoritative; visual shapes
carry the same information without that freshness trap.

**Test:** if the report has more than a couple of lines that look
like Rust / Python / Nix implementation, refactor those into a
visual.

**The narrow allowance:** a few-line *sample* of the surface the
design talks about — a snippet of a config file showing its
shape, a one-line CLI invocation, a single field declaration to
anchor a name — is fine. The rule is about implementation blocks,
not about showing the shape of the thing the design is about.

---

## Session-response style — substance goes in reports

If the agent's final-session response would be more than very
minimal (a few lines), write the substance as a report (in
`mentci/reports/`) and keep the chat reply minimal — a one-line
pointer at the report. Two reasons: (1) the Claude Code UI is a
poor reading interface; files are easier; (2) the author reviews
responses asynchronously while the agent moves to next work, so
the substance must be in a stable, scrollable, file-backed place.

Small reports are fine — the report doesn't have to be large.
Acknowledgements, tool-result summaries, "done; pushed"
confirmations don't need reports. Anything that explains,
proposes, analyses, or summarises does.

**Use relative paths in reports.** When a report references files
in sibling repos, link via `../repos/<name>/...` (the workspace
symlinks), not via GitHub URLs.

---

## Verify each parallel-tool result

When batching tool calls (parallel `Write`, `Edit`, `Bash`), scan
each result block for errors **before** any follow-up step that
depends on the results. The bundle returning is not the same as
the bundle succeeding.

Failure mode: failed `Write` calls (typically the "must Read
first" guard) don't show up in subsequent state until something
else reads the file. By then the failure has cascaded — a wrong
commit, a misled subagent, a published doc that doesn't match
what the code reflects.

How to apply:
- After any parallel batch, look at every `<result>` block. If
  any says "File has not been read yet" or "Exit code N" or
  `<tool_use_error>`, fix that *before* moving on.
- Double-check by reading the file or running `git status` / `ls`
  if a write was meant to land.
- Especially load-bearing when the next step is committing,
  pushing, or spawning an agent that consumes the written file.
- When a `Write` fails on the must-Read-first guard, do the Read
  then redo the Write before continuing.

---

## Version control: always push

After every change in any repo under `~/git/`, push immediately.
**Blanket authorization** — do not ask for confirmation.

```
jj commit -m '<msg>' && jj bookmark set main -r @- && jj git push --bookmark main
```

Push per logical commit, not batched. Unpushed work is invisible
to nix flake-input consumers and to other machines.

Use `jj`, not `git`, for all VCS interactions — never run `git
commit` / `git add` / `git reset` directly.

---

## Commit message style

Single line. Short. Lead with the repo name, then a short verb +
scope. Optionally one short clause naming the change. No nested
parens, no em-dashes, no Li-quote markers, no enumerated bullets.
Detail belongs in the diff and the report, not the commit message.

Examples:

- `signal Slot<T> migration`
- `mentci report add 119`
- `criome reader for typed slots`
- `mentci AGENTS commit-style shortened`

If a single change touches multiple repos, each repo gets its
own short commit. Don't roll them into one long compound message.

---

## Tooling — bd

`bd` (beads) tracks short items (issues, tasks, workflow). Designs
and reports go in files. See
[bd vs files — when each is the right home](bd/basic-usage.md#bd-vs-files--when-each-is-the-right-home).

`bd prime` auto-runs at session start and gives current state.

---

## Tool basics

Curated daily-use docs for the tools Li works with. Read these
before assuming how a tool works — they're trimmed to what
actually matters for our workflow, based on verified upstream
sources.

**VCS / build:**
- [jj/basic-usage.md](jj/basic-usage.md) — jj (Jujutsu) daily
  loop: `@` model, commit/describe distinction, push, undo,
  bookmarks, remotes.
- [nix/basic-usage.md](nix/basic-usage.md) — Nix flakes +
  home-manager: blueprint layout, lock-side pinning, devshells,
  `nix log`, home-manager primitives.

**Programming discipline (cross-language):**
- [programming/beauty.md](programming/beauty.md) — Beauty as the
  criterion of correctness.
- [programming/abstractions.md](programming/abstractions.md) —
  Where behavior lives. Every reusable verb belongs to a noun.
- [programming/push-not-pull.md](programming/push-not-pull.md) —
  Polling is wrong. Always. Producers push, consumers subscribe.
- [programming/micro-components.md](programming/micro-components.md)
  — One capability, one crate, one repo.
- [programming/naming.md](programming/naming.md) — Full English
  words.

**Language style:**
- [rust/style.md](rust/style.md) — Rust object style: methods on
  types, typed newtypes, single-object I/O at boundaries, typed
  `Error` enum per crate.
- [rust/nix-packaging.md](rust/nix-packaging.md) — Canonical crane
  + fenix flake layout for Rust crates.
- [rust/ractor.md](rust/ractor.md) — Ractor 0.15 in CANON daemons:
  per-file four-piece template, per-verb typed messages.
- [rust/rkyv.md](rust/rkyv.md) — Rkyv 0.8 portable feature set,
  derive-alias pattern, encode/decode API, schema-fragility limit.

**Data / issues:**
- [bd/basic-usage.md](bd/basic-usage.md) — bd (beads) daily loop.
- [dolt/basic-usage.md](dolt/basic-usage.md) — Dolt SQL daily loop.

**CLI bundle tools** (bundled via `mentci-tools.packages.<sys>.cli`):
- [annas/basic-usage.md](annas/basic-usage.md) — Anna's Archive
  CLI.
- [linkup/basic-usage.md](linkup/basic-usage.md) — Linkup
  web-search CLI.
- [substack/basic-usage.md](substack/basic-usage.md) — Publish
  Markdown to Substack.

---

## Style docs are patterns, not snapshots

Docs under `programming/` and language-style docs under
`<language>/` describe **patterns**. They must not embed specifics
from any *current* downstream repo — no exact actor counts, no
file-path listings, no concrete function or message-variant names
that will rot as the example evolves. Reference an example repo
as a one-line "see for current shape" pointer, not as
authoritative structure the doc seems to claim is true forever.

The same applies to cross-citations: a style doc should not
invoke a downstream `ARCHITECTURE.md` to backstop its claims. The
principle stands on its own; appeals-to-authority across repo
boundaries rot when the cited doc restructures.

---

## Adding new docs

When you research something non-obvious about these tools, write
it as a new topic file in the appropriate subdir. Keep files
small (~100 lines), one topic each, with the frontmatter from
README.md (`source:`, `fetched:`, `trimmed:`). Prefer verbatim
excerpts from upstream over paraphrase.
