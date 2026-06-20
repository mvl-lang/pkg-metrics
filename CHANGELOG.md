# Changelog

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
