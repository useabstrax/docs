# Abstrax

Abstrax is a command line tool for managing common Linux server tasks. It wraps the everyday administration commands for users, packages, services, web projects, databases, firewalls, and more behind a single, consistent interface.

The aim is simple: you should not need to remember the exact syntax of `useradd`, `ufw`, `supervisorctl`, `certbot`, or nginx configuration files to get routine server work done.

## What Abstrax is

Abstrax is a single Go binary that runs directly on the server you want to manage. Every command follows the same pattern:

```text
abstrax <group> <action> [arguments] [flags]
```

For example:

```bash
abstrax user add deploy --grant-sudo
abstrax package install nginx
abstrax firewall allow 443 --protocol=tcp
```

Each command validates its input, performs the requested change, and prints a clear result. Add `--json` to any command to get machine-readable output for scripts and automation.

## Who it is for

- People who manage one or more Linux servers and want a consistent set of commands.
- Developers deploying web applications who need to set up users, projects, databases, and certificates.
- Teams who want predictable, scriptable server operations with structured output.

You do not need to be a Linux expert to use Abstrax, but you should understand that the commands change real system state.

## What problems it solves

- **Inconsistent syntax.** Different tools (`useradd`, `ufw`, `certbot`, `supervisorctl`) each have their own flags and conventions. Abstrax gives them a single, predictable style.
- **Repetitive setup.** Creating a web project usually means making a directory, setting ownership, writing an nginx virtual host, and reloading the web server. Abstrax does these steps together.
- **Scripting.** The `--json` flag produces stable, structured output that is easier to parse than the text output of individual system tools.
- **Safety.** Destructive commands ask for confirmation, SSH configuration is validated before it is applied, and many commands support `--dry-run` so you can preview changes.

## What it can help with

| Area | Examples |
|---|---|
| Users | Create and remove users, manage groups, grant or revoke sudo, lock accounts |
| SSH | Add authorised keys, change the SSH port, disable root login or password auth |
| Packages | Install, remove, update, and upgrade apt packages |
| Services | Start, stop, restart, enable, and check systemd services |
| Cron | Add, modify, list, enable, and disable scheduled jobs |
| Daemons | Run long-lived processes under Supervisor |
| Projects | Create nginx-backed web projects for static, PHP, Node.js, or Ruby apps |
| Certificates | Obtain and renew Let's Encrypt certificates with Certbot |
| Databases | Manage MySQL/MariaDB databases, users, and grants |
| Cache | Install and manage Redis or Memcached |
| Firewall | Enable UFW and manage allow/deny rules |
| Server status | View CPU, memory, disk, load, and running services |

## What it does not try to be

- It is not a configuration management system like Ansible, Puppet, or Chef. It does not enforce a desired state over time.
- It is not a replacement for the underlying tools. It calls `apt`, `systemctl`, `ufw`, `certbot`, `supervisorctl`, and `mysql` on your behalf.
- It does not currently provide a hosted control plane or remote agent. A future agent is described in the code and README, but it is not implemented yet. See [Supported platforms](/reference/supported-platforms.html) for current scope.

## A short example

```bash
# Inspect the system first – this command does not require root
abstrax doctor

# Create a deploy user with sudo
sudo abstrax user add deploy --grant-sudo --create-home

# Preview a firewall change without applying it
sudo abstrax firewall enable --allow-ssh --dry-run

# Get JSON output for a script
abstrax server status --json
```

## Where to go next

- [Installation](/getting-started/installation.html) – install Abstrax on your server.
- [Quick start](/getting-started/index.html) – your first commands.
- [Commands](/commands/index.html) – detailed documentation for every command group.
- [Command reference](/reference/index.html) – every command and flag in one place.
- [Configuration](/configuration/index.html) – config files, environment, and permissions.
- [Permissions and security](/reference/security.html) – what Abstrax needs and why.
- [Troubleshooting](/reference/troubleshooting.html) – common problems and fixes.
- [Contributing](/contributing/index.html) – build, test, and develop Abstrax.
