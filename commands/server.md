# Server

The `server` group reports server status and resource usage. These commands are read-only and do not require root.

```text
abstrax server <action> [flags]
```

## `server status`

Show a comprehensive overview.

```bash
abstrax server status
```

The output includes hostname, uptime, timezone, load average, CPU cores, memory usage, swap usage (if present), OS information, kernel version, private IP addresses, and disk usage.

```text
  Hostname:            web01
  Uptime:              up 3 days, 4 hours
  Timezone:            Europe/London (GMT, +00:00)
  Load average:        0.15  0.10  0.05

  CPU:                 4 cores
  Memory:              7976 MB total / 2104 MB used (26.4%)
  Swap:                2048 MB total / 0 MB used (0.0%)

  OS:    Ubuntu 22.04.3 LTS
  Kernel: 5.15.0-91-generic

  Private IPs:
    10.0.0.5

  Disk usage:
    /                    12.3 GB / 50.0 GB (25%)
```

The timezone is the zone the server is using now. `name` is an IANA name such as `Europe/London`. `abbreviation` and `utc_offset` follow that zone at the current time, so a zone with daylight saving changes between winter and summer. `--json` returns:

```json
"timezone": { "name": "Europe/London", "abbreviation": "GMT", "utc_offset": "+00:00", "utc_offset_seconds": 0 }
```

Abstrax reads the name from `timedatectl` when that command is available. Otherwise it follows the `/etc/localtime` symlink, then `/etc/timezone`. If none of those exist, the name is `UTC`. This command does not change the timezone.

## Other server commands

```bash
abstrax server cpu
abstrax server memory
abstrax server disk
abstrax server disk --path=/var/www
abstrax server load
abstrax server services
abstrax server services --failed
```

| Command | Description |
|---|---|
| `server cpu` | Number of CPU cores |
| `server memory` | Total, used, free memory and usage percentage |
| `server disk` | Disk usage per mount point; `--path` limits to one path |
| `server load` | Load average (1, 5, 15 minutes) |
| `server services` | List services; `--failed` shows only failed services |

## Related

- [Daemons](/docs/commands/daemons) - for Supervisor-managed processes
- [Packages](/docs/commands/packages)
- [Configuration](/docs/configuration/index)
- [Security](/docs/reference/security)
