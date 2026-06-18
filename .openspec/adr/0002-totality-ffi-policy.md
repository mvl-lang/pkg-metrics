# ADR-0002: Totality and FFI Policy — Builtin Functions Are Implicitly Total

**Status:** Accepted
**Date:** 2026-06-18
**Context:** pkg-metrics exposes all operations as `builtin fn`. MVL can infer totality (`total*`) for functions it can inspect, but `builtin fn` bodies are opaque. We investigated adding explicit `total fn` qualifiers (as pkg-http and pkg-sqlite do for pure MVL functions) but the MVL compiler rejects `total builtin fn` with: `` `builtin` cannot be combined with `total` or `partial` ``.

## Decision

**`builtin fn` in MVL is implicitly total.** The compiler does not permit explicit `total`/`partial` qualifiers on builtins. All functions in pkg-metrics are `builtin fn` and are therefore implicitly total in the assurance report.

The totality contract is documented per-function in doc comments (e.g. "O(1) atomic increment, no loops") rather than as a compiler-checked annotation. Code review of the Rust bridge is the enforcement mechanism.

## Why builtins are implicitly total

The MVL runtime contract for `builtin fn` requires the implementation to be:
- Terminating: no unbounded blocking in the caller's thread
- Free of untracked effects: any side effect must be declared in the effect signature

This is a runtime ABI requirement, not a source-level proof. The compiler trusts the runtime to uphold it. Functions that would block (e.g., a hypothetical flush-and-wait) must not be declared `builtin fn` — they would instead be `extern "rust"` with an explicit effect.

## Per-function termination argument

| Function | Why it terminates |
|---|---|
| `counter_inc`, `counter_add` | Atomic integer increment; no loops, no allocation |
| `gauge_set`, `gauge_inc`, `gauge_dec` | Atomic float write; O(1) |
| `histogram_observe` | Bounded bucket scan (fixed at registration) + atomic increment |
| `histogram_time` | Calls `f` once, records monotonic clock delta; O(1) wrapper overhead |
| `start_prometheus_exporter` | Binds a TCP socket and spawns a background thread; the call itself returns |

## Contrast with pkg-sqlite and pkg-http

pkg-sqlite and pkg-http use `extern "rust"` FFI blocks where `total fn` annotations are valid and verified (for the MVL-layer wrapper, not the Rust body). pkg-metrics uses `builtin fn` exclusively — a different FFI mechanism where totality is implicit and verified only by the runtime ABI contract.

## Consequences

- No explicit `total fn` keywords appear in `src/metrics.mvl`; this is correct and expected
- `make assurance` will report all builtins as implicitly total
- Any new builtin that could block must be rejected in review; blocking operations require `extern "rust"` with an appropriate effect annotation
- The totality argument lives in doc comments and this ADR, not in compiler output

## Connected to

- ADR-0001: Metrics API design — motivation for the builtin approach
- pkg-http ADR-0002: Totality policy for pure MVL functions (different mechanism)
- pkg-sqlite ADR-0003: Totality policy for `extern "rust"` functions (different mechanism)
- MVL ADR-0006: FFI extern "rust" trust boundary model
- `src/metrics.mvl` — all `builtin fn` declarations with termination doc comments
