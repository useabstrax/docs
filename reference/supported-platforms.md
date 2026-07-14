# Supported platforms

Abstrax supports Debian/Ubuntu-based and RHEL-compatible Linux distributions. This page describes what is supported today, how detection works, and important family differences.

## Functional parity

Abstrax aims for **functional command parity** across supported distro families.

The same Abstrax commands should work on supported Debian/Ubuntu and Rocky/Alma systems where technically possible. Under the hood, Abstrax may use different packages, services, paths, and repositories per distro family. Those differences are handled by the platform provider layer — not by leaving commands unsupported.

A command should only remain unsupported on a supported distro family when there is a genuine technical limitation, an unacceptable safety risk, or a deliberate product decision documented here and in the code.

## Architecture

Release builds are produced for:

- `linux/amd64`
- `linux/arm64`

Check your architecture with `uname -m`.

## Operating systems

Abstrax reads `/etc/os-release` and builds a platform profile from the distribution `ID`, `ID_LIKE`, `VERSION_ID`, and related fields.

### Fully supported

#### Debian/Ubuntu family

These distributions are officially tested and receive `official` support level:

- Ubuntu 20.04+
- Debian 11+
- Linux Mint
- Pop!_OS
- Raspbian / Raspberry Pi OS

Recognised `ID` values include `ubuntu`, `debian`, `linuxmint`, `pop`, `raspbian`, and `raspberrypi`.

#### RHEL-compatible family

- Rocky Linux 9+
- AlmaLinux 9+

Recognised `ID` values include `rocky` and `almalinux`.

### Experimental or compatible

#### Debian/Ubuntu family

Other Debian/Ubuntu-based distributions may work but are not officially tested. Examples include:

- Ubuntu releases older than 20.04
- Debian releases older than 11
- Debian/Ubuntu derivatives detected via `ID_LIKE` but with an unlisted `ID`

These receive `compatible` support level. Abstrax allows mutating commands on compatible platforms, but behaviour is best-effort.

#### RHEL-compatible family

These receive `compatible` support level and are treated as experimental:

- Red Hat Enterprise Linux 9+
- CentOS Stream 9+
- Oracle Linux 9+

Recognised `ID` values include `rhel`, `centos`, `ol`, and `oracle`.

### Unsupported

Distributions outside the Debian and RHEL families are not supported and receive `unsupported` support level. Mutating commands exit cleanly before making system changes.

RHEL-family major versions older than 9 (for example Rocky Linux 8) are also unsupported. The supported RHEL-family targets are Rocky Linux 9+ and AlmaLinux 9+.

If the OS cannot be detected at all, Abstrax reports `could not detect operating system from /etc/os-release`.

Run `abstrax doctor` to see the detected profile on your server, including:

- distro ID, name, and version
- distro family
- package manager and service manager
- nginx layout and config directory
- web user/group and default project root
- PHP-FPM naming strategy
- firewall strategy
- SELinux status (RHEL-family)
- support level

## Platform conventions

### Debian family

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

### RHEL-compatible family

| Area | Convention |
|---|---|
| Package manager | `dnf` |
| Service manager | `systemd` |
| Nginx layout | `conf.d` under `/etc/nginx/conf.d` |
| Nginx config directory | `/etc/nginx/conf.d` |
| Web user / group | `nginx` / `nginx` |
| Default project root | `/var/www` |
| Multi-version PHP | Remi Software Collections (`php83-php-fpm`, paths under `/opt/remi` and `/etc/opt/remi`) |
| Firewall | firewalld |
| SELinux | Detected and warned about; never disabled automatically |

Important RHEL-family differences:

- Package commands use `dnf` rather than `apt`.
- Nginx site configs are written to `/etc/nginx/conf.d/{site}.conf`. There is no Debian-style `sites-available` / `sites-enabled` symlink layout. Disabling a site renames the file to `{site}.conf.disabled`.
- The default web user/group is usually `nginx`, not `www-data`.
- Multi-version PHP uses Remi SCL packages (for example `php83-php-fpm`). Services, binaries, sockets, and pool configs are version-specific under Remi paths. Abstrax resolves these through the provider layer.
- Remi (and often EPEL/CRB) must be enabled before PHP install, and before Redis install on EL10+. Use `abstrax repo enable remi --enable-required-repos` or pass `--enable-required-repos`. Abstrax will not silently enable Remi.
- On Rocky/Alma 10+, AppStream ships Valkey instead of Redis. `abstrax cache install redis` installs Redis from Remi’s `redis:remi-7.2` module stream (not Valkey) so the `redis` package and service names stay consistent.
- Firewall commands use firewalld (`firewall-cmd`) where available, with permanent rules and reload after changes.
- firewalld does not use UFW-style numbered rules. `abstrax firewall rule list` shows Abstrax-assigned IDs that map to services/ports; `abstrax firewall rule remove <id>` removes the matching entry. You can also use `abstrax firewall remove service http` or `abstrax firewall remove port 8080/tcp`.
- Database install uses MariaDB (`mariadb-server`) as the MySQL-compatible server. The CLI command remains `mysql` for compatibility.
- Certbot may install EPEL automatically on Rocky/Alma/CentOS Stream. On RHEL/Oracle Linux, enable EPEL explicitly (`abstrax repo enable epel --enable-required-repos` or pass `--enable-required-repos`).
- Supervisor uses the `supervisord` service and `/etc/supervisord.d` with `.ini` program configs.
- Node.js auto-install uses the NodeSource RPM setup script (`https://rpm.nodesource.com/setup_XX.x`).
- Exact Ruby version pinning is a deliberate product limitation on RHEL-family systems today: Abstrax installs distro `ruby` + `ruby-devel`. Cross-distro exact pinning (for example via rbenv/asdf) is not implemented yet. Debian-family continues to use versioned apt packages where available.
- SELinux may require additional manual context rules for web project paths, nginx config, or PHP-FPM. Abstrax detects and warns only; it will not run `setenforce 0` or rewrite SELinux policy.

