# Supported platforms

Abstrax currently supports Debian/Ubuntu-based Linux distributions. This page describes what is supported today, how detection works, and what is not yet implemented.

## Architecture

Release builds are produced for:

- `linux/amd64`
- `linux/arm64`

Check your architecture with `uname -m`.

## Operating systems

Abstrax reads `/etc/os-release` and builds a platform profile from the distribution `ID`, `ID_LIKE`, `VERSION_ID`, and related fields.

### Fully supported

These distributions are officially tested and receive `official` support level:

- Ubuntu 20.04+
- Debian 11+
- Linux Mint
- Pop!_OS
- Raspbian / Raspberry Pi OS

Recognised `ID` values include `ubuntu`, `debian`, `linuxmint`, `pop`, `raspbian`, and `raspberrypi`.

### Compatible (best-effort)

Other Debian/Ubuntu-based distributions may work but are not officially tested. Examples include:

- Ubuntu releases older than 20.04
- Debian releases older than 11
- Debian/Ubuntu derivatives detected via `ID_LIKE` but with an unlisted `ID`

These receive `compatible` support level. Abstrax allows mutating commands on compatible platforms, but behaviour is best-effort.

### Unsupported

Non-Debian-family distributions are not currently supported and receive `unsupported` support level. Mutating commands exit cleanly before making system changes.

If the OS cannot be detected at all, Abstrax reports `could not detect operating system from /etc/os-release`.

Run `abstrax doctor` to see the detected profile on your server, including:

- distro ID, name, and version
- distro family
- package manager and service manager
- nginx layout
- web user and default project root
- PHP-FPM naming strategy
- firewall strategy
- support level

## Platform conventions (Debian family)

On supported Debian/Ubuntu-based systems, Abstrax assumes:

| Area | Convention |
|---|---|
| Package manager | `apt` |
| Service manager | `systemd` (with sysvinit fallback where detected) |
| Nginx layout | `sites-available` / `sites-enabled` under `/etc/nginx` |
| Web user / group | `www-data` |
| Default project root | `/var/www` |
| PHP-FPM services | `php{version}-fpm` (for example `php8.5-fpm`) |
| PHP-FPM sockets | `/run/php/php{version}-fpm.sock` |
| Firewall | UFW where installed |

Abstrax can detect other package managers (`dnf`, `yum`, `apk`, `pacman`) and firewall backends (`firewalld`, `iptables`), but the implemented commands use the Debian-family conventions above.

## Tools Abstrax manages

Abstrax calls other tools rather than replacing them. The relevant feature only works if the tool is installed. `abstrax doctor` reports the presence of:

| Tool | Used by |
|---|---|
| nginx | `project`, `web` |
| apache2 / httpd | detected only; Apache support is not implemented |
| certbot | `ssl` |
| mysql | `mysql` |
| mariadb | `mysql` |
| supervisor (`supervisorctl`) | `daemon` |
| redis (`redis-server`) | `cache` |
| memcached | `cache` |
| ufw | `firewall` |
| curl | general |
| git | general |

## Feature status

| Feature | Status |
|---|---|
| User management | Implemented |
| SSH key management | Implemented |
| SSH server config | Implemented |
| Package management (apt) | Implemented |
| Service management (systemd) | Implemented |
| Cron management | Implemented |
| Daemon management (Supervisor) | Implemented |
| Project management (nginx) | Implemented |
| Web server management (nginx) | Implemented |
| SSL (Certbot) | Implemented |
| MySQL / MariaDB | Implemented |
| Cache (Redis, Memcached) | Implemented |
| Firewall (UFW) | Implemented |
| Server status | Implemented |
| Apache support | Not implemented (flags exist but are not wired up) |
| Hosted agent | Not implemented |

## Apache

The `project` commands accept an `--apache` flag and the platform detection reports Apache presence, but Apache support is not implemented. Use nginx, which is the default.

## Future agent

The `agent` command and its subcommands (`connect`, `status`, `run`, `update`) are placeholders. Running any of them prints:

```text
Agent mode is not yet implemented.
```

The code and README describe a planned hosted agent that would connect outbound to a hosted API, fetch structured jobs, run them locally through the same action layer as the CLI, and report results - without requiring inbound SSH. This is not built yet. A systemd unit file for the agent is installed by the packages at `/etc/systemd/system/abstrax-agent.service`, but it is intentionally left disabled.

Do not rely on agent functionality; it does nothing today.

## Related

- [Installation](/docs/getting-started/installation)
- [Troubleshooting](/docs/reference/troubleshooting)
