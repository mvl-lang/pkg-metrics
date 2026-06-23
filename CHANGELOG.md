# Changelog

## [0.3.0] - 2026-06-23

### Changed
- **Breaking**: replaced `builtin fn` free functions with an actor-based pure MVL implementation (#3)
  - `pub actor Metrics` — registry actor holding all metric state
  - `new_metrics() -> Metrics ! Spawn` — explicit constructor, call once at startup
  - All emit operations are now actor behaviors called as `metrics.counter_inc(...)` etc.
  - `histogram_time` and `start_prometheus_exporter` are now free functions taking `Metrics` as first argument
  - `pub effect Metric > Send + Clock` — subsumes actor messaging and clock effects
- `start_prometheus_exporter` now takes `(metrics, host, port)` instead of `addr: String`; returns `Result[Unit, NetError]`
- `histogram_time` now takes `metrics` as first argument; duration measured via `std.time.elapsed_ms`
- Prometheus exporter implemented in pure MVL using `std.net`; serves `GET /metrics`, returns 404 otherwise

### Added
- `with mailbox(unbounded)` on the registry actor — prevents backpressure drops under high metric volume
- Causal consistency: scrape responses are formatted after all prior emit messages in the mailbox

### Requires
- `std.time.elapsed_ms` — added to the MVL stdlib alongside this release

## [0.2.1] - 2026-06-20

### Fixed
- Declare `pub effect Metric` — was used but never declared, causing downstream packages that subsume it to fail with "unknown effect" errors

## [0.2.0] - 2026-06-18

### Added
- `Makefile` with `check`, `test`, `coverage`, `prove`, `assurance`, `version`, and `clean` targets
- `.openspec/adr/0001-metrics-api-design.md` — documents why builtins, `! Metric` effect, and Prometheus-convention naming
- `.openspec/adr/0002-totality-ffi-policy.md` — documents totality and FFI trust-boundary policy for the metrics package
- `total fn` annotations on all `builtin fn` declarations; `histogram_time` marked `total fn` (effect-polymorphic, but terminates)

### Changed
- Bumped version `0.1.0 → 0.2.0` (new Makefile, ADRs, totality annotations)

## [0.1.0] - 2026-06-18

### Added
- Initial metrics package scaffolding: `Counter`, `Gauge`, `Bucket`, `Histogram` types
- `counter_inc`, `counter_add`, `gauge_set`, `gauge_inc`, `gauge_dec`, `histogram_observe`, `histogram_time` builtins
- `start_prometheus_exporter` builtin
- Effect system integration: `! Metric` on all emit operations
