# ADR-0001: Metrics API Design — Builtins, `! Metric` Effect, and Prometheus Naming

**Status:** Accepted
**Date:** 2026-06-18
**Context:** pkg-metrics provides counters, gauges, and histograms as the metrics pillar of MVL's observability triangle (logs / metrics / trace). The design question is: how should metric operations be surfaced in MVL, and what naming discipline should callers follow?

## Decision

**All metric emit operations are `total builtin fn` with the `! Metric` effect.** No pure MVL wrappers are introduced. The API follows Prometheus naming conventions enforced by documentation comment and linter warning (not type system).

Specifically:

- `counter_inc`, `counter_add` — counter mutation, `! Metric`
- `gauge_set`, `gauge_inc`, `gauge_dec` — gauge mutation, `! Metric`
- `histogram_observe` — single observation, `! Metric`
- `histogram_time[T, E]` — effect-polymorphic timer wrapper, `! E + Metric`
- `start_prometheus_exporter` — lifecycle, `! Net + Metric`

Types `Counter`, `Gauge`, `Bucket`, and `Histogram` are pure MVL structs — they serve as documentation types and potential return types for future snapshot queries. They carry no FFI state.

## Rationale

### Builtins, not pure MVL

Metric operations interact with a process-global registry (Prometheus, StatsD, OTLP). There is no pure MVL representation of this mutable global state. Making operations `builtin` is honest: the implementation lives in the runtime, and MVL's type system tracks *that* an effect happens (via `! Metric`) without pretending it can verify *what* the runtime does.

### `! Metric` effect

A dedicated `Metric` effect row makes metric emission visible in call-site signatures. Functions that emit metrics cannot be silently composed with code that requires a pure `fn`; the effect must be threaded explicitly. This mirrors how `! IO` and `! Net` work in the MVL stdlib.

### `histogram_time` is effect-polymorphic

`histogram_time[T, E]` accepts any `fn() -> T ! E` and returns `T ! E + Metric`. The wrapper itself is `total fn` — it calls `f` once, records the elapsed time, and returns. The totality annotation applies to the wrapper's own structure; it does not claim `f` terminates.

### Prometheus naming conventions

Prometheus is the dominant pull-based metrics system. Following its naming conventions (`snake_case`, `_total` suffix for counters, `_seconds`/`_bytes` unit suffixes, static names with tag-based variation) maximises compatibility with Grafana, Alertmanager, and recording rules without any runtime cost.

## Consequences

- All metric emit functions appear with `! Metric` in signatures — no silent side effects
- `histogram_time` is the idiomatic way to time a code block; direct `start`/`stop` timing is not provided
- Callers must not construct dynamic metric names (linter warning in `mvl check`)
- The `Counter`, `Gauge`, `Histogram` types are available for documentation and snapshot use; they are not connected to the live registry in 0.2.0

## Connected to

- [#805](https://github.com/LAB271/mvl_language/issues/805) — Original design ticket
- ADR-0002: Totality and FFI policy for this package
- MVL ADR-0006: FFI extern "rust" trust boundary
- `src/metrics.mvl` — the full API surface