## Repositories (RHEL-family)

Some features require extra repositories:

| Repository | Used for | Rocky/Alma | RHEL/Oracle |
|---|---|---|---|
| EPEL | Certbot; Remi dependency | May enable automatically when required | Requires `abstrax repo enable epel --enable-required-repos` or `--enable-required-repos` |
| CRB | Remi dependency on EL9 | Enabled as part of Remi setup when consented | Same, with explicit consent |
| Remi | Multi-version PHP; Redis on EL10+ (AppStream ships Valkey) | Requires `--enable-required-repos` or `abstrax repo enable remi --enable-required-repos` | Same |

```bash
sudo abstrax repo enable epel --enable-required-repos
sudo abstrax repo enable remi --enable-required-repos
sudo abstrax project add app --php --php-version=8.3 --enable-required-repos
```

Abstrax will not silently enable policy-sensitive third-party repositories on enterprise distributions without explicit user action.

## Feature status by family

| Feature | Debian family | RHEL family |
|---|---|---|
| Platform detection / `doctor` | Implemented | Implemented |
| User management | Implemented | Implemented |
| SSH key management | Implemented | Implemented |
| SSH server config | Implemented | Implemented |
| Package management | apt | dnf |
| Service management (systemd) | Implemented | Implemented |
| Cron management | Implemented | Implemented |
| Daemon management (Supervisor) | Implemented | Implemented (`supervisord`, `/etc/supervisord.d`, `.ini`) |
| Project management (nginx) | Implemented | Implemented (`conf.d` layout) |
| Web server management (nginx) | Implemented | Implemented |
| PHP multi-version install | Implemented (apt `php{version}-fpm`) | Implemented (Remi SCL; requires Remi/EPEL with explicit consent) |
| Node.js runtime auto-install | Implemented (deb.nodesource) | Implemented (rpm.nodesource) |
| Ruby runtime auto-install | Implemented (versioned apt packages) | Implemented (stock `ruby` + `ruby-devel`; exact pinning is a deliberate product limitation) |
| SSL (Certbot) install | Implemented | Implemented (EPEL on Rocky/Alma/CentOS; explicit on RHEL/Oracle) |
| MySQL / MariaDB install | Implemented (`mysql-server`) | Implemented (`mariadb-server` as MySQL-compatible server) |
| Cache (Redis, Memcached) | Implemented | Implemented (`redis` package/service; Rocky/Alma 10+ installs Redis via Remi with `--enable-required-repos`) |
| Firewall | UFW (numbered rule delete) | firewalld (list IDs + service/port remove) |
| Repository helpers | Not required | `abstrax repo enable` + `--enable-required-repos` |
| SELinux warnings | Detected where present | Detected and warned |
| Apache support | Not implemented | Not implemented |
| Hosted agent | Not implemented | Not implemented |

## Remaining non-parity areas

| Area | Status | Why |
|---|---|---|
| Exact Ruby version pinning on RHEL | Deliberate product limitation | Distro packages do not provide Debian-style exact pinning; a shared runtime manager (rbenv/asdf) is a larger feature and is not implemented yet |
| Rich-rule / complex firewalld deny policies | Temporary implementation gap | Simple service/port allow and remove are supported; complex rich-rule editing is not fully abstracted |
| Apache | Not implemented on any family | Product decision |
| Hosted agent | Not implemented on any family | Product decision |

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
| redis (`redis-server` / `redis`) | `cache` |
| memcached | `cache` |
| ufw | `firewall` (Debian family) |
| firewalld (`firewall-cmd`) | `firewall` (RHEL family) |
| curl | general |
| git | general |

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
- [Firewall](/docs/commands/firewall)
- [Repositories](/docs/commands/repo)
