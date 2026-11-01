# Permissions and security

Abstrax performs real changes to your server, often as root. This page explains what permissions it needs, what changes it can make, the safeguards built into the application, and how to use it safely.

## What permissions Abstrax needs

Most commands that change system state require root, because the operations themselves require root: creating users, editing `/etc/ssh`, installing packages, controlling systemd, writing to `/etc/cron.d`, managing the firewall, and so on.

Commands check for root before doing any work and stop with a clear error if it is missing. Read-only commands (`doctor`, `version`, `server status`, and the various `list`, `info`, and `status` commands) generally do not require root, though some need root to read protected files.

See [Permissions](/docs/configuration/permissions) for the full list of which commands need root.

## What changes Abstrax can make

Depending on the command, Abstrax can:

- Create, modify, lock, and remove user accounts and their groups, including granting sudo.
- Add and remove SSH authorised keys, change the SSH port, and change root-login and password-authentication settings.
- Install, remove, update, and upgrade system packages.
- Start, stop, enable, and disable systemd services.
- Create and remove cron files in `/etc/cron.d`.
- Create and control Supervisor-managed processes.
- Create nginx virtual hosts, record intended project ownership, and reload or restart nginx.
- Obtain, renew, and remove TLS certificates with Certbot.
- Create and drop MySQL databases and users, and change grants.
- Install and control Redis and Memcached.
- Enable or disable the UFW firewall and add or remove rules.

These are powerful operations. Treat Abstrax commands with the same care you would treat running the underlying tools as root.

## Safeguards in the application

The application includes several safeguards. These are accurate descriptions of what the code does, not guarantees of safety.

### Confirmation prompts for destructive commands

Commands that destroy or significantly change state ask for confirmation before proceeding. These include:

- `user remove`, `user revoke-sudo`
- `project remove`
- `firewall enable`, `firewall disable`
- `mysql database remove`, `mysql user remove`, `mysql revoke`
- `cache install redis` when binding to `0.0.0.0`

Pass `--yes` to skip the prompt in scripts. Use this only when you are sure of the command.

### Dry-run mode

`--dry-run` prints the underlying commands Abstrax would run, without running them. Use it to review a change before applying it.

### SSH configuration is validated

SSH settings are written to `/etc/ssh/sshd_config.d/00-abstrax.conf`, which sshd reads before later drop-ins such as `50-cloud-init.conf`. An existing `99-abstrax.conf` is moved into that file on the next write. If `sshd_config` does not include the drop-in directory, or sets the same keyword before `Include`, Abstrax updates that keyword in `sshd_config`. Before the change is reloaded, the configuration is validated with `sshd -t` when `sshd` is available. After reload, `sshd -T` is checked. If the new value did not take effect, the command fails and names the file that still wins. `--dry-run` writes nothing.

### Warnings before risky changes

Abstrax warns before actions that commonly cause lockouts or exposure:

- Changing the SSH port without `--allow-firewall` warns that the firewall was left unchanged. With `--allow-firewall`, the new port is opened before sshd reloads, and the previous port stays open until `--close-old-port`. If the new port does not answer, the change is rolled back.
- Disabling password authentication warns you to confirm you have a working key.
- Restarting SSH warns that active sessions will briefly disconnect.
- Enabling the firewall without `--allow-ssh` warns that you may be locked out. Abstrax also allows your current IP on the SSH port when you run `firewall enable`, `allow`, `deny`, `allow-ip`, or `deny-ip`, but only in an interactive SSH session (`SSH_CONNECTION`, `SSH_CLIENT`, or `who`). From a script, that protection does nothing (see [Firewall](/docs/commands/firewall#automatic-ssh-lockout-protection)).
- `firewall deny`, `firewall rule remove`, `firewall remove service`, and `firewall remove port` are refused when the firewall is active and the change would leave no rule allowing TCP to the SSH port. An allow limited to one address still counts. `--force` applies the change anyway. `--yes` does not skip the check. When the firewall is inactive, the change is allowed and Abstrax warns that `firewall enable` will need `--allow-ssh`.
- Binding Redis to `0.0.0.0` warns that it is exposed on all interfaces and asks for confirmation.

### Passwords are never on the command line

User passwords and the MySQL connection password are entered at a secure prompt (with `--password`), never passed as command-line arguments. The MySQL connection config is written with mode `0600` so only its owner can read it, and `mysql config show` does not print the password.

### Input validation

Arguments such as usernames, package names, database names, domains, ports, and CIDR ranges are validated before any change is made. Cron and daemon values cannot contain line breaks, carriage returns, or null bytes. MySQL privileges are limited to an allowlist, and column-level privileges are rejected. Invalid input is rejected with a clear error.

## What to review before running a command

- **Are you on the right server?** Many commands change global system state.
- **Do you understand the effect?** Use `--dry-run` and `--verbose` to see what will run.
- **Will it affect your access?** SSH and firewall changes can lock you out. Keep a second session open and have console access available.
- **Is the target correct?** Double-check the user, project, database, or rule ID before destructive commands.

## Safe usage recommendations

- Prefer `--dry-run` for unfamiliar commands.
- Keep a second SSH session open while changing SSH or firewall settings.
- Always use `--allow-ssh` when enabling the firewall.
- Test key-based login before disabling password authentication.
- Bind Redis and Memcached to `127.0.0.1` unless you specifically need network access, and use the firewall to restrict access if you do.
- Use the `--staging` flag with `ssl add` while testing to avoid Let's Encrypt rate limits.
- Avoid `--yes` in interactive use; reserve it for scripts you trust.

## Accurate scope

Abstrax reduces the chance of certain mistakes through confirmations, validation, dry-run, and SSH config validation. Firewall changes that would remove the last TCP path to SSH are refused unless you pass `--force`. Commands run with the privileges of the user who invokes them, and a valid command is applied as written. `ssh config set-port` restores the previous port when the new port does not answer. You remain responsible for the changes you apply.

## Plugins

Abstrax supports standalone executable plugins. See [Plugins](/docs/plugins/) for full documentation.

Key security properties:

- Built-in commands always take priority over plugin names.
- Plugin binaries are verified with SHA-256 checksums before installation; downloaded plugins are never executed before verification.
- Plugins are launched with direct process execution (no shell).
- The default plugin registry uses HTTPS.
- Registry-blocked plugins cannot be installed; execution is blocked unless `--allow-blocked-plugin` is used.
- Project configuration files cannot trigger automatic plugin installation.
- Privileged operations are available to plugins only through narrow, validated core commands such as `abstrax project service restart`.

Trust levels (`official`, `verified`, `community`) are policy metadata, not proof that a plugin is safe. Treat third-party plugins like any executable you install on your server.

## Related

- [Permissions](/docs/configuration/permissions)
- [Troubleshooting](/docs/reference/troubleshooting)
- [Adding SSH access](/docs/guides/adding-ssh-access)
