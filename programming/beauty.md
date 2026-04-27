---
source: (our own — distilled from mentci/reports/097)
fetched: 2026-04-27
---

# Beauty and elegance — the criterion

A short discipline for thinking about engineering: **beauty is
not a luxury laid on top of correct work — it is the test of
correctness.** Ugly code is evidence that the underlying
problem is unsolved. The aesthetic instinct — *make the
invariant visible, make the type carry meaning, make the dead
branch impossible to reach* — is what stands between a working
system and a coroner's report.

The full research backing this doc, with citations across two
millennia of philosophy and seven decades of software disasters,
lives in
[mentci reports/097](https://github.com/LiGoldragon/mentci/blob/main/reports/097-beauty-and-elegance-research-2026-04-27.md).
This file is the working document.

## The rule

When something feels ugly, you have a diagnostic reading: the
underlying structure has not yet been found. Slow down and find
it. Do not paper over the discomfort with a workaround.

Conversely: when the right structure is found, the ugliness
dissolves. Special cases collapse into the normal case.
Repetition resolves into a single named pattern. The diff
shrinks instead of growing. **This is the test.**

If you cannot make it beautiful, you do not understand it yet.

## Why beauty is the criterion (the practitioners)

Across mathematics, physics, and software, the people who have
actually done the work converge on the same claim. **Hardy,
1940**: "Beauty is the first test: there is no permanent place
in the world for ugly mathematics." **Dirac, 1963**: "It is
more important to have beauty in one's equations than to have
them fit experiment" — because the beautiful equation usually
survives as the deeper structure is filled in. **Hoare, 1980
Turing lecture**: "There are two ways of constructing a
software design: one way is to make it so simple that there are
obviously no deficiencies, and the other way is to make it so
complicated that there are no obvious deficiencies. The first
method is far more difficult." **Dijkstra (EWD648)**: "How do
we convince people that in programming simplicity and clarity —
in short: what mathematicians call elegance — are not a
dispensable luxury, but a crucial matter that decides between
success and failure?"

**Brooks, 1975**: "Conceptual integrity is the most important
consideration in system design. It is better to have a system
omit certain anomalous features and improvements, but to
reflect one set of design ideas, than to have one that contains
many good but independent and uncoordinated ideas."

**Wirth**: "A program should not only be correct, it should be
*obviously* correct."

**Torvalds, 2016 (TED, on linked-list deletion via `**` rather
than a head-special-case `if`)**: "Sometimes you can see a
problem in a different way and rewrite it so that a special
case goes away and becomes the normal case. And that's good
code."

**Hickey, 2011 (Simple Made Easy)**: complexity is what happens
when you *complect* — braid together — things that should be
separate. Simplicity is a moral discipline because it is what
makes systems reasonable about, and reasoning, not testing, is
what catches the bugs that matter.

Every figure here is making the same engineering claim in
different vocabulary: *an ugly solution is evidence of an
unsolved problem.*

## Why beauty is the criterion (the philosophical tradition)

This is not a software-engineering invention. It is the
dominant view of the philosophical tradition from Plato to
Christopher Alexander.

**Plato** treats beauty, truth, and goodness as a single Form
seen from different angles. **Aristotle** locates beauty in
*order, symmetry, and definiteness* and ties it to *telos* —
purpose: a thing is beautiful when it fulfils its function
well. **Plotinus** (*Ennead* I.6) describes the work of becoming
beautiful as subtraction: "as does the creator of a statue: he
cuts away here, he smooths there, he makes this line lighter,
this other purer, until a lovely face has grown upon his work."
The medieval scholastics formalized this: *integritas*
(wholeness — nothing missing that should be there),
*consonantia* (right proportion), and *claritas* (radiance —
the form shining through the matter). These map almost exactly
onto modern engineering virtues: completeness, coherence,
legibility.

**Wittgenstein** makes the move that matters most for
engineers. Aesthetic judgments, he insists, are not vague
effusions; serious criticism uses words like "right,"
"correct," "precise" — not "beautiful." The tailor who alters a
sleeve, the musician who hears that the chord wanted a minor
ninth not a minor seventh, the poet for whom only *this* word
will do: in each case the criterion is identifiable, and a
person trained in the practice can locate the fault. Aesthetic
competence is a skill of perceiving rightness, not a report of
inner glow.

**Christopher Alexander** (*The Nature of Order*, 2002–04)
argues that beauty is an *objective* property of designed
systems, rooted in geometric properties of "centers" — strong
centers, levels of scale, deep interlock, good shape, local
symmetries, gradients, simplicity-and-inner-calm,
not-separateness. He claims trained observers can rank pairs of
objects by which has more "life" and converge.

The unifying claim across two millennia: **beauty is objective,
perceivable by trained attention, convertible with truth and
goodness, and the criterion by which the rightness of a made
thing is judged.** To call something ugly is to claim it has
misjudged its own being. That is not an aesthetic complaint. It
is a structural one.

## Why this matters: the cost of ugly

Beauty in engineering is not a luxury because **ugly code has
killed people.** The cases below are not metaphors.

**Therac-25** (1985–87) — six fatal radiation overdoses. Race
conditions between operator input and beam-mode state, a
one-byte counter that wrapped to zero and bypassed safety
checks, and the deliberate replacement of mechanical interlocks
with software interlocks the prior hardware had silently
masked. Reused code, untyped flags, no named invariant. A
typed mode variable would have saved those lives.

**Ariane 5 Flight 501** (1996) — $370M lost in 37 seconds. A
64-bit floating-point velocity converted to a 16-bit signed
integer overflowed and triggered an unhandled exception. The
offending alignment routine was inherited from Ariane 4 and
"does not apply to Ariane 5… and it was maintained for
commonality reasons." Dead code retained, type assumptions
unjustified.

**Mars Climate Orbiter** (1999) — burned in the Martian
atmosphere because Lockheed software emitted thrust impulse in
pound-seconds while NASA navigation expected newton-seconds.
$327M lost. A `Newtons` and `PoundForce` newtype — twenty lines
of beauty — would have made the program refuse to compile.

**Heartbleed** (CVE-2014-0160) — a single missing bounds check
bled the secrets of half the web for two years. The attacker
sent a length field; the code believed the length. A
bounded-length type would have made the bug unspeakable.

**Knight Capital** (2012) — $460M lost in 45 minutes when an
eight-year-dormant code path called "Power Peg" was reanimated
by a deployment that reused a feature flag and missed one of
eight servers. Dead code retained, flags repurposed. The firm
did not survive.

**Boeing 737 MAX MCAS** (2018–19) — 346 deaths. MCAS depended
on a single Angle-of-Attack sensor with no cross-check, was
hidden from crew training documents, and could not be disabled
once triggered. Undisclosed coupling, missing redundancy,
single point of failure. *Design ugliness with body counts.*

**Toyota unintended acceleration** (2009–11) — Michael Barr's
expert testimony documented 81,514 MISRA-C violations, more
than 10,000 global variables, and a "kitchen-sink" task whose
death the watchdog could not detect. The jury returned
"reckless disregard."

**Northeast Blackout of 2003** — ~55 million without power,
~100 deaths attributed. Root-caused to a race condition in GE
Energy's XA/21 alarm system: "a couple of processes that were
in contention for a common data structure… they were both able
to get write access at the same time."

In every case, the engineering ugliness was named: dead code,
untyped quantities, hidden coupling, missing checks, unbounded
inputs, race conditions, undocumented behavior. A more
beautiful design — *make the invariant visible, make the type
carry meaning, make the dead branch impossible to reach* — would
have prevented the harm.

The aesthetic instinct is not decoration. It is what stands
between a working system and a coroner's report.

## How ugliness manifests in this codebase

The signals to watch for:

- **A name that doesn't read as English.** `pf`, `de`, `kd`,
  `tok`, `op`. Each abbreviation costs the reader one mental
  lookup per occurrence forever, to save the writer three
  keystrokes once. The "verbosity" objection to full words is
  itself a sign of training-data corruption — engineers and
  agents alike have been taught a wrong frame. See
  [the naming rule](../../mentci/AGENTS.md) and
  [reports/092](https://github.com/LiGoldragon/mentci/blob/main/reports/092-naming-research-and-rule.md).
- **A type whose wrapped field is `pub`.** `Slot(pub u64)` is
  a label, not an abstraction; the wrapper does no work. See
  [rust/style.md §"Domain values are types"](../rust/style.md).
- **A free function that should be a method.** A verb that
  could attach to a noun reads as a missing model. See
  [`programming/abstractions.md`](abstractions.md).
- **Dead code retained "for safety" or "for backward
  compatibility."** Ariane 5. Knight Capital. Delete it. The
  history is in `git log`.
- **Special cases stacked on the normal case.** Linus's
  linked-list lesson. Find the rewrite that makes the special
  case disappear.
- **A stringly-typed dispatch.** `match name.as_str()` over
  cases that should be a closed enum. Perfect-specificity
  violation. See
  [criome ARCHITECTURE.md §2 Invariant D](https://github.com/LiGoldragon/criome/blob/main/ARCHITECTURE.md).
- **A doc comment that explains *what* the code does.**
  Well-named code already explains what it does; the comment
  is a sign the names aren't carrying their weight.
- **A long function with multiple responsibilities.**
  Anti-Brooks-conceptual-integrity. Split it.
- **A boolean parameter at a call site.** `frob(x, true)`
  reads as gibberish; the call site has no idea what `true`
  means. Split into two functions or pass a typed enum.
- **Naming a thing for what it is *not*.** `non_root`,
  `non_empty`, `not_admin`. Negative names compose poorly.
  Find the positive name (`leaf`, `populated`, `member`).

Each of these is a *signal*, not a sin. Notice it, decide what
the underlying problem is, fix the underlying problem. The
ugliness goes away when the structure is right.

## The rule applied to identifier length

The most common slip in our codebase is the "verbosity"
objection to spelled-out names. Three things to internalize:

1. **A name is a contract with future readers.** Future
   readers include the agent reading it next session. The cost
   of mis-naming is paid every time the name is read. The
   benefit of saving three keystrokes is paid once.
2. **A name should be exactly as long as the noun it names.**
   `id` is fine when "id" is what we mean (a general-English
   acronym); `identifier` is required when we mean an
   identifier of a specific kind that has a meaning beyond
   "ID." `AssertOperation` reads as English; `AssertOp` is an
   abbreviation that has to be decoded.
3. **The aesthetic discomfort with long names is usually
   training-data drift, not informed judgment.** When
   `AssertOperation` "feels needlessly verbose," that feeling
   is the signal to question the feeling — not the signal to
   shorten the name. The name reads as English; the discomfort
   reads as inherited prejudice.

Per Li (2026-04-27): *"You were taught wrong."* The rule is
beauty, not keystroke-economy.

## The one-line summary

**If it isn't beautiful, it isn't done. Slow down and find the
structure that makes it beautiful — that structure is the one
you were missing.**
