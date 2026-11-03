# Projects

The `project` group sets up a web application: it can create the project directory, record the intended owner, write an nginx virtual host, and save the project's state. Project state is stored as JSON in:

```text
/etc/abstrax/projects/<name>.json
```

```text
abstrax project <action> [arguments] [flags]
```

## Permissions

`add`, `remove`, `modify`, `runtime set`, `vhost add`, `vhost remove`, `redirect add`, `redirect modify`, `redirect remove`, `enable`, `disable`, and `reload` require root. The read-only commands `list`, `info`, `inspect`, and `redirect list` do not require root.

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
| `--shared-logs` | `false` | Keep the server-wide nginx logs instead of per-project log files |
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

### Nginx logs

By default the virtual host sends this project to its own log files:

```nginx
access_log /var/log/nginx/<name>-access.log;
error_log /var/log/nginx/<name>-error.log;
```

`project info` and `project inspect` include those paths as `access_log_path` and `error_log_path`.

Pass `--shared-logs` to omit both directives and keep the server-wide nginx logs. The choice is stored as `shared_logs` in the project state file. A later `project modify` that does not pass `--shared-logs` leaves that choice as it is. Pass `--shared-logs=false` on `project modify` to switch back to per-project logs.

Projects created before this behaviour keep their current virtual host until the next `project modify` rewrites it. Upgrading Abstrax does not rewrite existing vhosts. `--shared-logs` keeps the previous server-wide log behaviour.

`project remove` leaves the log files in place. `project remove --delete-files` also deletes the two per-project log files.

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
| PHP | Provider PHP-FPM package for the version | Debian: `php{version}-fpm` / `php{version}-cli` via apt. RHEL-family: Remi SCL packages such as `php83-php-fpm` / `php83-php-cli` (requires Remi; pass `--enable-required-repos` or run [`repo enable remi`](/docs/commands/repo) first). Extensions come from [`config php.extensions`](/docs/commands/config); Abstrax enables and starts the versioned PHP-FPM service |
| Node.js | `node --version` major matches | NodeSource setup for the requested major (`deb.nodesource.com` on Debian-family, `rpm.nodesource.com` on RHEL-family), then `nodejs` |
| Ruby | `ruby --version` matches major.minor on Debian-family | Debian: `ruby{version}` via apt, or `ruby-full` as a fallback. RHEL-family: stock `ruby` + `ruby-devel`. Exact version pinning on RHEL is a deliberate product limitation until a shared runtime manager is added |

PHP extensions are configured server-wide with `abstrax config` (default: `mysql`, `xml`, `curl`, `mbstring`, `zip`, `bcmath`, `gd`, `intl`, `redis`, `sqlite3`; `pcntl` and `posix` are included in `php*-cli`). See [Config](/docs/commands/config).

On RHEL-family systems, supported Remi PHP versions are currently 8.1–8.5. Unsupported versions fail with a clear error before any install attempt.

### PHP on the command line

Abstrax routes web requests to the correct PHP-FPM version per project. For command-line tasks (Artisan, Composer, queue workers, and cron jobs), use the matching versioned binary:

**Debian/Ubuntu:**

```bash
php8.5 artisan migrate
php8.4 artisan queue:work
```

**RHEL-family (Remi):**

```bash
/opt/remi/php85/root/usr/bin/php artisan migrate
```

The unversioned `php` command may not match a given project. Check the configured version with `abstrax project info <name>`.

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

### Machine-readable progress (`--json-stream`)

For agents and other control-plane consumers, use `--json-stream` (with `--yes` for non-interactive runs). Abstrax prints flushed NDJSON lines: zero or more `type=progress` events, then one `type=result` line.

```bash
sudo abstrax project add myapp \
  --domains=myapp.com --php --public-dir=public \
  --json-stream --yes
```

Example stream (abbreviated):

```json
{"type":"progress","action":"project.add","step":"validate","message":"Validating options"}
{"type":"progress","action":"project.add","step":"directories","message":"Creating project directories"}
{"type":"result","status":"success","action":"project.add","summary":"Project myapp created.","data":{...}}
```

