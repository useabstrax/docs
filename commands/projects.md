# Projects

The `project` group sets up a web application: it can create the project directory, record the intended owner, write an nginx virtual host, and save the project's state. Project state is stored as JSON in:

```text
/etc/abstrax/projects/<name>.json
```

```text
abstrax project <action> [arguments] [flags]
```

## Permissions

`add`, `remove`, `modify`, `enable`, `disable`, and `reload` require root. The read-only commands `list` and `info` do not require root.

## Project names

A project name may contain letters, digits, underscores, and hyphens, up to 64 characters.

## Runtimes and web servers

A project has a runtime and a web server backend.

| Runtime | Flag | Notes |
|---|---|---|
| Static | `--static` | The default if no runtime flag is given |
| PHP | `--php` | Uses PHP-FPM; set `--php-version` (default `8.5`) and `--public-dir` |
| Node.js | `--node` | Proxies to a local port set with `--proxy-port`; set `--node-version` (default `24`) |
| Ruby | `--ruby` | Proxies to a local port set with `--proxy-port`; set `--ruby-version` (default `4.0`) |

The web server defaults to nginx. The `--apache` flag selects Apache, but Apache support is not yet implemented; use nginx.

## Application code

`project` commands set up server-side infrastructure: the project directory, intended ownership (recorded in state), nginx virtual host, and runtime. They do not deploy application source code.

After creating a project, deploy your code separately - for example with CI/CD, `git clone`/`git pull`, rsync, or another deployment tool. Abstrax does not clone repositories or check out branches as part of `project add`.

## Ownership

Abstrax supports two project ownership modes. **No separate isolation flag is required** - the mode is inferred from whether you supply `--user`.

### Shared web user mode (default)

When you omit `--user`, Abstrax uses **shared web user mode**:

| Setting | Value |
|---|---|
| Project owner | `www-data` |
| Project group | `www-data` |
| Runtime user | `www-data` (via the system PHP-FPM pool) |
| Default path | `/var/www/<name>` |

```bash
sudo abstrax project add example.com
# path: /var/www/example.com, owner: www-data:www-data
```

This preserves existing behaviour. Commands and paths you used before continue to work unchanged.

### User isolated mode

When you **explicitly** pass `--user`, Abstrax selects **user isolated mode** automatically:

| Setting | Value |
|---|---|
| Project owner | the supplied Linux user |
| Project group | that user's primary group |
| Runtime user | the same user (dedicated PHP-FPM pool for PHP projects) |
| Default path | `<resolved home directory>/<name>` |

```bash
sudo abstrax project add example.com --user=mike
# path: /home/mike/example.com (home resolved from the system account)
```

Abstrax resolves the user's real home directory, UID, GID, and primary group from the operating system. It does **not** use `SUDO_USER` unless you pass that name explicitly with `--user`.

### Path precedence

| User | Path | Result |
|---|---|---|
| omitted | omitted | `/var/www/<name>`, `www-data:www-data`, shared mode |
| supplied | omitted | `<home>/<name>`, user isolated mode |
| omitted | supplied | explicit path, `www-data:www-data`, shared mode |
| supplied | supplied | explicit path, user isolated mode |

An explicit `--path` always wins. Without `--path`, user isolated projects are created under the resolved home directory; shared mode projects use `/var/www/<name>`.

### Path examples (all four combinations)

```bash
# 1. No user, no path → /var/www/example, www-data:www-data
sudo abstrax project add example

# 2. User, no path → /home/mike/example, mike:mike
sudo abstrax project add example --user=mike

# 3. Path, no user → /srv/sites/example, www-data:www-data
sudo abstrax project add example --path=/srv/sites/example

# 4. User and path → /srv/sites/example, mike:mike (with validation)
sudo abstrax project add example --user=mike --path=/srv/sites/example
```

### Custom paths in user isolated mode

- Paths **inside the selected user's home directory** are allowed (for example `/home/mike/projects/example`).
- Paths **inside another user's home directory** are rejected.
- Paths **outside the user's home** are allowed only when inside an [approved shared project root](#approved-shared-project-roots) configured in `/etc/abstrax/config.json`.
- Abstrax **never** recursively `chown`s an existing directory owned by another user.
- Abstrax **never** uses the approved root itself or a user's home directory as the project directory.

### Approved shared project roots

Configure approved roots for user isolated projects outside home directories:

```bash
sudo abstrax config set projects.approved_roots /srv/sites /srv/www
```

Or edit `/etc/abstrax/config.json`:

```json
{
  "projects": {
    "approved_roots": ["/srv/sites", "/srv/www"]
  }
}
```

### Nginx and PHP in user isolated mode

