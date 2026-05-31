# Environment variables

Abstrax does not define its own environment variables to configure its behaviour. There is no `ABSTRAX_*` environment variable that changes defaults, output, or paths.

This behaviour is based on the current application code. If you are looking for a way to change defaults, use the global flags instead.

## Global flags instead of environment variables

Abstrax behaviour is controlled by flags rather than the environment:

| Flag | Effect |
|---|---|
| `--json` | Output machine-readable JSON |
| `--dry-run` | Show what would happen without making changes |
| `--yes` | Skip confirmation prompts |
| `--quiet` | Reduce output |
| `--verbose` | Increase output, including the underlying commands run |
| `--no-color` | Disable coloured output |

You can apply these per command. There is no configuration file or environment variable to set them globally.

## Environment that does affect Abstrax indirectly

While Abstrax has no variables of its own, it runs as a normal process and the usual environment still applies:

- **`PATH`** – Abstrax finds the tools it manages (`apt`, `systemctl`, `ufw`, `nginx`, `certbot`, `supervisorctl`, `mysql`, and so on) by looking them up on the `PATH`. If a tool is installed in a non-standard location that is not on the `PATH`, Abstrax may report it as not found. The `abstrax doctor` command uses the same lookup.
- **The invoking user** – whether you are root determines which commands you can run. See [Permissions](permissions.md).

## Setting environment for managed jobs

Some commands let you set environment variables for the processes they create. These are not Abstrax's own environment; they are written into the job or daemon definition:

- `abstrax cron add ... --env=KEY=VALUE` sets variables for a cron job.
- `abstrax daemon add ... --environment=KEY=VALUE` sets variables for a Supervisor program.

Both flags can be repeated to set multiple variables.

```bash
sudo abstrax cron add report --command="php artisan report" --daily --env=APP_ENV=production
sudo abstrax daemon add worker --command="node worker.js" --environment=NODE_ENV=production
```

## Related

- [Config file](config-file.md)
- [Command reference](../reference/command-reference.md)