Progress `step` values align with the hosted-agent style stages where practical (`validate`, `identity`, `path`, `runtime`, `directories`, `ownership`, `php_pool`, `nginx_vhost`, `state`, `complete`, plus install steps such as `package_update` / `ensure_php` when a runtime is installed). See [Exit codes and output](/docs/reference/exit-codes#json-stream-ndjson) for the full contract.

`--json` remains a single indented result object with no progress lines. `--json` and `--json-stream` cannot be combined.

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
| `--delete-files` | `false` | Delete the project files and the per-project nginx log files |
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
| `--path` | Change the stored project root path (does not move files on disk) |
| `--domains` | Replace the domain list (comma-separated). `--domains=""` clears every domain |
| `--clear-domains` | Remove every domain. Cannot be combined with `--domains` |
| `--add-domain` | Add a single domain |
| `--remove-domain` | Remove a single domain |
| `--php-version` | Change the PHP version. An empty value clears the stored version; the active PHP runtime keeps its default |
| `--node-version` | Change the Node.js version (empty clears; active Node.js keeps its default) |
| `--ruby-version` | Change the Ruby version (empty clears; active Ruby keeps its default) |
| `--public-dir` | Change the public directory. `--public-dir=""` clears it |
| `--proxy-port` | Change the proxy port. `--proxy-port=0` clears it |
| `--shared-logs` | Use the server-wide nginx logs. Omit the flag to keep the stored choice. `--shared-logs=false` switches back to per-project logs |

Flags that are not passed leave the stored field unchanged. A flag passed with an empty value (or `0` for `--proxy-port`) clears that field.

Removing every domain (`--domains=""`, `--clear-domains`, or the last `--remove-domain`) is refused when the project has an nginx vhost, because the vhost would be written with `server_name _` and answer for every host. `--path` must not be empty and is validated the same way as `project add`. `--public-dir` must stay inside the project path.

The runtime install check runs only when the active runtime's version (or the runtime itself) changes. The nginx vhost is rewritten only when a value that appears in it changed. A disabled vhost stays disabled across rewrites. If `nginx -t` rejects the new vhost, the previous file is restored.

To change between PHP, Node.js, Ruby, and static, use [`project runtime set`](#project-runtime-set) instead of `project modify`.

```bash
sudo abstrax project modify myapp --add-domain=www.myapp.com
sudo abstrax project modify myapp --public-dir=""
sudo abstrax project modify myapp --clear-domains   # only when there is no vhost
```

## `project runtime set`

Change a project's runtime between PHP, Node.js, Ruby, and static. Exactly one of `--php`, `--node`, `--ruby`, or `--static` is required. Registered as `project.runtime.set`.

```bash
sudo abstrax project runtime set <name> --php|--node|--ruby|--static [flags]
```

| Flag | Description |
|---|---|
| `--php` / `--node` / `--ruby` / `--static` | Target runtime (exactly one) |
| `--php-version` | PHP version (only with `--php`) |
| `--node-version` | Node.js version (only with `--node`) |
| `--ruby-version` | Ruby version (only with `--ruby`) |
| `--public-dir` | Public directory (PHP and static) |
| `--proxy-port` | Local proxy port (Node.js and Ruby) |

Version and port flags that do not match the target runtime are refused. The target runtime is installed first (with confirmation unless `--yes`).

Fields from the old runtime are cleared: PHP keeps `php_version` and `public_dir`; Node.js and Ruby keep their version and `proxy_port` (the port carries over between them); static keeps `public_dir` only. On a user-isolated project, leaving PHP removes its PHP-FPM pool, entering PHP creates one, and changing the PHP version creates the new pool, switches the vhost, then removes the old pool.

The nginx vhost is regenerated. If `nginx -t` rejects it, the previous vhost is restored and the project is left unchanged. A disabled vhost stays disabled. `--dry-run` previews the new state without changing anything.

```bash
sudo abstrax project runtime set myapp --php --php-version=8.4 --public-dir=public --yes
sudo abstrax project runtime set myapp --node --node-version=22 --proxy-port=3000 --yes
sudo abstrax project runtime set myapp --static --yes
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
  Access log:    /var/log/nginx/myapp-access.log
  Error log:     /var/log/nginx/myapp-error.log
  Owner:         www-data
  Created:       2024-01-01 12:00:00
  Updated:       2024-01-01 12:00:00
```

When SSL is configured, `project info` (and the project state file) expose an `ssl` object with `certificate_name`, `domains`, `redirect_http`, `staging`, `issued_at`, `expires_at`, and `last_error`. `ssl_enabled` is still written. Projects that only have the older `ssl_enabled` flag are adopted the first time Abstrax needs a certificate name (from the `ssl_certificate` path in the existing vhost, then from the first domain). When redirects exist, the text output notes the count and points at `project redirect list`.

## `project inspect`

Inspect a project using the stable v1 API designed for plugins and automation. Does not require root. Still `api_version: v1`; new fields are additive.

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
    "web_server": "nginx",
    "runtime": {
      "type": "php",
      "version": "8.5"
    },
    "public_dir": "public",
    "domains": ["myapp.com", "www.myapp.com"],
    "vhost": {
      "present": true,
      "path": "/etc/nginx/sites-available/abstrax-myapp",
      "enabled": true
    },
    "shared_logs": false,
    "access_log_path": "/var/log/nginx/myapp-access.log",
    "error_log_path": "/var/log/nginx/myapp-error.log",
    "ssl": {
      "enabled": false
    },
    "services": [
      {
        "name": "worker",
        "type": "worker"
      }
    ],
    "created_at": "2026-11-01T10:00:00Z",
    "updated_at": "2026-11-02T12:30:00Z"
  }
}
```

This API exposes non-secret information only. The response may also include `group` and `ownership_mode` when set, and `proxy_port` when the runtime uses one. `access_log_path` and `error_log_path` are omitted when the project uses `--shared-logs`. `public_dir` and `proxy_port` are omitted when unset.

`vhost.present` is true when the project records a vhost path. `vhost.enabled` is false when `vhost_disabled` is set in state; otherwise it is read from disk (Debian-family: `sites-enabled` symlink; RHEL-family: file in `conf.d`, with a `.disabled` sibling meaning disabled).

When SSL is enabled, `ssl` also includes `certificate_name`, `domains`, `redirect_http`, `staging`, and `expires_at` when known. `created_at` and `updated_at` are RFC3339 UTC strings. `project info --json` includes the same log paths and an `ssl` object with certificate metadata when SSL is configured.

## `project vhost`

Add or remove a project's managed nginx virtual host without deleting the project. Registered as `project.vhost.add` and `project.vhost.remove`.

### `project vhost add`

Create an nginx vhost for a project that has none (for example one created with `--no-vhost`).

```bash
sudo abstrax project vhost add <name> [flags]
```

| Flag | Description |
|---|---|
| `--domains` | Override stored domains (comma-separated) |
| `--port` | Override stored HTTP port |
| `--web-root` | Override stored web root |

`project add` stores `port` and `web_root` so later vhost rewrites keep them. If `nginx -t` rejects the new vhost, the previous file (if any) is restored.

```bash
sudo abstrax project vhost add myapp --domains=myapp.com,www.myapp.com
```

### `project vhost remove`

Delete the vhost (and its `.disabled` copy), clear `vhost_path` and `ssl_enabled`, reload nginx, and keep the project files and state.

```bash
sudo abstrax project vhost remove <name>
```

## `project redirect`

Manage exact-path redirects in a project's managed nginx vhost. Redirects are stored in project state and written as `location = <source> { return <code> <destination>; }` blocks ahead of the runtime locations, sorted by ID, so `project modify` keeps them. The project needs a managed nginx vhost.

Sources and destinations containing whitespace, quotes, `;`, braces, backslashes, or `$` are rejected. The vhost is written before state is saved; a vhost that nginx rejects is rolled back to its previous contents.

### `project redirect list`

List redirects. Does not require root. Registered as `project.redirect.list`.

```bash
abstrax project redirect list <project>
```

```text
ID      SOURCE      DESTINATION              CODE
r1a2b3  /old-page   /new-page                301
r4c5d6  /blog       https://blog.example.com 302
```

### `project redirect add`

Add a redirect. Registered as `project.redirect.add`.

```bash
sudo abstrax project redirect add <project> <source> <destination> [--code]
```

| Flag | Default | Description |
|---|---|---|
| `--code` | `301` | HTTP status: `301`, `302`, `307`, or `308` |

`<source>` must be a path starting with `/`. `<destination>` is a path starting with `/` or an absolute `http(s)` URL. Each redirect gets a short ID shown by `list`.

```bash
sudo abstrax project redirect add myapp /old-page /new-page
sudo abstrax project redirect add myapp /blog https://blog.example.com/ --code 302
```

### `project redirect modify`

Change a redirect by ID. Registered as `project.redirect.modify`.

```bash
sudo abstrax project redirect modify <project> <id> [flags]
```

| Flag | Description |
|---|---|
| `--source` | New source path |
| `--destination` | New destination path or URL |
| `--code` | New HTTP status code |

### `project redirect remove`

Remove a redirect by ID. Registered as `project.redirect.remove`.

```bash
sudo abstrax project redirect remove <project> <id>
```

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

- `enable` enables the project's nginx virtual host (symlinks it into `sites-enabled` on Debian-family, or restores the conf.d file on RHEL-family). The virtual host file is named `abstrax-<name>`.
- `disable` disables that vhost.
- Both store the choice as `vhost_disabled` in project state so a later `project modify` or `project runtime set` that rewrites the file keeps a disabled vhost disabled.
- `reload` reloads nginx for the project.

## Related

- [Integrating with Abstrax](/docs/plugins/integrating-with-abstrax)
- [Deploy plugin](/docs/plugins/official/deploy) — zero-downtime application deployments on top of project infra
- [Composer plugin](/docs/plugins/official/composer) — install Composer and run it with the project's PHP CLI
- [Creating a project](/docs/guides/creating-a-project)
- [Web server](/docs/commands/web)
- [Certificates (SSL)](/docs/commands/certificates)
- [Security](/docs/reference/security)
