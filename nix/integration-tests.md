---
source: (our own — distilled from workspace/checks/ + workspace/lib/scenario.nix)
fetched: 2026-04-28
---

# Nix integration tests — chained derivations

How to write a multi-process integration test as a chain of
nix derivations rather than one monolithic bash script.

## The shape

Each step in the test is its own derivation. A step's output
(captured to `$out/...` files) becomes a dependency of the
next step via Nix string-interpolation of the store path:

```nix
let
  step-A = pkgs.runCommand "step-A" {} ''
    ...do work...
    mkdir -p $out
    cp output-bytes $out/result.bin
  '';

  step-B = pkgs.runCommand "step-B" {} ''
    cat ${step-A}/result.bin | do-next-thing > $out
  '';
in step-B
```

Nix tracks the dependency automatically through the
`${step-A}` interpolation. Step B only rebuilds when step A's
output hash changes; step A only rebuilds when its inputs
change. Failures localise to the step that broke.

## Two patterns, two purposes

**Daemon-mode chain — state persistence across restarts.**
Each step starts the full daemon stack in its own sandbox,
runs one user-visible interaction, and captures the resulting
state file. The next step copies that state in and runs again
against a fresh daemon process. Tests that *application
state* survives clean shutdown and re-open. Use when:

- Verifying a database file format is stable across daemon
  restarts (this is what workspace's `scenario-chain` exercises)
- Schema migration testing: state-V1 → daemon v2 reads it →
  state-V2
- Bootstrap-loop testing: criome v1 stores records → criome v2
  (a different binary build) reads them

**Binary-stability chain — per-transformation isolation.**
Each step runs ONE one-shot binary, capturing its raw byte
output. The next step consumes those bytes as input. Tests
that *binary serialisation* is stable across process and
version boundaries. Use when:

- Verifying rkyv (or other binary) wire format is stable
  across process boundaries — bytes written to disk by one
  process must decode in another (workspace's `roundtrip-chain`
  exercises this with length-prefixed signal Frames)
- Per-daemon-transformation isolation: parser bug, handler
  bug, renderer bug each fail the specific phase that
  produces them
- Cross-version binary compatibility: producer-version A,
  consumer-version B

The two patterns are complementary. Daemon-mode tests the
*application contract*; binary-stability tests the *wire
contract*.

## Skeleton — daemon-mode step builder

A step builder factored as a Nix function. Spawns the daemon
graph in a sandbox, pipes user-text through a CLI, captures
both the rendered reply text and the resulting state file
into the step's `$out`.

```nix
# lib/scenario.nix
{ pkgs, criome, nexus, nexus-cli }:

{ name, input, priorState ? null }:

pkgs.runCommand name { } ''
  set -euo pipefail
  cd $TMPDIR

  ${pkgs.lib.optionalString (priorState != null)
    "install -m 644 ${priorState}/state.redb sema.redb"}

  cleanup() {
    kill ''${nexus_pid:-} ''${criome_pid:-} 2>/dev/null || true
    wait ''${nexus_pid:-} ''${criome_pid:-} 2>/dev/null || true
  }
  trap cleanup EXIT

  CRIOME_SOCKET=$PWD/criome.sock SEMA_PATH=$PWD/sema.redb \
    ${criome}/bin/criome-daemon &
  criome_pid=$!
  for i in $(seq 1 50); do [ -S "$PWD/criome.sock" ] && break; sleep 0.1; done

  NEXUS_SOCKET=$PWD/nexus.sock CRIOME_SOCKET=$PWD/criome.sock \
    ${nexus}/bin/nexus-daemon &
  nexus_pid=$!
  for i in $(seq 1 50); do [ -S "$PWD/nexus.sock" ] && break; sleep 0.1; done

  echo ${pkgs.lib.escapeShellArg input} | \
    NEXUS_SOCKET=$PWD/nexus.sock ${nexus-cli}/bin/nexus > response.txt

  # Stop daemons cleanly so state flushes.
  kill $nexus_pid; wait $nexus_pid 2>/dev/null || true
  kill $criome_pid; wait $criome_pid 2>/dev/null || true

  mkdir -p $out
  cp response.txt $out/response.txt
  cp sema.redb    $out/state.redb
''
```

A leaf check then chains steps + asserts:

```nix
let
  step = flake.lib.scenario { inherit pkgs; criome = ...; nexus = ...; nexus-cli = ...; };
  assertNode = step { name = "assert-node"; input = ''(Node "User")''; };
  queryNodes = step {
    name       = "query-nodes";
    input      = ''(| Node @name |)'';
    priorState = assertNode;        # ← state.redb chains forward
  };
in
pkgs.runCommand "scenario-chain" { } ''
  grep -qE '^\(Ok\)$' ${assertNode}/response.txt || exit 1
  grep -qE '^\[\(Node "User"\)\]$' ${queryNodes}/response.txt || exit 1
  touch $out
''
```

## Skeleton — binary-stability per-phase step

For per-daemon-transformation testing, each step runs ONE
one-shot binary. The daemons need to expose a one-shot mode
— `<crate>-<verb>` binaries that read stdin and write stdout
(e.g. `nexus-parse`, `nexus-render`, `criome-handle-frame`).

```nix
# checks/roundtrip-parse.nix
{ pkgs, inputs, system, ... }:
let
  nexus = inputs.nexus.packages.${system}.default;
  input = pkgs.writeText "input" ''(Node "User")'';
in
pkgs.runCommand "roundtrip-parse" { } ''
  ${nexus}/bin/nexus-parse < ${input} > frame.bin
  mkdir -p $out
  cp frame.bin $out/frame.bin
''

# checks/roundtrip-handle.nix
{ pkgs, inputs, system, flake, ... }:
let
  criome = inputs.criome.packages.${system}.default;
  parsed = flake.checks.${system}.roundtrip-parse;
in
pkgs.runCommand "roundtrip-handle" { } ''
  cd $TMPDIR
  SEMA_PATH=$PWD/sema.redb \
    ${criome}/bin/criome-handle-frame < ${parsed}/frame.bin > reply.bin
  mkdir -p $out
  cp reply.bin   $out/reply.bin
  cp sema.redb   $out/state.redb
''

# checks/roundtrip-render.nix
{ pkgs, inputs, system, flake, ... }:
let
  nexus   = inputs.nexus.packages.${system}.default;
  handled = flake.checks.${system}.roundtrip-handle;
in
pkgs.runCommand "roundtrip-render" { } ''
  ${nexus}/bin/nexus-render < ${handled}/reply.bin > output.txt
  mkdir -p $out
  cp output.txt $out/output.txt
''
```

A leaf assertion then greps the rendered text. Each phase is
content-addressed: change the parser's code → only `roundtrip-parse`
and its descendants rebuild; change the renderer → only the last
two rebuild.

## Choosing which pattern

| Question | Use |
|---|---|
| "Does state survive a process restart?" | daemon-mode chain |
| "Do bytes-on-disk decode in a fresh process?" | binary-stability chain |
| "Does the demo work end-to-end?" | a single monolithic `runCommand` is enough — no chain needed |
| "Where in the pipeline did this break?" | binary-stability chain (each phase fails distinctly) |

If you don't need either property, write a single
`runCommand` that does the whole thing. Adding chained
phases costs evaluation time and file-shuffling; only pay
that when you need the localisation or the cache-by-phase.

## Blueprint discovery

Per `lore/rust/nix-packaging.md`, blueprint
auto-discovers `.nix` files under `checks/` as flake outputs.
Cross-references between checks use `flake.checks.${system}.<name>`
inside the receiving file's `let` block — no `import` needed
between sibling check files. The shared step builder lives
under `lib/<name>.nix` and is exposed via
`flake.lib.<name>` from `lib/default.nix`.

## Pitfalls

- **State files inherit read-only mode** when copied from a
  prior derivation's `$out`. Use `install -m 644` (or `cp`
  followed by `chmod +w`) to copy in writable form.
- **Daemons must shut down before state is captured** so the
  underlying database flushes its write transaction. Trap
  SIGEXIT with `kill + wait` rather than letting the
  derivation fail and leave a partial state file.
- **`pkgs.runCommand`'s default shell strict mode** is
  permissive; always start scripts with `set -euo pipefail`.
- **Don't `tail -f` daemon logs** in a chained step — the
  derivation will hang. Start daemons in the background,
  poll for the socket file with a bounded loop, then proceed.
- **One-shot binaries don't need tokio** even if the daemon
  does. Single-frame dispatchers use plain `fn main()`; the
  async runtime is for the long-running listener.

## Reference implementations

- `workspace/lib/scenario.nix` — daemon-mode step builder
- `workspace/checks/scenario-*.nix` — daemon-mode chain (assert-node → query-nodes → chain)
- `workspace/checks/roundtrip-*.nix` — binary-stability chain (parse → handle → render → assert)
- `workspace/checks/integration.nix` — single-derivation monolithic shuttle (the simpler pattern)
