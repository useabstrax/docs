# Projects

The `project` group sets up a web application: it can create the project directory, set ownership, write an nginx virtual host, and record the project's state. Project state is stored as JSON in:

```text
/var/lib/abstrax/projects/<name>.json
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

## `project add`

Create a new project.

```bash
abstrax project add <name> [flags]
```

If `--path` is not given, it defaults to `/var/www/<name>`.

| Flag | Default | Description |
|---|---|---|
| `--path` | `/var/www/<name>` | Project root path |
| `--nginx` | `true` | Use nginx (default) |
| `--apache` | `false` | Use Apache (not yet implemented) |
| `--no-vhost` | `false` | Do not create a virtual host |
| `--domains` | | Comma-separated domain names |
| `--port` | `80` | HTTP port |
| `--web-root` | | Custom web root directory |
| `--user` | `www-data` | Project owner user |
| `--group` | `www-data` | Project owner group |
| `--ssl` | `false` | Enable SSL (requires certbot) |
| `--email` | | Email for the SSL certificate |
| `--redirect-http` | `true` | Redirect HTTP to HTTPS |
| `--git` | | Git repository URL |
| `--branch` | `main` | Git branch to deploy |
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
| PHP | `php{version}-fpm` package | `php{version}-fpm`, `php{version}-cli`; enables and starts PHP-FPM |
| Node.js | `node --version` major matches | NodeSource repository for the requested major, then `nodejs` |
| Ruby | `ruby --version` matches major.minor | `ruby{version}` via apt, or `ruby-full` as a fallback |

Use `--dry-run` to preview the prompt and installation steps without making changes.

### Examples

```bash
# Static site
sudo abstrax project add myapp --path=/var/www/myapp \
  --domains=myapp.com,www.myapp.com --static

# PHP application (uses default PHP 8.5)
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

- [Creating a project](/docs/guides/creating-a-project)
- [Web server](/docs/commands/web)
- [Certificates (SSL)](/docs/commands/certificates)
- [Security](/docs/reference/security)
