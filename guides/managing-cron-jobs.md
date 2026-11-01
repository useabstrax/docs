# Managing cron jobs

This guide schedules a recurring task, then lists, inspects, and modifies it. Abstrax stores each job as a file in `/etc/cron.d/abstrax-<id>`, so its jobs stay separate from any hand-written crontabs.

Adding, modifying, and removing jobs requires root.

## 1. Add a job

Every job needs an ID, a `--command`, and a schedule. The schedule can be a frequency flag or a custom expression.

Using a frequency flag:

```bash
sudo abstrax cron add backup --command="/usr/local/bin/backup.sh" --daily --user=root
```

Using a custom 5-field cron expression, or a macro such as `@daily` or `@reboot`:

```bash
sudo abstrax cron add report --command="php8.5 artisan report" --schedule="0 8 * * 1" --user=www-data
sudo abstrax cron add reboot-check --command="/usr/local/bin/check.sh" --schedule="@reboot"
```

Each field of a 5-field expression is validated. `@reboot`, `@hourly`, `@daily`, `@weekly`, `@monthly`, `@yearly`, `@annually`, and `@midnight` are accepted.

Expected output:

```text
Cron job backup created.
  Schedule: 0 0 * * *
  User:     root
  Command:  /usr/local/bin/backup.sh
  File:     /etc/cron.d/abstrax-backup
```

If you leave out `--command`, or provide neither a frequency flag nor `--schedule`, the command stops with an error explaining what is missing.

## 2. Capture output and set environment

Redirect output to a log file and pass environment variables:

```bash
sudo abstrax cron add cleanup \
  --command="/usr/local/bin/cleanup.sh" \
  --daily \
  --output=/var/log/cleanup.log \
  --env=APP_ENV=production
```

To discard all output instead:

```bash
sudo abstrax cron add noisy --command="/usr/local/bin/noisy.sh" --hourly --discard-output
```

## 3. List and inspect jobs

```bash
abstrax cron list
```

```text
ID       SCHEDULE       USER       ENABLED   COMMAND
backup   0 0 * * *      root       yes       /usr/local/bin/backup.sh
report   0 8 * * 1      www-data   yes       php8.5 artisan report
```

```bash
abstrax cron info backup
```

## 4. Modify a job

Only the fields you pass are changed. Working directory and output redirects stay separate from the command, so a modify does not nest `cd` or redirects inside it.

```bash
sudo abstrax cron modify backup --schedule="0 3 * * *"
sudo abstrax cron modify backup --append-output --output=/var/log/backup.log
sudo abstrax cron modify backup --discard-output
sudo abstrax cron modify backup --no-output
```

`cron info` prints the working directory and output paths when they are set. Environment values may contain spaces and are stored as you entered them.

## 5. Enable or disable a job

Disable a job without removing it, then enable it again later:

```bash
abstrax cron disable backup
abstrax cron enable backup
```

## Run a job now

`cron run` starts a managed job immediately as its user, with that job's environment and working directory. It requires root. Output is shown in the terminal. Pass `--use-job-output` to write it the way the cron file would. `--timeout` defaults to 300 seconds; the command exits 124 if the job exceeds it. A disabled job still runs, and the result says so.

```bash
sudo abstrax cron run backup
sudo abstrax cron run backup --timeout 60
```

## 6. Remove a job

```bash
sudo abstrax cron remove backup
```

## Frequency flags

| Flag | Runs |
|---|---|
| `--every-minute` | Every minute |
| `--every-five-minutes` | Every 5 minutes |
| `--every-ten-minutes` | Every 10 minutes |
| `--every-fifteen-minutes` | Every 15 minutes |
| `--every-thirty-minutes` | Every 30 minutes |
| `--hourly` | Top of every hour |
| `--daily` | Midnight every day |
| `--weekly` | Midnight on Sunday |
| `--monthly` | Midnight on the 1st |
| `--yearly` | Midnight on 1 January |

A custom `--schedule` takes priority over a frequency flag. It must be a validated 5-field expression or a macro such as `@daily` or `@reboot`.

## Cron versus daemons

Cron jobs run on a schedule and then exit. For a process that should run continuously (such as a queue worker), use a [daemon](/docs/guides/managing-daemons) instead.

## Related

- [Cron command reference](/docs/commands/cron)
- [Managing daemons](/docs/guides/managing-daemons)