- **Nginx** continues to run as `www-data`.
- Abstrax sets the **other execute** bit (`chmod o+x`) on existing directories along the path from the user's home to the project so nginx can traverse without filesystem ACLs. This does not grant read access to private home files. Deeper paths inside the project (for example a `public` directory) are created by your deployment tool and must be readable by the web server once your application is deployed.
- **PHP** runs as the project user through a **dedicated PHP-FPM pool** with a project-specific Unix socket.
- Socket permissions use `listen.mode = 0660` with `listen.group = www-data` (not `0666`).

### Project removal

`project remove` reverses the correct mode:

- **Shared mode:** removes the nginx vhost (by default) and project state; existing behaviour is unchanged.
- **User isolated mode:** also removes the dedicated PHP-FPM pool and the socket configuration. It does **not** delete the user's home directory, remove the Linux user, or revert traverse permissions on shared parent directories (other projects in the same home may still need them).

Ownership mode and managed resources are read from project state (`/etc/abstrax/projects/<name>.json`), not guessed from the filesystem.

### Legacy `--group` flag

In shared mode you may pass `--group` to record a group in project state. In user isolated mode the primary group is always derived from the selected user.

## `project add`

Create a new project.

```bash
abstrax project add <name> [flags]
```

If `--path` is not given, the default depends on ownership mode: `/var/www/<name>` in shared mode, or `<home>/<name>` when `--user` is set.

| Flag | Default | Description |
|---|---|---|
| `--path` | `/var/www/<name>` when `--user` is omitted; `<home>/<name>` when `--user` is set | Project root path |
| `--nginx` | `true` | Use nginx (default) |
| `--apache` | `false` | Use Apache (not yet implemented) |
| `--no-vhost` | `false` | Do not create a virtual host |
| `--domains` | | Comma-separated domain names |
| `--port` | `80` | HTTP port |
| `--web-root` | | Custom web root directory |
| `--user` | | Linux user for a user-owned project (omit for shared `www-data` mode) |
| `--group` | `www-data` in shared mode | Project group for shared mode only |
| `--ssl` | `false` | Enable SSL (requires certbot) |
| `--email` | | Email for the SSL certificate |
| `--redirect-http` | `true` | Redirect HTTP to HTTPS |
| `--php` | `false` | PHP application |
| `--node` | `false` | Node.js application |
| `--ruby` | `false` | Ruby application |
| `--static` | `false` | Static site (default behaviour) |
| `--php-version` | `8.5` | PHP version |
| `--node-version` | `24` | Node.js version |
| `--ruby-version` | `4.0` | Ruby version |
| `--public-dir` | | Public directory relative to the project path |
| `--proxy-port` | `0` | Local port to proxy to (Node.js/Ruby) |

Domains are validated. Each must be a well-formed domain name.

If a virtual host will be created (the default), Abstrax checks that the requested web server is installed. When nginx is missing, the command stops with a message pointing you to `sudo abstrax web install`. Use `--no-vhost` to create the project directory without this check.

### Runtime installation

When you choose a PHP, Node.js, or Ruby runtime, Abstrax checks that the requested version is installed on the server before creating the project. Static projects skip this check.

If the runtime is missing, you see a message like:

```text
PHP 8.5 is not installed on this server.
Abstrax can install PHP 8.5.
Install PHP 8.5 now? [y/N]
```

The version shown is the one you passed with `--php-version`, `--node-version`, or `--ruby-version`, or the default for that runtime if you omitted the flag.

- Answer **yes** (or pass global `--yes`) and Abstrax installs the runtime, then continues with project creation.
- Answer **no** and the command fails with an error; nothing is changed.

The same check runs for `project modify` when the project uses a PHP, Node.js, or Ruby runtime (for example when you change `--php-version` to a version that is not yet installed).

| Runtime | How Abstrax checks | What it installs |
|---|---|---|
| PHP | `php{version}-fpm` package | `php{version}-fpm`, `php{version}-cli`, plus extensions from [`config php.extensions`](/docs/commands/config); enables and starts PHP-FPM |
| Node.js | `node --version` major matches | NodeSource repository for the requested major, then `nodejs` |
| Ruby | `ruby --version` matches major.minor | `ruby{version}` via apt, or `ruby-full` as a fallback |

PHP extensions are configured server-wide with `abstrax config` (default: `mysql`, `xml`, `curl`, `mbstring`, `zip`, `bcmath`, `gd`, `intl`, `redis`, `sqlite3`; `pcntl` and `posix` are included in `php*-cli`). See [Config](/docs/commands/config).

Use `--dry-run` to preview the prompt and installation steps without making changes.

### Examples

