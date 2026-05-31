# Cron

The `cron` command group manages scheduled jobs. Abstrax writes each job as a file in `/etc/cron.d`, named `abstrax-<id>`. This keeps Abstrax-managed jobs separate from hand-written crontabs.

```text
abstrax cron <action> [arguments] [flags]
```

## Permissions

`add`, `remove`, and `modify` require root because they write to `/etc/cron.d`. The `list`, `info`, `enable`, and `disable` commands do not enforce a root check in code, but writing to or reading the cron files may still require root depending on file permissions.

## Cron job IDs

A cron job is identified by an ID you choose. The ID may contain letters, digits, underscores, and hyphens, up to 64 characters.

## `cron add`

Add a new cron job. You must provide a command and a schedule.

```bash
abstrax cron add <id> --command="<command>" (--schedule="..." | frequency flag) [flags]
```

### Required

- `--command` – the command to run. If omitted, the command fails with `--command is required`.
- A schedule, provided either with `--schedule` or one of the frequency flags. If neither is given, the command fails with `--schedule or a frequency flag is required`.

### Schedule flags

Provide a raw cron expression, or use a convenience flag:

| Flag | Equivalent expression |
|---|---|
| `--every-minute` | `* * * * *` |
| `--every-five-minutes` | `*/5 * * * *` |
| `--every-ten-minutes` | `*/10 * * * *` |
| `--every-fifteen-minutes` | `*/15 * * * *` |
| `--every-thirty-minutes` | `*/30 * * * *` |
| `--hourly` | `0 * * * *` |
| `--daily` | `0 0 * * *` |
| `--weekly` | `0 0 * * 0` |
| `--monthly` | `0 0 1 * *` |
| `--yearly` | `0 0 1 1 *` |
| `--schedule` | A custom 5-field cron expression |

A `--schedule` value takes priority. It must have exactly 5 fields (minute, hour, day, month, weekday) or it is rejected.

### Other flags

| Flag | Default | Description |
|---|---|---|
| `--user` | `root` | User to run the job as |
| `--output` | | Redirect stdout to this file |
| `--error-output` | | Redirect stderr to this file |
| `--append-output` | `false` | Append output instead of overwriting |
| `--discard-output` | `false` | Discard all output |
| `--working-dir` | | Working directory for the command |
| `--env` | | Environment variable as `KEY=VALUE` (repeatable) |
| `--enabled` | `true` | Create the job enabled |
| `--disabled` | `false` | Create the job disabled |

### Examples

```bash
sudo abstrax cron add backup --command="/usr/local/bin/backup.sh" --daily --user=root
sudo abstrax cron add report --command="php artisan report" --schedule="0 8 * * 1" --user=www-data
sudo abstrax cron add queue --command="php artisan queue:work" --every-minute
sudo abstrax cron add cleanup --command="/usr/local/bin/cleanup.sh" --daily \
  --output=/var/log/cleanup.log --env=APP_ENV=production
```

### Example output

```text
Cron job backup created.
  Schedule: 0 0 * * *
  User:     root
  Command:  /usr/local/bin/backup.sh
  File:     /etc/cron.d/abstrax-backup
```

## `cron remove`

Remove a managed cron job by ID.

```bash
sudo abstrax cron remove <id>
```

## `cron modify`

Change an existing job. Only the fields you pass are updated.

```bash
sudo abstrax cron modify <id> [flags]
```

| Flag | Description |
|---|---|
| `--user` | Change the user |
| `--command` | Change the command |
| `--schedule` | Change the schedule (validated as a 5-field expression) |
| `--output` | Change stdout redirection |
| `--error-output` | Change stderr redirection |
| `--working-dir` | Change the working directory |
| `--env` | Set an environment variable as `KEY=VALUE` (repeatable) |

```bash
sudo abstrax cron modify backup --schedule="0 3 * * *"
```

## `cron list`

List managed cron jobs.

```bash
abstrax cron list
```

```text
ID       SCHEDULE       USER       ENABLED   COMMAND
backup   0 0 * * *      root       yes       /usr/local/bin/backup.sh
queue    * * * * *      www-data   yes       php artisan queue:work
```

## `cron info`

Show details for a single job.

```bash
abstrax cron info backup
```

```text
  ID:        backup
  Schedule:  0 0 * * *
  User:      root
  Command:   /usr/local/bin/backup.sh
  Enabled:   yes
  File:      /etc/cron.d/abstrax-backup
```

## `cron enable` and `cron disable`

Enable or disable a job without removing it.

```bash
abstrax cron enable backup
abstrax cron disable backup
```

## Notes

- Only jobs created by Abstrax (files named `abstrax-<id>` in `/etc/cron.d`) are listed and managed. Existing system crontabs are not touched.
- The `--schedule` value must be a standard 5-field cron expression. Abstrax checks the field count but does not deeply validate each field's syntax.

## Related

- [Managing cron jobs](../guides/managing-cron-jobs.md)
- [Daemons](daemons.md) – for long-running processes rather than scheduled jobs
- [Configuration](../configuration/index.md)
