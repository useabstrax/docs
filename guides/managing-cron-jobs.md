# Managing cron jobs

This guide schedules a recurring task, then lists, inspects, and modifies it. Abstrax stores each job as a file in `/etc/cron.d/abstrax-<id>`, so its jobs stay separate from any hand-written crontabs.

Adding, modifying, and removing jobs requires root.

## 1. Add a job

Every job needs an ID, a `--command`, and a schedule. The schedule can be a frequency flag or a custom expression.

Using a frequency flag:

```bash
sudo abstrax cron add backup --command="/usr/local/bin/backup.sh" --daily --user=root
```

Using a custom 5-field cron expression:

```bash
sudo abstrax cron add report --command="php artisan report" --schedule="0 8 * * 1" --user=www-data
```

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
report   0 8 * * 1      www-data   yes       php artisan report
```

```bash
abstrax cron info backup
```

## 4. Modify a job

Only the fields you pass are changed:

```bash
sudo abstrax cron modify backup --schedule="0 3 * * *"
```

## 5. Enable or disable a job

Disable a job without removing it, then enable it again later:

```bash
abstrax cron disable backup
abstrax cron enable backup
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

A custom `--schedule` takes priority over a frequency flag and must have exactly 5 fields.

## Cron versus daemons

Cron jobs run on a schedule and then exit. For a process that should run continuously (such as a queue worker), use a [daemon](guides/managing-daemons.html) instead.

## Related

- [Cron command reference](commands/cron.html)
- [Managing daemons](guides/managing-daemons.html)