```bash
# Shared static site (default mode)
sudo abstrax project add myapp \
  --domains=myapp.com,www.myapp.com --static

# User isolated PHP application in the user's home
sudo abstrax project add myapp --user=mike \
  --domains=myapp.com --php --public-dir=public

# Shared PHP application with custom path
sudo abstrax project add myapp --path=/var/www/myapp \
  --domains=myapp.com --php --public-dir=public

# PHP application with a specific version
sudo abstrax project add myapp --path=/var/www/myapp \
  --domains=myapp.com --php --php-version=8.4 --public-dir=public

# Node.js application (reverse proxy, uses default Node.js 24)
sudo abstrax project add myapp --path=/var/www/myapp \
  --domains=myapp.com --node --proxy-port=3000

# Node.js application with a specific version
sudo abstrax project add myapp --path=/var/www/myapp \
  --domains=myapp.com --node --node-version=22 --proxy-port=3000

# Ruby application (reverse proxy, uses default Ruby 4.0)
sudo abstrax project add myapp --path=/var/www/myapp \
  --domains=myapp.com --ruby --proxy-port=3000

# Ruby application with a specific version
sudo abstrax project add myapp --path=/var/www/myapp \
  --domains=myapp.com --ruby --ruby-version=3.4 --proxy-port=3000
```

### Example output

```text
Project myapp created.
  Path:       /var/www/myapp
  Web server: nginx
  Domains:    myapp.com, www.myapp.com
  Vhost:      /etc/nginx/sites-available/abstrax-myapp
```

## `project remove`

Remove a project. This is a destructive command and asks for confirmation unless `--yes` is given. By default the project files are kept and the nginx virtual host is removed.

```bash
sudo abstrax project remove <name> [flags]
```

| Flag | Default | Description |
|---|---|---|
| `--remove-vhost` | `true` | Remove the nginx virtual host |
| `--remove-ssl` | `false` | Remove the SSL certificate |
| `--delete-files` | `false` | Delete the project files |
| `--keep-files` | `true` | Keep the project files (default) |
| `--force` | `false` | Force removal without confirmation |

```bash
sudo abstrax project remove myapp --remove-vhost
sudo abstrax project remove myapp --delete-files --force
```

## `project modify`

Change a project's configuration.

```bash
sudo abstrax project modify <name> [flags]
```

| Flag | Description |
|---|---|
| `--path` | Change the project root path |
| `--domains` | Replace the domain list (comma-separated) |
| `--add-domain` | Add a single domain |
| `--remove-domain` | Remove a single domain |
| `--php-version` | Change the PHP version |
| `--node-version` | Change the Node.js version |
| `--ruby-version` | Change the Ruby version |
| `--public-dir` | Change the public directory |
| `--proxy-port` | Change the proxy port |

```bash
sudo abstrax project modify myapp --add-domain=www.myapp.com
```

## `project list`

List managed projects. Does not require root.

```bash
abstrax project list
```

```text
NAME    PATH              WEB SERVER   RUNTIME   DOMAINS
myapp   /var/www/myapp    nginx        php       myapp.com, www.myapp.com
```

## `project info`

Show details for a project. Does not require root.

```bash
abstrax project info myapp
```

```text
  Name:          myapp
  Path:          /var/www/myapp
  Web server:    nginx
  Runtime:       php
  PHP version:   8.5
  Public dir:    public
  Domains:       myapp.com, www.myapp.com
  SSL:           no
  Vhost:         /etc/nginx/sites-available/abstrax-myapp
  Owner:         www-data
  Created:       2024-01-01 12:00:00
  Updated:       2024-01-01 12:00:00
```

## `project inspect`

Inspect a project using the stable v1 API designed for plugins and automation. Does not require root.

```bash
abstrax project inspect myapp --json
```

```json
{
  "api_version": "v1",
  "project": {
    "name": "myapp",
    "path": "/var/www/myapp",
    "user": "www-data",
    "runtime": {
      "type": "php",
      "version": "8.5"
    },
    "domains": ["myapp.com", "www.myapp.com"],
    "services": [
      {
        "name": "worker",
        "type": "worker"
      }
    ]
  }
}
```

This API exposes non-secret information only. The existing `project info --json` output remains unchanged for backward compatibility.

## `project service`

Restart or reload project-owned supervisor services. Requires root.

```bash
sudo abstrax project service restart <project> <service>
sudo abstrax project service reload <project> <service>
```

Services must belong to the project - either recorded in project state or named `abstrax-<project>-<service>` in supervisor. These commands do not wrap arbitrary systemctl calls.

## Enable, disable, and reload

```bash
sudo abstrax project enable <name>
sudo abstrax project disable <name>
sudo abstrax project reload <name>
```

- `enable` enables the project's nginx virtual host (symlinks it into `sites-enabled`). The virtual host file is named `abstrax-<name>` in `/etc/nginx/sites-available`.
- `disable` removes that symlink.
- `reload` reloads nginx for the project.

## Related

- [Integrating with Abstrax](/docs/plugins/integrating-with-abstrax)
- [Creating a project](/docs/guides/creating-a-project)
- [Web server](/docs/commands/web)
- [Certificates (SSL)](/docs/commands/certificates)
- [Security](/docs/reference/security)
