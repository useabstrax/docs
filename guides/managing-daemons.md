# Managing daemons

This guide runs a long-lived process under Supervisor, then checks its status and logs. Use daemons for processes that should run continuously, such as queue workers or websocket servers. For scheduled tasks, use [cron](/docs/guides/managing-cron-jobs) instead.

Adding, modifying, removing, and controlling daemons requires root. Supervisor must be installed; you can install it as part of the first command with `--install-supervisor`.

## 1. Add a daemon

```bash
sudo abstrax daemon add queue-worker \
  --command="php8.5 artisan queue:work" \
  --directory=/var/www/myapp \
  --user=www-data \
  --processes=2 \
  --autostart \
  --autorestart=unexpected
```

If Supervisor is not installed yet:

```bash
sudo abstrax daemon add queue-worker \
  --command="php8.5 artisan queue:work" \
  --install-supervisor
```

Expected output:

```text
Daemon queue-worker added.
  Config: /etc/supervisor/conf.d/abstrax-queue-worker.conf
```

The `--command` flag is required. If you leave it out, the command stops with `--command is required`.

## 2. Check the status

```bash
abstrax daemon status queue-worker
```

```text
  Name:        queue-worker
  Status:      RUNNING
  Processes:   2/2
  Config:      /etc/supervisor/conf.d/abstrax-queue-worker.conf
    queue-worker:queue-worker_00  pid 12345, uptime 0:10:00
    queue-worker:queue-worker_01  pid 12346, uptime 0:09:58
```

`Status` is the worst process state. `Processes` is running/total.

Show the stored settings as well as live status:

```bash
abstrax daemon info queue-worker
```

List Abstrax-managed daemons. Add `--all` to include other Supervisor programs:

```bash
abstrax daemon list
abstrax daemon list --all
```

## 3. View the logs

```bash
abstrax daemon logs queue-worker
abstrax daemon logs queue-worker --lines=100 --follow
```

`--lines` returns that many lines from the log file. Use `--stderr` or `--stdout` to choose which log to read. When the daemon runs more than one process, `--process` selects one of them. The first process is `0`.

## 4. Control the process

These commands apply to every process in the daemon:

```bash
sudo abstrax daemon restart queue-worker
sudo abstrax daemon stop queue-worker
sudo abstrax daemon start queue-worker
```

## 5. Modify the daemon

Only the flags you pass are written. Omitted settings stay as they are in the Supervisor config. Modify accepts the same settings as `daemon add`, including restart policy and log paths. `--environment` merges by key. After the file is written, Supervisor restarts the program when the config changed.

```bash
sudo abstrax daemon modify queue-worker --processes=4
sudo abstrax daemon modify queue-worker --autorestart=always
sudo abstrax daemon modify queue-worker --unset-environment=DEBUG
```

## 6. Remove the daemon

By default the process is stopped before removal:

```bash
sudo abstrax daemon remove queue-worker --stop
```

To also delete its log files:

```bash
sudo abstrax daemon remove queue-worker --stop --delete-logs
```

## Useful options on `daemon add`

| Flag | Default | Purpose |
|---|---|---|
| `--processes` | `1` | Run multiple copies of the process |
| `--autostart` | `true` | Start when Supervisor starts |
| `--autorestart` | `unexpected` | `always`, `unexpected`, or `false` |
| `--startsecs` | `1` | Seconds before the process counts as started |
| `--startretries` | `3` | Start attempts before giving up |
| `--stopsignal` | `TERM` | Signal used to stop the process |
| `--stopwaitsecs` | `10` | Seconds to wait for a clean stop |
| `--environment` | | `KEY=VALUE` variables (repeatable) |

## Related

- [Daemons command reference](/docs/commands/daemons)
- [Managing cron jobs](/docs/guides/managing-cron-jobs)
- [Services](/docs/commands/service)
