# Daemons

The `daemon` command group manages long-running background processes using [Supervisor](http://supervisord.org/). Abstrax writes a Supervisor program configuration for each daemon using the platform provider:

- Debian/Ubuntu family: `/etc/supervisor/conf.d/abstrax-<name>.conf` (service `supervisor`)
- RHEL-compatible family: `/etc/supervisord.d/abstrax-<name>.ini` (service `supervisord`)

Use daemons for processes that should run continuously (queue workers, websocket servers, and similar). For scheduled tasks that run and exit, use [cron](/docs/commands/cron) instead.

```text
abstrax daemon <action> [arguments] [flags]
```

## Permissions

`add`, `remove`, `modify`, `start`, `stop`, and `restart` require root. The read-only commands `info`, `status`, `list`, and `logs` do not require root.

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

When the command runs PHP, use the versioned binary that matches the project (`php8.5`, `php8.4`, and so on). The unversioned `php` command follows the system default and may not match the project's runtime. Check the version with `abstrax project info <name>`.

```bash
sudo abstrax daemon add queue-worker \
  --command="php8.5 artisan queue:work" \
  --directory=/var/www/myapp \
  --user=www-data \
  --processes=2 \
  --autostart \
  --autorestart=unexpected

# Install Supervisor at the same time if it is missing
sudo abstrax daemon add queue-worker --command="php8.5 artisan queue:work" --install-supervisor
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

Change a daemon's configuration. Only the flags you pass are written. Omitted settings are read back from the existing Supervisor config.

`daemon modify` accepts the same settings as `daemon add`, including restart policy and log paths. After the file is written, Abstrax runs `supervisorctl update`, which restarts the program when its config changed.

```bash
sudo abstrax daemon modify <name> [flags]
```

| Flag | Description |
|---|---|
| `--command` | Change the command |
| `--directory` | Change the working directory |
| `--user` | Change the user |
| `--processes` | Change the number of processes |
| `--autostart` | Start automatically when Supervisor starts |
| `--autorestart` | `always`, `unexpected`, or `false` |
| `--startsecs` | Seconds before the process is considered started |
| `--startretries` | Number of start retries |
| `--stopsignal` | Signal used to stop the process |
| `--stopwaitsecs` | Seconds to wait for the process to stop |
| `--exitcodes` | Expected exit codes |
| `--stdout-logfile` | Path for the stdout log file |
| `--stderr-logfile` | Path for the stderr log file |
| `--environment` | Set an environment variable as `KEY=VALUE` (repeatable). Merges by key |
| `--unset-environment` | Remove one environment variable (repeatable) |
| `--clear-environment` | Remove every environment variable before applying `--environment` |

```bash
sudo abstrax daemon modify queue-worker --processes=4
sudo abstrax daemon modify queue-worker --autorestart=always --stdout-logfile=/var/log/queue.log
sudo abstrax daemon modify queue-worker --environment=APP_ENV=production --unset-environment=DEBUG
```

## Lifecycle commands

`start`, `stop`, and `restart` apply to every process in the daemon.

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
  Processes:   2/2
  Config:      /etc/supervisor/conf.d/abstrax-queue-worker.conf
    queue-worker:queue-worker_00  pid 12345, uptime 0:10:00
    queue-worker:queue-worker_01  pid 12346, uptime 0:09:58
```

`Status` is the worst process state. A running process that has a `FATAL` sibling is reported as `FATAL`. With `--json`, processes are grouped under one daemon. Each running process includes `pid`, `uptime`, and `uptime_seconds`.

## `daemon info`

Show the settings stored in the Supervisor config, plus live status. Does not require root. Registered for agents as the `daemon.info` action.

```bash
abstrax daemon info queue-worker
```

```text
  Name:        queue-worker
  Status:      RUNNING
  Command:     php8.5 artisan queue:work
  Directory:   /var/www/myapp
  User:        www-data
  Processes:   2
  Running:     2/2
  Autostart:   true
  Autorestart: unexpected
  Config:      /etc/supervisor/conf.d/abstrax-queue-worker.conf
```

`--json` also includes restart policy details, environment variables, and the stdout and stderr log paths.

## `daemon list`

List Abstrax-managed daemons. Does not require root. `--all` also includes other Supervisor programs. In JSON, those extra programs have `managed: false` and no `config_path`.

```bash
abstrax daemon list
abstrax daemon list --all
```

```text
NAME           STATUS     PROCESSES
queue-worker   RUNNING    2/2
```

`PROCESSES` is running/total. A fatal process is called out, for example `1/2  (1 FATAL)`. JSON uses the same grouping as `daemon status`: one object per daemon, with a `processes` array.

## `daemon logs`

Show a daemon's logs. Does not require root.

```bash
abstrax daemon logs <name> [flags]
```

| Flag | Default | Description |
|---|---|---|
| `--lines` | `50` | Number of lines to return from the log file |
| `--follow` | `false` | Follow the log output |
| `--stderr` | `false` | Show the stderr log |
| `--stdout` | `false` | Show the stdout log |
| `--process` | `0` | Process index when `numprocs` is greater than 1. The first process is `0` |

```bash
abstrax daemon logs queue-worker
abstrax daemon logs queue-worker --lines=100 --follow
abstrax daemon logs queue-worker --stderr --process=1
```

## Notes

- These commands require Supervisor. If it is not installed, use `--install-supervisor` with `daemon add`, or install it first. Run `abstrax doctor` to check whether Supervisor is present.
- `--autorestart` accepts `always`, `unexpected`, or `false`. The default on `daemon add` is `unexpected`, which restarts the process only when it exits with an unexpected code.
- `daemon add` and `daemon modify` reject line breaks, carriage returns, and null bytes. They also reject an invalid user, a relative path, an unknown stop signal, an invalid exit-code list, and an invalid environment name.
- A `%` in the command is stored as `%%`, so Supervisor treats it as a literal percent. Environment values are double-quoted. A value that contains `"` or `\` is rejected, because Supervisor parses `environment` with a non-POSIX reader that does not treat `\` as an escape.

## Related

- [Managing daemons](/docs/guides/managing-daemons)
- [Cron](/docs/commands/cron) - for scheduled jobs
- [Services](/docs/commands/service) - for systemd services
