---
source: (our own) — distilled from rkyv 0.8 source + workspace usage
fetched: 2026-04-28
---

# Rkyv in CANON projects

[`rkyv`](https://crates.io/crates/rkyv) 0.8 is the binary contract format
between Rust components: storage (zero-copy reads), wire envelopes, IPC,
durable records. Plain text formats use `nota-codec`.
Serde appears only at external boundaries that demand it (legacy JSON).

## The canonical feature set

Every rkyv-using crate pins the same Cargo dependency — character-for-character:

```toml
rkyv = { version = "0.8", default-features = false, features = [
    "std",
    "bytecheck",
    "little_endian",
    "pointer_width_32",
    "unaligned",
] }
```

Each feature is load-bearing for portability:

| Feature | What it does | Why required |
|---|---|---|
| `std` | `Vec`, `String`, `HashMap`, etc. archive impls | Daemons run on real machines; not no_std |
| `bytecheck` | Validation on read; rejects malformed/malicious bytes | Wire is untrusted in principle; capability tokens cross machines later |
| `little_endian` | Pin archived integer byte order | Without this, archived integers use host endianness; cross-machine archives break |
| `pointer_width_32` | RelPtr offsets are 4 bytes | Without this, offsets follow host pointer width (4 vs 8); cross-machine archives break. 32-bit fits any single archive < 4 GiB |
| `unaligned` | Archived types readable from arbitrary byte offsets | Receiver buffers may not align; UDS gives raw bytes; without this the receiver must align before reading |

**Mandatory feature parity.** Archived types interop only when both ends
compile against the *same* feature set. A crate that adds or drops a
feature breaks archive compatibility silently — no compile error, just
wrong bytes. The exact-string match in `Cargo.toml` is the discipline.

**Pinned to rkyv 0.8.x.** Minor versions within 0.8 don't change archive
layout per rkyv's stability guarantees. A 0.9 jump would require a
coordinated upgrade across every rkyv-using crate.

## Derive pattern — alias to avoid serde clash

When a type carries both rkyv and serde derives (typical for records that
also need text rendering), import rkyv's `Serialize` / `Deserialize` under
aliases so the symbols don't clash with serde's:

```rust
use rkyv::{Archive, Deserialize as RkyvDeserialize, Serialize as RkyvSerialize};
use serde::{Deserialize, Serialize};

#[derive(
    Archive,
    RkyvSerialize,
    RkyvDeserialize,
    Serialize,        // serde, for text codecs / debug
    Deserialize,
    Debug, Clone, PartialEq, Eq, Hash,
)]
pub struct Foo {
    pub field: u32,
}
```

For pure-rkyv types (no serde at all), drop the aliases and the serde
derives.

## Encode / decode

rkyv 0.8's high-level API (re-exported at the crate root from
`api::high::*`):

```rust
// Encode: returns AlignedVec; Deref<Target=[u8]> works for sockets.
let bytes = rkyv::to_bytes::<rkyv::rancor::Error>(&value)?;

// Decode: validates + deserialises in one call.
let value: T = rkyv::from_bytes::<T, rkyv::rancor::Error>(bytes)?;
```

For zero-copy reads where ownership transfer isn't needed:

```rust
// Validated borrow into the buffer; returns &Archived<T>.
let archived = rkyv::access::<ArchivedT, rkyv::rancor::Error>(bytes)?;
// Field reads are direct memory access on archived bytes.
```

The error type `rkyv::rancor::Error` is the boxed-error variant. For
protocol code, catch any rkyv error and map to the protocol's own error
enum (e.g. `FrameDecodeError::BadArchive`) so callers don't depend on
rkyv's internal error shape.

## Type adaptations

Most stdlib types archive natively: `String`, `Vec<T>`, `Option<T>`,
`Box<T>`, `BTreeMap<K,V>`, primitives, fixed-size arrays.

Some don't and need adapters:

- **`PathBuf` / `OsString`** — bytes on Unix; UTF-16-ish on Windows.
  Don't archive these directly. Wrap in a newtype (`WirePath(Vec<u8>)`
  shape) so the wire form is deterministic and platform-independent.
- **Recursive types** — rkyv handles cycles via `Box`/`Rc` with
  shared-pointer support enabled by default in `std`.
- **Foreign types you don't own** — use `#[rkyv(with = ...)]` per the
  rkyv `with` module. Avoid where possible; wrap in your own newtype
  instead so the schema is yours.

## Schema evolution — the limit

rkyv archives are schema-fragile. Adding / removing / reordering fields
changes the archive bytes. Two consequences:

1. **No silent backward compatibility.** Old binaries can't read new
   archives or vice versa. Schema changes are coordinated upgrades.
2. **Use a version-skew guard.** A known-slot record carrying
   schema-version + wire-format-version, checked at boot. Hard-fail on
   mismatch. rkyv's own version handling is not enough.

Within a single rkyv 0.8 release, reordering struct fields *also* changes
the archive layout. Discipline: append-only field additions as a soft
convention; treat any change as breaking for safety.

## Wire framing

The frame schema *is* the framing. Both parties know the rkyv schema; the
wire is a stream of `Archived<Frame>` instances. `from_bytes::<Frame, E>`
validates + deserialises a single frame from a byte slice. Splitting a
TCP/UDS stream into per-frame slices is the transport layer's concern —
typically a 4-byte big-endian length prefix per archived frame.

## See also

- [rkyv 0.8 docs](https://docs.rs/rkyv/0.8) — full API reference
- [`style.md` §Serialization](style.md) — when to reach for rkyv vs nota-codec vs serde
