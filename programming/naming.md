---
source: (our own)
fetched: 2026-04-30
---

# Naming — full words by default

Identifiers are read far more than they are written. Cryptic
abbreviations optimize for the writer (a few keystrokes saved) at
the reader's expense (one mental lookup per occurrence).

**Default: spell every identifier as full English words.**

Examples (bad → good):

| bad | good |
|---|---|
| `lex` | `lexer` |
| `tok` | `token` |
| `ident` | `identifier` |
| `op` | `operation` (or specific: `assert_op`) |
| `de` | `deserializer` |
| `pf` | `pattern_field` |
| `ctx` | `context` (or specific: `parse_context`) |
| `cfg` | `config` (or `configuration`) |
| `addr` | `address` |
| `buf` | `buffer` |
| `tmp` | `temporary` (or — better — name what it holds) |
| `arr` | `array` (or — better — what it contains) |
| `obj` | (name what it actually is) |
| `params` | `parameters` |
| `args` | `arguments` |
| `vars` | `variables` |
| `proc` | `procedure` or `process` |
| `calc` | `calculate` |
| `init` | `initialize` |
| `repr` | `representation` |
| `gen` | `generate` or `generator` |
| `ser` / `deser` | `serialize` / `deserialize` |

## Permitted exceptions — tight, named, no others

1. **Loop counters in tight scopes (<10 lines).** `for i in 0..n`
   is fine. Beyond ~10 lines or nested, use descriptive names.
2. **Mathematical contexts** where the math itself uses the symbol.
   `x`, `y`, `z`, `theta`, `phi`, `lambda`, `n` for sample size,
   `p` for probability — only when the surrounding code or comment
   establishes the math context.
3. **Generic type parameters.** `T`, `U`, `V`, `K`, `E`. Use a
   descriptive name when the parameter has non-trivial semantic
   content.
4. **Acronyms that have passed into general English.** `id`, `url`,
   `http`, `json`, `uuid`, `db`, `os`, `cpu`, `ram`, `io`, `ui`,
   `tcp`, `udp`, `dns`. Spell them when ambiguous in context.
5. **Names inherited from `std` or well-known libraries.** `Vec`,
   `HashMap`, `Arc`, `Rc`, `Box`, `Cell`, `RefCell`, `Mutex`,
   `mpsc`, `regex`. Do not rename these; do *not* extend the
   abbreviation pattern to your own types.
6. **Domain-standard short names already documented in an
   `ARCHITECTURE.md`.** `slot`, `node`, `edge`, `frame` are full
   words and need no exception. If a true short form is load-bearing
   in the schema, name it in `ARCHITECTURE.md` so the exception is
   explicit; otherwise spell it out.

## Rule of thumb

**Name length proportional to scope.** A 3-line loop counter can
be `i`. A module-level type that appears across the codebase must
spell itself out. A function parameter that lives for 50 lines
must read as English.

## What this rule is NOT

- Not "verbose names everywhere" —
  `calculate_the_total_amount_of_items` is worse than
  `total_items`. The goal is *clear*, not *long*.
- Not "no acronyms ever" — see exception 4.
- Not "rewrite std" — see exception 5.

## How to apply when generating code

When generating new code: **spell identifiers as full English
words by default.** When the surrounding code uses cryptic
identifiers, do not propagate them into new code. Either rename
(if rename is in scope) or use the full form for new identifiers
and flag the inconsistency as a follow-up. Pattern-matching the
local dialect is exactly the failure mode this rule exists to
break.

## The "feels too verbose" anti-pattern

When a spelled-out name (`AssertOperation`, `Deserializer`,
`PatternField`, `RelationKind`) "feels needlessly verbose" — that
feeling is **not** a signal to shorten the name. It is a signal
that the writer has been taught wrong by a culture inherited from
constraints that no longer apply.

The full word reads as English. The abbreviation reads as ceremony
to be decoded. The cost of mis-naming is paid every time the name
is read; the benefit of saving three keystrokes is paid once.
There is no contest.

When you catch yourself thinking "this name feels too long" or
"this is unnecessarily ceremonial":

1. **Question the feeling.** It is almost certainly inherited
   prejudice, not informed judgment.
2. **Re-read the name as English.** Does `AssertOperation` read as
   English? (Yes.) Does `AssertOp` read as English? (No — it
   requires expansion.)
3. **Apply the rule.** The full English form wins unless the name
   falls in one of the six named exception classes above.

There is no exception class for "feels verbose." That feeling is
the bug, not the criterion.

## Companion rule

Pairs with [beauty.md](beauty.md): a name that doesn't read as
English is one of the diagnostic readings of structural ugliness.
The aesthetic discomfort is the signal that the right structure
(the right name, the right type) hasn't been found.
