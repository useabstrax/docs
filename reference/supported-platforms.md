# Supported platforms

Abstrax targets Debian and Ubuntu based Linux systems. This page describes what is supported today, how detection works, and what is not yet implemented.

## Architecture

Release builds are produced for:

- `linux/amd64`
- `linux/arm64`

Check your architecture with `uname -m`.

## Operating systems

Abstrax reads `/etc/os-release` and treats these `ID` values as supported:

- `ubuntu`
- `debian`
- `linuxmint`
- `pop`
- `raspbian`

On any other distribution, Abstrax reports the platform as not fully supported, with a message such as:

```text
OS "fedora" is not fully supported; Abstrax targets Debian and Ubuntu based systems
```

If the OS cannot be detected at all, it reports `could not detect OS`.

Detection of the surrounding tools also matters:

- **Package manager**: the `package` commands use `apt`. Abstrax can detect `apt`, `dnf`, `yum`, `apk`, and `pacman`, but the package commands are implemented for `apt`.
- **Service manager**: service operations assume `systemd`. Abstrax detects `systemd` and `sysvinit`.
- **Firewall backend**: the `firewall` commands use UFW. Abstrax detects `ufw`, `firewalld`, and `iptables`.

Run `abstrax doctor` to see what was detected on your server.

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
| git | `project` (git deploy) |

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

The code and README describe a planned hosted agent that would connect outbound to a hosted API, fetch structured jobs, run them locally through the same action layer as the CLI, and report results — without requiring inbound SSH. This is not built yet. A systemd unit file for the agent is installed by the packages at `/etc/systemd/system/abstrax-agent.service`, but it is intentionally left disabled.

Do not rely on agent functionality; it does nothing today.

## Related

- [Installation](getting-started/installation.html)
- [Troubleshooting](reference/troubleshooting.html)
