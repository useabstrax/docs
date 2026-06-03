# Command reference

A single-page listing of every Abstrax command and its flags. Each command links to its detailed documentation. For global flags that apply to every command, see [Global flags](#global-flags).

## Global flags

| Flag | Description |
|---|---|
| `--json` | Output machine-readable JSON |
| `--dry-run` | Show what would happen without making changes |
| `--yes` | Skip confirmation prompts for destructive commands |
| `--quiet` | Reduce output |
| `--verbose` | Increase output, including the underlying commands run |
| `--no-color` | Disable coloured output |
| `--help`, `-h` | Show help for a command |
| `--version`, `-v` | Show the version (root command only) |

## Top-level commands

| Command | Root | Description | Docs |
|---|---|---|---|
| `abstrax version` | No | Print version, commit, build date | [commands](/docs/commands/index#version) |
| `abstrax doctor` | No | Inspect the system | [commands](/docs/commands/index#doctor) |
| `abstrax log` | No | View the Abstrax log file | [commands](/docs/commands/index#log) |
| `abstrax agent ...` | No | Placeholder, not implemented | [platforms](/docs/reference/supported-platforms#future-agent) |
| `abstrax help` | No | Show help for any command | [commands](/docs/commands/index#top-level-commands) |
| `abstrax completion <shell>` | No | Generate a shell autocompletion script | [commands](/docs/commands/index#top-level-commands) |

`abstrax log` flags: `--lines` (default 50), `--follow`/`-f`. The `help` and `completion` commands are provided automatically by the command framework.

## user

See [Users](/docs/commands/users).

| Command | Root | Key flags |
|---|---|---|
| `user add <name>` | Yes | `--create-home`, `--no-create-home`, `--grant-sudo`, `--groups`, `--shell`, `--uid`, `--system`, `--password`, `--disabled-password`, `--comment` |
| `user remove <name>` | Yes | `--delete-home`, `--keep-home`, `--remove-cron`, `--kill-processes`, `--force` |
| `user grant-sudo <name>` | Yes | |
| `user revoke-sudo <name>` | Yes | |
| `user set-groups <name> <groups>` | Yes | |
| `user add-groups <name> <groups>` | Yes | |
| `user remove-groups <name> <groups>` | Yes | |
| `user set-shell <name> <shell>` | Yes | |
| `user lock <name>` | Yes | |
| `user unlock <name>` | Yes | |
| `user info <name>` | No | |
| `user list` | No | `--system`, `--regular`, `--sudo` |

## ssh-key

See [SSH keys](/docs/commands/ssh-keys#ssh-key-management).

| Command | Root | Key flags |
|---|---|---|
| `ssh-key add <user> <key>` | No* | `--name`, `--comment`, `--from-file`, `--force` |
| `ssh-key remove <user> <key-id>` | No* | `--fingerprint`, `--force` |
| `ssh-key list <user>` | No* | `--managed-only` |
| `ssh-key info <user> <key-id>` | No* | |

\* No root check in code, but managing another user's keys usually needs root.

## ssh

See [SSH configuration](/docs/commands/ssh-keys#ssh-server-configuration).

| Command | Root | Key flags |
|---|---|---|
| `ssh config show` | No | |
| `ssh config set-port <port>` | Yes | `--allow-firewall` |
| `ssh config set-timeout <seconds>` | Yes | |
| `ssh config disable-root-login` | Yes | |
| `ssh config enable-root-login` | Yes | |
| `ssh config disable-password-auth` | Yes | |
| `ssh config enable-password-auth` | Yes | |
| `ssh reload` | Yes | |
| `ssh restart` | Yes | |

## package

See [Packages](/docs/commands/packages).

| Command | Root | Key flags |
|---|---|---|
| `package install <name>` | Yes | `--version` |
| `package remove <name>` | Yes | `--purge` |
| `package update` | Yes | |
| `package upgrade` | Yes | `--security-only` |
| `package search <query>` | No | |
| `package info <name>` | No | |
| `package list` | No | |

## service

See [Service commands](/docs/commands/system#service-commands).

| Command | Root | Description |
|---|---|---|
| `service start <name>` | Yes | Start a service |
| `service stop <name>` | Yes | Stop a service |
| `service restart <name>` | Yes | Restart a service |
| `service reload <name>` | Yes | Reload a service |
| `service enable <name>` | Yes | Enable at boot |
| `service disable <name>` | Yes | Disable at boot |
| `service status <name>` | No | Show status |

## cron

See [Cron](/docs/commands/cron).

| Command | Root | Key flags |
|---|---|---|
| `cron add <id>` | Yes | `--command`, `--schedule`, frequency flags, `--user`, `--output`, `--error-output`, `--append-output`, `--discard-output`, `--working-dir`, `--env`, `--enabled`, `--disabled` |
| `cron remove <id>` | Yes | |
| `cron modify <id>` | Yes | `--user`, `--command`, `--schedule`, `--output`, `--error-output`, `--working-dir`, `--env` |
| `cron list` | No | |
| `cron info <id>` | No | |
| `cron enable <id>` | No | |
| `cron disable <id>` | No | |

Frequency flags: `--every-minute`, `--every-five-minutes`, `--every-ten-minutes`, `--every-fifteen-minutes`, `--every-thirty-minutes`, `--hourly`, `--daily`, `--weekly`, `--monthly`, `--yearly`.

## daemon

See [Daemons](/docs/commands/daemons).

| Command | Root | Key flags |
|---|---|---|
| `daemon add <name>` | Yes | `--command`, `--directory`, `--user`, `--processes`, `--autostart`, `--autorestart`, `--install-supervisor`, `--startsecs`, `--startretries`, `--stopsignal`, `--stopwaitsecs`, `--exitcodes`, `--stdout-logfile`, `--stderr-logfile`, `--environment` |
| `daemon remove <name>` | Yes | `--stop`, `--delete-logs`, `--force` |
| `daemon modify <name>` | Yes | `--command`, `--directory`, `--user`, `--processes`, `--environment` |
| `daemon start <name>` | Yes | |
| `daemon stop <name>` | Yes | |
| `daemon restart <name>` | Yes | |
| `daemon status <name>` | No | |
| `daemon list` | No | |
| `daemon logs <name>` | No | `--lines`, `--follow`, `--stderr`, `--stdout` |

## project

See [Projects](/docs/commands/projects).

| Command | Root | Key flags |
|---|---|---|
| `project add <name>` | Yes | `--path`, `--nginx`, `--apache`, `--no-vhost`, `--domains`, `--port`, `--web-root`, `--user`, `--group`, `--ssl`, `--email`, `--redirect-http`, `--git`, `--branch`, `--php`, `--node`, `--ruby`, `--static`, `--php-version`, `--public-dir`, `--proxy-port` |
| `project remove <name>` | Yes | `--remove-vhost`, `--remove-ssl`, `--delete-files`, `--keep-files`, `--force` |
| `project modify <name>` | Yes | `--path`, `--domains`, `--add-domain`, `--remove-domain`, `--php-version`, `--public-dir`, `--proxy-port` |
| `project list` | No | |
| `project info <name>` | No | |
| `project enable <name>` | Yes | |
| `project disable <name>` | Yes | |
| `project reload <name>` | Yes | |

## web

See [Web server commands](/docs/commands/projects#web-server-commands).

| Command | Root | Description |
|---|---|---|
| `web test` | No | Test the web server config |
| `web reload` | Yes | Reload gracefully |
| `web restart` | Yes | Restart the web server |

## ssl

See [Certificates](/docs/commands/certificates).

| Command | Root | Key flags |
|---|---|---|
| `ssl add <project>` | Yes | `--domains`, `--email`, `--staging`, `--redirect-http` |
| `ssl remove <project>` | Yes | |
| `ssl renew` | Yes | `--project` |
| `ssl status [project]` | No | |

## mysql

See [MySQL commands](/docs/commands/system#mysql-commands).

| Command | Root | Key flags |
|---|---|---|
| `mysql config set` | Yes | `--host`, `--port`, `--user`, `--password`, `--socket` |
| `mysql config show` | No | |
| `mysql test` | No | |
| `mysql install` | Yes | `--version`, `--secure` |
| `mysql database add <name>` | No | `--charset`, `--collation`, `--if-not-exists` |
| `mysql database remove <name>` | No | |
| `mysql database list` | No | |
| `mysql user add <name>` | No | `--host`, `--password`, `--grant-db`, `--privileges`, `--preset` |
| `mysql user remove <name>` | No | `--host` |
| `mysql user list` | No | |
| `mysql user info <name>` | No | |
| `mysql grant <user> <database>` | No | `--privileges`, `--preset` |
| `mysql revoke <user> <database>` | No | |

Privilege presets: `readonly`, `app`, `admin`.

## cache

See [Cache commands](/docs/commands/system#cache-commands).

| Command | Root | Key flags |
|---|---|---|
| `cache install <driver>` | Yes | `--version`, `--port`, `--bind`, `--memory`, `--enable`, `--start` |
| `cache remove <driver>` | Yes | `--purge` |
| `cache start <driver>` | Yes | |
| `cache stop <driver>` | Yes | |
| `cache restart <driver>` | Yes | |
| `cache status [driver]` | No | |
| `cache config <driver>` | No | |

Drivers: `redis`, `memcached`.

## firewall

See [Firewall commands](/docs/commands/system#firewall-commands).

| Command | Root | Key flags |
|---|---|---|
| `firewall status` | No | |
| `firewall enable` | Yes | `--allow-ssh`, `--ssh-port` |
| `firewall disable` | Yes | |
| `firewall allow <port>` | Yes | `--protocol`, `--from`, `--comment` |
| `firewall deny <port>` | Yes | `--protocol` |
| `firewall allow-ip <ip-or-cidr>` | Yes | `--to`, `--port` |
| `firewall deny-ip <ip-or-cidr>` | Yes | |
| `firewall rule list` | No | |
| `firewall rule remove <id>` | Yes | |

## server

See [Server status commands](/docs/commands/system#server-status-commands).

| Command | Root | Key flags |
|---|---|---|
| `server status` | No | |
| `server cpu` | No | |
| `server memory` | No | |
| `server disk` | No | `--path` |
| `server load` | No | |
| `server services` | No | `--failed` |

## Action names

Every command maps to a stable action name used in JSON output (the `action` field) and reserved for the future agent. Examples: `user.add`, `firewall.enable`, `project.add`, `mysql.database.add`, `cron.modify`. See the [JSON output](/docs/reference/exit-codes#json-result-shape) reference for the result shape.
