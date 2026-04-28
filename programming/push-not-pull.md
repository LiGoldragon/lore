---
source: (our own)
fetched: 2026-04-29
---

# Push, not pull — events flow toward the consumer

When a system has a producer of state and a consumer of changes,
the producer **pushes** updates to the consumer; the consumer
does **not poll** the producer.

## The rule

**Polling is wrong. Always.**

When a UI needs to reflect database state, the database tells
the UI what changed. When a daemon needs to react to client
requests, the client connects and sends; the daemon doesn't
sweep a mailbox directory. When an actor needs to know a child
failed, the supervisor receives a supervision event; it doesn't
heartbeat-tick its children to ask "still alive?"

If you find yourself reaching for a polling loop, stop. The
producer should expose a subscription primitive instead.

## Why

- **Polling burns latency.** A consumer polling at interval *N*
  has worst-case latency *N* between event and reaction. Pushed
  events arrive at the speed of the medium.
- **Polling burns work.** Most polls are no-ops; the work is
  wasted before the loop body runs.
- **Polling encodes the consumer's pacing into the producer's
  protocol.** The consumer says "I'll check every 1Hz" — but
  the right cadence is "whenever something happens." The push
  model says exactly that.
- **Polling fakes change-detection.** Two consecutive snapshots
  identical doesn't mean nothing happened — it means nothing
  *visible to the consumer's projection* happened, which is not
  the same thing.

## How

Producers expose a subscription primitive: register a callback,
open a stream, or accept a long-lived RPC. Consumers subscribe
once and receive events indefinitely.

In actor systems, each actor's mailbox is already a push
channel; nothing in the actor model requires polling. In
databases with change feeds, subscribe to the feed and react to
deltas. In UIs over a backing store, the store emits change
events and the UI rerenders.

If the producer can't yet push (the subscription primitive
isn't built), the consumer **defers its real-time feature**
rather than fall back to polling. A feature that depends on
live updates simply waits for the push primitive to ship; it
does not paper over the gap with a poll loop "for now." A poll
"for now" never gets removed.

## When polling is acceptable

There are no general exceptions. Two narrow shapes that look
like polling but aren't:

- **Reachability probes** with explicit transport-layer
  semantics — a monitoring agent that periodically checks "is
  service X reachable" is doing transport-layer reachability,
  not state-change detection. The contract is "are you alive,"
  not "what changed."
- **Backpressure-aware pacing** where the consumer decides
  *when* it can accept the next push (rate-limiting itself).
  The producer still pushes; the consumer just drains its
  buffer at its own pace. This is a flow-control concern, not
  a polling one.

If you're not sure which case you're in, you're in the polling
case. Default to push.

## See also

- [`programming/abstractions.md`](abstractions.md) — every
  reusable verb belongs to a noun (the same discipline applied
  to behavior dispatch).
- [`programming/beauty.md`](beauty.md) — when polling feels
  necessary, the right structure usually hasn't been found yet.
