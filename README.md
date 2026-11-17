# aspire-otel-demo

A local demo that runs the [.NET Aspire dashboard](https://learn.microsoft.com/dotnet/aspire/fundamentals/dashboard/standalone)
as an OpenTelemetry backend, with an [OpenTelemetry Collector](https://opentelemetry.io/docs/collector/) in front of it.
[`telemetrygen`](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/cmd/telemetrygen)
creates test traces, metrics and logs.

```
telemetrygen ──OTLP gRPC──▶ otel-collector:4317 ──OTLP gRPC──▶ aspire-dashboard:18889
                             (resource + batch)                 UI on http://localhost:18888
```

All telemetry goes through the collector. Aspire's OTLP port isn't published to the host.

## Prerequisites

- Docker with Compose v2
- [Task](https://taskfile.dev) (`brew install go-task`)

## Usage

```bash
task demo            # start the stack, send traces + metrics + logs
task open            # open the Aspire dashboard
```

| Task | What it does |
|------|--------------|
| `task up` | Start the Aspire dashboard and the collector |
| `task gen` | Send traces, metrics and logs in parallel |
| `task gen:traces` / `gen:metrics` / `gen:logs` | Send one signal |
| `task logs` | Follow the collector logs (the `debug` exporter prints each batch) |
| `task restart` | Recreate the stack, for example after editing `otel-collector/config.yaml` |
| `task down` | Stop and remove everything |

The generator tasks accept `SERVICE`, `DURATION` and `RATE` (items per second):

```bash
SERVICE=checkout DURATION=30s RATE=20 task gen:traces
```

## Ports

Host ports are set in `.env`. The collector ports are moved away from the usual 4317/4318
so they don't clash with a collector you may already run locally.

| Port | Service |
|------|---------|
| 18888 | Aspire dashboard UI |
| 14317 | Collector OTLP gRPC |
| 14318 | Collector OTLP HTTP |
| 13134 | Collector health check |

To send telemetry from an app on your machine, point it at the collector:
`OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:14317`.

## Notes

- Authentication is turned off for both the dashboard UI and its OTLP endpoint (`Unsecured`). This is for local demos only.
- The Aspire dashboard keeps telemetry in memory. `task restart` clears it.
- Image versions are pinned in `.env`.
