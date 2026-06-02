# Daemons

- [Users](/commands/users.html)
- [SSH keys](/commands/ssh-keys.html)
- [Packages](/commands/packages.html)
- [System](/commands/system.html)
- [Cron](/commands/cron.html)
- [Daemons](/commands/daemons.html)
- [Projects](/commands/projects.html)
- [Certificates](/commands/certificates.html)

The `daemon` command group manages long-running background processes using [Supervisor](http://supervisord.org/). Abstrax writes a Supervisor program configuration for each daemon to:

```text
/etc/supervisor/conf.d/abstrax-<name>.conf
```

Use daemons for processes that should run continuously (queue workers, websocket servers, and similar). For scheduled tasks that run and exit, use [cron](/commands/cron.html) instead.

```text
abstrax daemon <action> [arguments] [flags]
```

## Permissions

`add`, `remove`, `modify`, `start`, `stop`, and `restart` require root. The read-only commands `status`, `list`, and `logs` do not require root.

## Daemon names

A daemon name may contain letters, digits, underscores, and hyphens, up to 64 characters.

## `daemon add`

Add a new managed daemon. The `--command` flag is required.

```bash
abstrax daemon add <name> --command="<command>" [flags]
```

### Flags

| Flag | Default | Description |
|---|---|---|
| `--command` | | Command to run (required) |
| `--directory` | | Working directory |
| `--user` | | User to run as |
| `--processes` | `1` | Number of processes |
| `--autostart` | `true` | Start automatically when Supervisor starts |
| `--autorestart` | `unexpected` | Autorestart mode (`always`, `unexpected`, `false`) |
| `--install-supervisor` | `false` | Install Supervisor if it is not present |
| `--startsecs` | `1` | Seconds before the process is considered started |
| `--startretries` | `3` | Number of start retries |
| `--stopsignal` | `TERM` | Signal used to stop the process |
| `--stopwaitsecs` | `10` | Seconds to wait for the process to stop |
| `--exitcodes` | | Expected exit codes |
| `--stdout-logfile` | | Path for the stdout log file |
| `--stderr-logfile` | | Path for the stderr log file |
| `--environment` | | Environment variable as `KEY=VALUE` (repeatable) |

### Examples

```bash
sudo abstrax daemon add queue-worker \
  --command="php artisan queue:work" \
  --directory=/var/www/myapp \
  --user=www-data \
  --processes=2 \
  --autostart \
  --autorestart=unexpected

# Install Supervisor at the same time if it is missing
sudo abstrax daemon add queue-worker --command="php artisan queue:work" --install-supervisor
```

### Example output

```text
Daemon queue-worker added.
  Config: /etc/supervisor/conf.d/abstrax-queue-worker.conf
```

## `daemon remove`

Remove a daemon. By default the process is stopped first.

```bash
sudo abstrax daemon remove <name> [flags]
```

| Flag | Default | Description |
|---|---|---|
| `--stop` | `true` | Stop the daemon before removing it |
| `--delete-logs` | `false` | Delete the log files |
| `--force` | `false` | Force removal |

```bash
sudo abstrax daemon remove queue-worker --stop
```

## `daemon modify`

Change a daemon's configuration. Only the fields you pass are updated.

```bash
sudo abstrax daemon modify <name> [flags]
```

| Flag | Description |
|---|---|
| `--command` | Change the command |
| `--directory` | Change the working directory |
| `--user` | Change the user |
| `--processes` | Change the number of processes |
| `--environment` | Set an environment variable as `KEY=VALUE` (repeatable) |

```bash
sudo abstrax daemon modify queue-worker --processes=4
```

## Lifecycle commands

```bash
sudo abstrax daemon start <name>
sudo abstrax daemon stop <name>
sudo abstrax daemon restart <name>
```

## `daemon status`

Show the status of a daemon. Does not require root.

```bash
abstrax daemon status queue-worker
```

```text
  Name:        queue-worker
  Status:      RUNNING
  Info:        pid 12345, uptime 0:10:00
  Config:      /etc/supervisor/conf.d/abstrax-queue-worker.conf
```

## `daemon list`

List managed daemons. Does not require root.

```bash
abstrax daemon list
```

```text
NAME           STATUS    INFO
queue-worker   RUNNING   pid 12345, uptime 0:10:00
```

## `daemon logs`

Show a daemon's logs. Does not require root.

```bash
abstrax daemon logs <name> [flags]
```

| Flag | Default | Description |
|---|---|---|
| `--lines` | `50` | Number of lines to show |
| `--follow` | `false` | Follow the log output |
| `--stderr` | `false` | Show the stderr log |
| `--stdout` | `false` | Show the stdout log |

```bash
abstrax daemon logs queue-worker
abstrax daemon logs queue-worker --lines=100 --follow
```

## Notes

- These commands require Supervisor. If it is not installed, use `--install-supervisor` with `daemon add`, or install it first. Run `abstrax doctor` to check whether Supervisor is present.
- `--autorestart` accepts `always`, `unexpected`, or `false`. The default, `unexpected`, restarts the process only when it exits with an unexpected code.

## Related

- [Managing daemons](/guides/managing-daemons.html)
- [Cron](/commands/cron.html) – for scheduled jobs
- [Services](/commands/system.html#service-commands) – for systemd services
