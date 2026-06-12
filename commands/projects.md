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
| PHP | `--php` | Uses PHP-FPM; set `--php-version` and `--public-dir` |
| Node.js | `--node` | Proxies to a local port set with `--proxy-port` |
| Ruby | `--ruby` | Proxies to a local port set with `--proxy-port` |

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
| `--php-version` | `8.2` | PHP version |
| `--public-dir` | | Public directory relative to the project path |
| `--proxy-port` | `0` | Local port to proxy to (Node.js/Ruby) |

Domains are validated. Each must be a well-formed domain name.

If a virtual host will be created (the default), Abstrax checks that the requested web server is installed. When nginx is missing, the command stops with a message pointing you to `sudo abstrax web install`. Use `--no-vhost` to create the project directory without this check.

### Examples

```bash
# Static site
sudo abstrax project add myapp --path=/var/www/myapp \
  --domains=myapp.com,www.myapp.com --static

# PHP application
sudo abstrax project add myapp --path=/var/www/myapp \
  --domains=myapp.com --php --php-version=8.2 --public-dir=public

# Node.js application (reverse proxy)
sudo abstrax project add myapp --path=/var/www/myapp \
  --domains=myapp.com --node --proxy-port=3000

# Ruby application (reverse proxy)
sudo abstrax project add myapp --path=/var/www/myapp \
  --domains=myapp.com --ruby --proxy-port=3000
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
