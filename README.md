# pkg-metrics

MVL metrics package — counters, gauges, and histograms with effect tracking.

## Installation

Add to your `mvl.toml`:

```toml
[dependencies]
metrics = { git = "https://github.com/mvl-lang/pkg-metrics" }
```

## Usage

```mvl
use pkg.metrics.{counter_inc, gauge_set, histogram_observe}

fn handle_request(req: Request) -> Response ! Net + Metric {
    counter_inc("http_requests_total", { "method": req.method });
    
    // ... handle request ...
    
    counter_inc("http_responses_total", { "status": "200" });
    response
}
```

## Features

- **Counters**: Monotonically increasing values (`http_requests_total`)
- **Gauges**: Values that go up and down (`connections_active`)
- **Histograms**: Distribution of values (`request_duration_ms`)
- **Effect tracking**: `! Metric` in function signatures
- **Type-safe tags**: `Map[String, String]` for labels

## Export Formats

- Prometheus (`/metrics` endpoint)
- StatsD (UDP push)
- OTLP (OpenTelemetry)

## Related

- [#805](https://github.com/LAB271/mvl_language/issues/805) — Original design ticket
- [pkg-health](https://github.com/mvl-lang/pkg-health) — Health checks
- [pkg-trace](https://github.com/mvl-lang/pkg-trace) — Distributed tracing

## License

Apache-2.0
