---
source: (our own)
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
  restarts.
- Schema migration testing: state-V1 → daemon v2 reads it →
  state-V2.
- Bootstrap-loop testing: engine v1 stores records → engine v2
  (a different binary build) reads them.

**Binary-stability chain — per-transformation isolation.**
Each step runs ONE one-shot binary, capturing its raw byte
output. The next step consumes those bytes as input. Tests
that *binary serialisation* is stable across process and
version boundaries. Use when:

- Verifying that a binary wire format (rkyv, etc.) is stable
  across process boundaries — bytes written to disk by one
  process must decode in another.
- Per-daemon-transformation isolation: parser bug, handler
  bug, renderer bug each fail the specific phase that
  produces them.
- Cross-version binary compatibility: producer-version A,
  consumer-version B.

The two patterns are complementary. Daemon-mode tests the
*application contract*; binary-stability tests the *wire
contract*.

## Skeleton — daemon-mode step builder

A step builder factored as a Nix function. Spawns the daemon
graph in a sandbox, pipes user-text through a CLI, captures
both the rendered reply text and the resulting state file
into the step's `$out`.

Substitute your real daemon and CLI names for `engine`,
`frontend`, `frontend-cli`. The state file's name (`state.db`
below) is whatever your engine writes.

```nix
# lib/scenario.nix
{ pkgs, engine, frontend, frontend-cli }:

{ name, input, priorState ? null }:

pkgs.runCommand name { } ''
  set -euo pipefail
  cd $TMPDIR

  ${pkgs.lib.optionalString (priorState != null)
    "install -m 644 ${priorState}/state.db state.db"}

  cleanup() {
    kill ''${frontend_pid:-} ''${engine_pid:-} 2>/dev/null || true
    wait ''${frontend_pid:-} ''${engine_pid:-} 2>/dev/null || true
  }
  trap cleanup EXIT

  ENGINE_SOCKET=$PWD/engine.sock STATE_PATH=$PWD/state.db \
    ${engine}/bin/engine-daemon &
  engine_pid=$!
  for i in $(seq 1 50); do [ -S "$PWD/engine.sock" ] && break; sleep 0.1; done

  FRONTEND_SOCKET=$PWD/frontend.sock ENGINE_SOCKET=$PWD/engine.sock \
    ${frontend}/bin/frontend-daemon &
  frontend_pid=$!
  for i in $(seq 1 50); do [ -S "$PWD/frontend.sock" ] && break; sleep 0.1; done

  echo ${pkgs.lib.escapeShellArg input} | \
    FRONTEND_SOCKET=$PWD/frontend.sock ${frontend-cli}/bin/frontend > response.txt

  # Stop daemons cleanly so state flushes.
  kill $frontend_pid; wait $frontend_pid 2>/dev/null || true
  kill $engine_pid; wait $engine_pid 2>/dev/null || true

  mkdir -p $out
  cp response.txt $out/response.txt
  cp state.db     $out/state.db
''
```

A leaf check then chains steps + asserts:

```nix
let
  step = flake.lib.scenario {
    inherit pkgs;
    engine = ...; frontend = ...; frontend-cli = ...;
  };
  firstStep = step { name = "first-step"; input = ''<your input>''; };
  secondStep = step {
    name       = "second-step";
    input      = ''<your input>'';
    priorState = firstStep;        # ← state.db chains forward
  };
in
pkgs.runCommand "scenario-chain" { } ''
  grep -qE '<expected output regex>' ${firstStep}/response.txt  || exit 1
  grep -qE '<expected output regex>' ${secondStep}/response.txt || exit 1
  touch $out
''
```

## Skeleton — binary-stability per-phase step

For per-daemon-transformation testing, each step runs ONE
one-shot binary. The daemons need to expose a one-shot mode
— `<crate>-<verb>` binaries that read stdin and write stdout.

```nix
# checks/roundtrip-parse.nix
{ pkgs, inputs, system, ... }:
let
  frontend = inputs.frontend.packages.${system}.default;
  input = pkgs.writeText "input" ''<your input>'';
in
pkgs.runCommand "roundtrip-parse" { } ''
  ${frontend}/bin/frontend-parse < ${input} > frame.bin
  mkdir -p $out
  cp frame.bin $out/frame.bin
''

# checks/roundtrip-handle.nix
{ pkgs, inputs, system, flake, ... }:
let
  engine = inputs.engine.packages.${system}.default;
  parsed = flake.checks.${system}.roundtrip-parse;
in
pkgs.runCommand "roundtrip-handle" { } ''
  cd $TMPDIR
  STATE_PATH=$PWD/state.db \
    ${engine}/bin/engine-handle-frame < ${parsed}/frame.bin > reply.bin
  mkdir -p $out
  cp reply.bin   $out/reply.bin
  cp state.db    $out/state.db
''

# checks/roundtrip-render.nix
{ pkgs, inputs, system, flake, ... }:
let
  frontend = inputs.frontend.packages.${system}.default;
  handled  = flake.checks.${system}.roundtrip-handle;
in
pkgs.runCommand "roundtrip-render" { } ''
  ${frontend}/bin/frontend-render < ${handled}/reply.bin > output.txt
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

Per [`rust/nix-packaging.md`](../rust/nix-packaging.md), blueprint
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
