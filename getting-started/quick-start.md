# Quick start

This guide walks through your first commands with Abstrax. The early steps are safe to run and do not change the system. Later steps that change state are clearly marked.

## 1. Check the CLI works

Confirm the binary runs and see the top-level help:

```bash
abstrax --help
```

You should see the list of command groups (user, ssh, package, service, cron, daemon, project, web, ssl, mysql, cache, firewall, server, and so on).

## 2. Inspect the system

`abstrax doctor` inspects the current server and reports what it finds. It does not require root and makes no changes, so it is a safe first command:

```bash
abstrax doctor
```

Example output:

```text
  OS:                  Ubuntu 22.04.3 LTS
  Version:             22.04
  Kernel:              5.15.0-91-generic
  Architecture:        x86_64

  Package manager:     apt
  Service manager:     systemd
  Firewall backend:    ufw

  Running as root:     no
  Platform support:    full

  Tools:
    nginx:           available
    apache2:         not found
    certbot:         available
    mysql:           available
    ...
```

This tells you which underlying tools are installed. Abstrax relies on these tools, so the list is useful before you try a command in a given area.

## 3. View available commands

Each command group has its own help. To see what a group can do, ask for its help:

```bash
abstrax user --help
abstrax firewall --help
abstrax package --help
```

To see the flags and arguments for a specific command, add `--help` at the end:

```bash
abstrax user add --help
```

## 4. Run a safe read-only command

Status and listing commands only read state. They do not change anything and most do not require root:

```bash
abstrax server status
abstrax user list
abstrax firewall status
```

`abstrax server status` shows hostname, uptime, load average, CPU cores, memory, disk usage, OS, kernel, and private IP addresses.

## 5. Preview a change with --dry-run

Before making a real change, you can preview it. With `--dry-run`, Abstrax prints the underlying commands it would run instead of running them:

```bash
sudo abstrax user add deploy --grant-sudo --dry-run
```

You will see lines like:

```text
[dry-run] would run: useradd ...
```

Note that some commands still need to read system state, which can require root even in dry-run mode.

## 6. Make a real change

When you are ready, drop `--dry-run`. Commands that change system state generally need root, so use `sudo`:

```bash
sudo abstrax user add deploy --grant-sudo --create-home
```

Example output:

```text
User deploy created.
  Home:   /home/deploy
  Shell:  /bin/bash
  UID:    1001
  Groups: deploy, sudo
  Sudo:   granted
```

Destructive commands such as `user remove` or `firewall disable` ask for confirmation first. Pass `--yes` to skip the prompt in scripts.

## Understanding command output

By default, Abstrax prints human-readable text. The general structure is:

- A success line (often shown in green) summarising what happened.
- Indented detail lines with relevant fields.
- Warnings are printed to standard error and prefixed with `WARNING:`.
- Errors are printed to standard error and prefixed with `ERROR:`.

For automation, add `--json` to any command:

```bash
abstrax doctor --json
```

JSON results follow a consistent shape:

```json
{
  "status": "success",
  "action": "doctor.check",
  "summary": "System inspection complete.",
  "data": { "...": "..." }
}
```

On failure the shape is:

```json
{
  "status": "error",
  "action": "user.add",
  "error_code": "command_error",
  "message": "user deploy already exists"
}
```

A successful command exits with status `0`. Any error exits with status `1`.

## Where to go next

- [Commands overview](../commands/index.md) – every command group explained.
- [Creating a user](../guides/creating-a-user.md) – a worked example.
- [Configuration](../configuration/index.md) – how Abstrax stores state and config.
- [Permissions and security](../reference/security.md) – when root is needed and why.
