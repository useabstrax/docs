# Monitoring plugin

Collects host, filesystem, and process metrics on the server and keeps a local history. Query it with `abstrax monitoring`. Samples stay on the machine, including a small `latest.json` vitals file.

| Item | Value |
|---|---|
| Binary | `abstrax-monitoring` |
| CLI | `abstrax monitoring …` |
| Trust level | `official` |
| Platforms | The [Abstrax platform matrix](/docs/reference/supported-platforms). Release binaries are `linux-amd64` and `linux-arm64`. |

Collectors read `/proc` and `/sys`. They do not have their own distribution list. `monitoring setup` asks `abstrax doctor --json` and stops when the CLI reports the host is unsupported.

## Install

```bash
sudo abstrax plugin install monitoring
sudo abstrax monitoring setup
abstrax monitoring status
abstrax monitoring latest
```

`setup` writes `/etc/abstrax/plugins/monitoring/config.yaml` when that file is missing, installs `abstrax-monitoring.service`, and waits for the first sample. It does not replace a config that is already there.

The daemon runs as root. The database, socket, and `latest.json` are mode `0600` under `/var/lib/abstrax/plugins/monitoring/`. Other users cannot open them. Command lines are truncated in storage and are not logged.

## Commands

| Command | Purpose |
|---|---|
| `monitoring setup` | Directories, default config, systemd, first sample |
| `monitoring config` | Show or change intervals, retention, rollups, and the storage ceiling |
| `monitoring start` / `stop` / `restart` | systemd control |
| `monitoring status` | Daemon health. Stopped is a successful answer, not a failed command |
| `monitoring latest` | Current host, filesystems, and top processes |
| `monitoring history --from --to` | One page of aligned host and filesystem history |
| `monitoring processes` | Live processes, or `--at` a past time |
| `monitoring process <instance-id>` | One process, including after it has exited |
| `monitoring process history <instance-id> --from --to` | History for that instance |
| `monitoring live` | NDJSON samples. Use `--json-stream` |
| `monitoring storage status` | Size, oldest sample, and whether the disk ceiling is binding |

`--json` wraps a command in the usual Abstrax result envelope. `data.schema_version` is `1`. `monitoring live` prints raw sample lines (`"type": "sample"`) and rejects `--json`.

History takes `--max-points` (default 240) and an opaque `--cursor`. A page is one resolution for both host and filesystem series. The cursor fails if that data has been deleted or rolled up.

## Retention

Defaults: raw samples for 1 hour, then 15s for 6 hours, 1m for 24 hours, 5m for 7 days, and 1h out to 90 days. Sampling defaults to 5s for host and processes and 60s for filesystems. Sub-second polling is rejected.

Rates are rolled up from byte deltas and duration, not by averaging the rates. The storage ceiling defaults to `1GB` (1000000000 bytes) and wins over the time horizon: the oldest rows are dropped, collection continues, and `storage status` shows both the configured retention and the oldest sample still on disk.

## What it does not do

No alerts, logs, journald, remote shell, process kill, or export to Prometheus, Netdata, OpenTelemetry, or Grafana. Per-core, per-interface, and per-device history are not stored in v1.

## Service

`abstrax-monitoring.service` restarts on failure, logs to the journal, and does not restart when `run` exits 78 because the config is invalid. `systemctl reload` is not used; send SIGHUP, or run `monitoring config`, to reload a changed file.

```bash
sudo abstrax monitoring install-service
sudo abstrax monitoring restart
journalctl -u abstrax-monitoring -f
```
