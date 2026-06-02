# Managing daemons

This guide runs a long-lived process under Supervisor, then checks its status and logs. Use daemons for processes that should run continuously, such as queue workers or websocket servers. For scheduled tasks, use [cron](guides/managing-cron-jobs.html) instead.

Adding, modifying, removing, and controlling daemons requires root. Supervisor must be installed; you can install it as part of the first command with `--install-supervisor`.

## 1. Add a daemon

```bash
sudo abstrax daemon add queue-worker \
  --command="php artisan queue:work" \
  --directory=/var/www/myapp \
  --user=www-data \
  --processes=2 \
  --autostart \
  --autorestart=unexpected
```

If Supervisor is not installed yet:

```bash
sudo abstrax daemon add queue-worker \
  --command="php artisan queue:work" \
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
  Info:        pid 12345, uptime 0:10:00
  Config:      /etc/supervisor/conf.d/abstrax-queue-worker.conf
```

List all managed daemons:

```bash
abstrax daemon list
```

## 3. View the logs

```bash
abstrax daemon logs queue-worker
abstrax daemon logs queue-worker --lines=100 --follow
```

Use `--stderr` or `--stdout` to choose which log to read.

## 4. Control the process

```bash
sudo abstrax daemon restart queue-worker
sudo abstrax daemon stop queue-worker
sudo abstrax daemon start queue-worker
```

## 5. Modify the daemon

Change settings such as the number of processes; only the fields you pass are updated:

```bash
sudo abstrax daemon modify queue-worker --processes=4
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

- [Daemons command reference](commands/daemons.html)
- [Managing cron jobs](guides/managing-cron-jobs.html)
- [Services](commands/system.html#service-commands)
