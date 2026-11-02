# Cron

The `cron` command group manages scheduled jobs. Abstrax writes each job as a file in `/etc/cron.d`, named `abstrax-<id>`. This keeps Abstrax-managed jobs separate from hand-written crontabs.

```text
abstrax cron <action> [arguments] [flags]
```

## Permissions

`add`, `remove`, `modify`, and `run` require root. `add`, `remove`, and `modify` write to `/etc/cron.d`. `run` starts the job as its user. The `list`, `info`, `enable`, and `disable` commands do not enforce a root check in code, but writing to or reading the cron files may still require root depending on file permissions.

## Cron job IDs

A cron job is identified by an ID you choose. The ID may contain letters, digits, underscores, and hyphens, up to 64 characters.

## `cron add`

Add a new cron job. You must provide a command and a schedule.

```bash
abstrax cron add <id> --command="<command>" (--schedule="..." | frequency flag) [flags]
```

### Required

- `--command` - the command to run. If omitted, the command fails with `--command is required`.
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
| `--schedule` | A 5-field cron expression, or a macro such as `@daily` |

A `--schedule` value takes priority over a frequency flag. A 5-field expression is minute, hour, day, month, and weekday. Each field is checked: ranges, lists, steps, and month or weekday names are accepted, and values outside the field's range are rejected. These macros are also accepted: `@reboot`, `@hourly`, `@daily`, `@weekly`, `@monthly`, `@yearly`, `@annually`, and `@midnight`.

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

When the command runs PHP, use the versioned binary that matches the project (`php8.5`, `php8.4`, and so on). The unversioned `php` command follows the system default and may not match the project's runtime. Check the version with `abstrax project info <name>`.

```bash
sudo abstrax cron add backup --command="/usr/local/bin/backup.sh" --daily --user=root
sudo abstrax cron add report --command="php8.5 artisan report" --schedule="0 8 * * 1" --user=www-data
sudo abstrax cron add queue --command="php8.5 artisan queue:work" --every-minute
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
| `--schedule` | Change the schedule (a 5-field expression or a macro such as `@daily`) |
| `--output` | Change stdout redirection. An empty value clears it |
| `--error-output` | Change stderr redirection. An empty value clears it |
| `--append-output` | Append stdout instead of overwriting |
| `--discard-output` | Discard stdout and stderr |
| `--no-output` | Remove stdout and stderr redirects |
| `--working-dir` | Change the working directory. An empty value clears it |
| `--env` | Set an environment variable as `KEY=VALUE` (repeatable) |

Working directory and redirects are stored separately from the command, so a later modify does not nest `cd` or redirects inside the command string.

```bash
sudo abstrax cron modify backup --schedule="0 3 * * *"
sudo abstrax cron modify backup --append-output --output=/var/log/backup.log
sudo abstrax cron modify backup --schedule="@daily"
sudo abstrax cron modify backup --no-output
```

## `cron list`

List managed cron jobs.

```bash
abstrax cron list
```

```text
ID       SCHEDULE       USER       ENABLED   COMMAND
backup   0 0 * * *      root       yes       /usr/local/bin/backup.sh
queue    * * * * *      www-data   yes       php8.5 artisan queue:work
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
  Work dir:  /var/backups
  Output:    /var/log/backup.log (append)
  Enabled:   yes
  File:      /etc/cron.d/abstrax-backup
```

`Work dir`, `Output`, and `Errors` are printed only when those fields are set. Discarded output is shown as `Output: discarded`. With `--json`, the same details are `working_dir`, `output`, `error_output`, `append_output`, and `discard_output`. Environment values, including values that contain spaces, are returned in `env`.

## `cron run`

Run a managed job immediately, the way cron would, and report the result. This requires root.

```bash
sudo abstrax cron run <id> [--timeout 300] [--use-job-output]
sudo abstrax cron run backup
sudo abstrax cron run backup --timeout 60 --use-job-output
```

The job runs as its user. Abstrax uses `runuser -u <user>` when `runuser` is available, and `sudo -u <user>` otherwise. The environment is the job's variables plus a cron-like base, and nothing else from the caller:

| Variable | Value |
|---|---|
| `SHELL` | `/bin/sh` |
| `PATH` | `/usr/bin:/bin` |
| `HOME` | The user's home directory |

`HOME` comes from the local account when it can be looked up. Otherwise it is `/root` for `root` and `/home/<user>` for anyone else. A failed lookup uses that fallback and does not fail the command, including `--dry-run`. Job variables replace these when they use the same name. When the job has a working directory, the shell `cd`s there before the command, the same way the cron file does. The command itself is `/bin/sh -c` with the job command.

A disabled job still runs. The summary says the job is disabled, and JSON includes `"disabled": true`.

By default, stdout and stderr are streamed to the terminal and captured in the result. They are not written to the job's log file. `--use-job-output` applies the job's redirects the way the cron file would: append or overwrite, separate stdout and stderr, or discard.

### Flags

| Flag | Default | Description |
|---|---|---|
| `--timeout` | `300` | Seconds to allow the job. `0` means no limit |
| `--use-job-output` | `false` | Apply the job's output redirects instead of capturing output |

### Exit codes

| Code | Meaning |
|---|---|
| Job exit code | The command ran and returned that status |
| `124` | The timeout elapsed and the process group was killed, matching `timeout(1)` |
| `1` | Abstrax could not run the job, including an unknown id |

### JSON

`--json` prints one result object. `--json-stream` prints each output line as a progress event (`step` is `output`) and then a final result. The result `data` object has these fields:

| Field | Description |
|---|---|
| `id`, `user`, `command` | The job that ran |
| `disabled` | `true` when the job is disabled |
| `started_at`, `finished_at` | UTC timestamps |
| `duration_ms` | How long the run took |
| `exit_code` | The job's exit code. Omitted on `--dry-run` |
| `timed_out` | `true` when the timeout killed the process group |
| `output` | The last 64 KB of combined stdout and stderr. The tail is kept |
| `output_truncated` | `true` when older output was cut to stay within 64 KB |
| `dry_run` | `true` on `--dry-run` |

```json
{
  "id": "backup",
  "user": "deploy",
  "command": "php artisan backup:run",
  "disabled": false,
  "started_at": "2026-10-28T03:17:00Z",
  "finished_at": "2026-10-28T03:17:42Z",
  "duration_ms": 42118,
  "exit_code": 0,
  "timed_out": false,
  "output": "...",
  "output_truncated": false
}
```

`--dry-run` prints the command it would run and does not execute the job. The result has `dry_run: true` and no `exit_code`.

## `cron enable` and `cron disable`

Enable or disable a job without removing it.

```bash
abstrax cron enable backup
abstrax cron disable backup
```

## Notes

- Only jobs created by Abstrax (files named `abstrax-<id>` in `/etc/cron.d`) are listed and managed. Existing system crontabs are not touched.
- A `%` in the command is stored as `\%`, so cron treats it as a literal percent rather than as a stdin separator. `cron info` shows the original `%`.
- Working directory and output paths are shell-quoted in the cron file.
- `cron add` and `cron modify` reject line breaks, carriage returns, and null bytes in every value. They also reject a user name that is not a valid Linux user, and an environment name that is not a valid variable name.

## Related

- [Managing cron jobs](/docs/guides/managing-cron-jobs)
- [Daemons](/docs/commands/daemons) - for long-running processes rather than scheduled jobs
- [Configuration](/docs/configuration/index)
