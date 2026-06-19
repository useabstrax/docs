# Permissions

Most Abstrax commands change system state and require root. This page explains which commands need root, which do not, and how Abstrax enforces this.

## How the root check works

Commands that change system state call an internal root check before doing any work. If the process is not running as root (UID 0), the command stops with a clear error:

```text
this command requires root privileges; please run with sudo
```

Run these commands with `sudo`:

```bash
sudo abstrax user add deploy --grant-sudo
```

## Commands that require root

The following change system state and require root:

- **user**: `add`, `remove`, `grant-sudo`, `revoke-sudo`, `set-groups`, `add-groups`, `remove-groups`, `set-shell`, `lock`, `unlock`
- **self**: `update`
- **ssh**: `config set-port`, `config set-timeout`, `config disable-root-login`, `config enable-root-login`, `config disable-password-auth`, `config enable-password-auth`, `reload`, `restart`
- **package**: `install`, `remove`, `update`, `upgrade`
- **service**: `start`, `stop`, `restart`, `reload`, `enable`, `disable`
- **cron**: `add`, `remove`, `modify`
- **daemon**: `add`, `remove`, `modify`, `start`, `stop`, `restart`
- **project**: `add`, `remove`, `modify`, `enable`, `disable`, `reload`
- **web**: `reload`, `restart`
- **ssl**: `add`, `remove`, `renew`
- **mysql**: `config set`, `install`, `reset-root-password`
- **cache**: `install`, `remove`, `start`, `stop`, `restart`
- **firewall**: `enable`, `disable`, `allow`, `deny`, `allow-ip`, `deny-ip`, `rule remove`

## Commands that do not enforce a root check

These read-only or query commands do not call the root check in code:

- `doctor`, `version`, `log`
- `self update --dry-run` (preview only; the actual update requires root)
- `user info`, `user list`
- `ssh config show`
- `package search`, `package info`, `package list`
- `service status`
- `cron list`, `cron info`, `cron enable`, `cron disable`
- `daemon status`, `daemon list`, `daemon logs`
- `project list`, `project info`
- `web test`
- `ssl status`
- All `server` commands (`status`, `cpu`, `memory`, `disk`, `load`, `services`)
- `cache status`, `cache config`
- `firewall status`, `firewall rule list`
- All `ssh-key` commands (`add`, `remove`, `list`, `info`)
- Most `mysql` commands except `config set`, `install`, and `reset-root-password`

A note on two groups:

- **ssh-key**: the commands do not enforce root, but editing another user's `~/.ssh/authorized_keys` requires file permissions you usually only have as root or as that user. Use `sudo` when managing another user's keys.
- **mysql**: most commands do not enforce a root check because they authenticate to the database using the saved connection config rather than relying on OS privileges. However, reading the config file at `/etc/abstrax/mysql.json` (mode 0600, owned by root) generally requires root.
- **cron enable/disable/list/info**: these do not enforce root, but reading or writing files in `/etc/cron.d` may still require root depending on file permissions.

## Why elevated permissions are needed

The operations Abstrax performs are the same ones you would otherwise run by hand as root: creating users, editing `/etc/ssh`, installing packages, controlling systemd, writing to `/etc/cron.d`, managing the firewall, and so on. These actions modify protected system files and services, which is why root is required.

## Reducing risk

- Use `--dry-run` to preview a command before running it for real.
- Use `--verbose` to see the exact underlying command Abstrax will run.
- Only run Abstrax as root for the commands that need it. Read-only checks like `doctor` and `server status` can run as a normal user.

See [Security](/docs/reference/security) for the safeguards Abstrax includes and recommendations for safe use.

## Related

- [Config file](/docs/configuration/config-file)
- [Security](/docs/reference/security)
- [Troubleshooting](/docs/reference/troubleshooting)
