# Creating a project

This guide creates an nginx-backed web project, then adds a TLS certificate. It uses the [`project`](/docs/commands/projects), [`web`](/docs/commands/web), and [`ssl`](/docs/commands/certificates) commands.

These commands change system state and require root. Make sure nginx is installed first (`abstrax doctor` will show you).

## 1. Create the project

Pick the example that matches your application and ownership model.

### Shared web user project (default)

Omit `--user` to use `www-data` and `/var/www/<name>`:

```bash
sudo abstrax project add myapp \
  --domains=myapp.com,www.myapp.com \
  --static
```

### User isolated project

Pass `--user` to run the application as that Linux user. Abstrax resolves the home directory from the system account:

```bash
sudo abstrax project add myapp --user=mike \
  --domains=myapp.com \
  --php --public-dir=public
```

No separate isolation flag is needed. The project is created at `/home/mike/myapp` (or the user's actual home path).

### Static site with explicit path

```bash
sudo abstrax project add myapp \
  --path=/var/www/myapp \
  --domains=myapp.com,www.myapp.com \
  --static
```

### PHP application

```bash
sudo abstrax project add myapp \
  --path=/var/www/myapp \
  --domains=myapp.com \
  --php --public-dir=public
```

Omit `--php-version` to use the default (`8.5`). Pass `--php-version=8.4` (or another installed version) when you need something else.

If PHP is not installed yet, Abstrax asks whether to install the requested version before continuing. Answer yes to install it, or no to abort. Pass `--yes` to install without prompting. PHP is installed with FPM, CLI, and the extension packages configured in [`abstrax config`](/docs/commands/config) (defaults include `mysql`, `xml`, `curl`, `mbstring`, `zip`, `bcmath`, `gd`, `intl`, `redis`, and `sqlite3`; `pcntl` and `posix` come with `php*-cli`).

For **user isolated** PHP projects, Abstrax also creates a dedicated PHP-FPM pool running as the project user and configures nginx to use the project socket.

### Node.js or Ruby application (reverse proxy)

```bash
sudo abstrax project add myapp \
  --path=/var/www/myapp \
  --domains=myapp.com \
  --node --proxy-port=3000
```

For Ruby, use `--ruby` instead of `--node`. Omit `--node-version` or `--ruby-version` to use the defaults (`24` and `4.0` respectively). If the runtime is missing, Abstrax offers to install it before creating the project.

Expected output:

```text
Project myapp created.
  Path:       /var/www/myapp
  Owner:      www-data:www-data
  Web server: nginx
  Domains:    myapp.com, www.myapp.com
  Vhost:      /etc/nginx/sites-available/abstrax-myapp
```

If you omit `--path` and `--user`, the path defaults to `/var/www/<name>`.

### Path precedence

| `--user` | `--path` | Result |
|---|---|---|
| omitted | omitted | `/var/www/<name>`, shared `www-data` mode |
| set | omitted | `<resolved home>/<name>`, user isolated mode |
| omitted | set | explicit path, shared `www-data` mode |
| set | set | explicit path, user isolated mode (validated) |

### Custom paths for user isolated projects

- Paths inside the selected user's home are allowed.
- Paths inside another user's home are rejected.
- Paths outside home require an approved shared root - configure with `sudo abstrax config set projects.approved_roots /srv/sites`.

Abstrax never recursively changes ownership of another user's directories. See [Projects](/docs/commands/projects) for full details.

### Deploy your application code

`project add` creates the directory and nginx configuration but does not put your application files on the server. Deploy source code separately - for example with CI/CD, `git clone`/`git pull`, rsync, or your existing deployment process - once the project path and virtual host are in place.

For user isolated projects, run deploy and build commands as the project user so files are owned correctly.

## 2. Confirm the nginx configuration

Test the web server configuration before relying on it:

```bash
abstrax web test
```

```text
nginx configuration test passed.
```

If the test passes, reload nginx so the new virtual host is served:

```bash
sudo abstrax web reload
```

## 3. Inspect the project

```bash
abstrax project info myapp
abstrax project list
```

## 4. Add a TLS certificate

Make sure ports 80 and 443 are open and the domains resolve to this server, then request a certificate. Use `--staging` first to test without hitting Let's Encrypt rate limits:

```bash
sudo abstrax ssl add myapp --email=admin@example.com --staging
```

When the staging run succeeds, request the real certificate:

```bash
sudo abstrax ssl add myapp --email=admin@example.com --redirect-http
```

Check the status:

```bash
abstrax ssl status myapp
```

```text
PROJECT   DOMAINS                    EXPIRY
myapp     myapp.com, www.myapp.com   2024-04-01
```

## 5. Add and remove domains later

```bash
sudo abstrax project modify myapp --add-domain=api.myapp.com
sudo abstrax project reload myapp
```

## 6. Removing the project

By default this keeps your files and removes the nginx virtual host (it asks for confirmation):

```bash
sudo abstrax project remove myapp --remove-vhost
```

For user isolated projects, Abstrax also removes the dedicated PHP-FPM pool recorded in project state.

To also delete the files:

```bash
sudo abstrax project remove myapp --delete-files --force
```

## Notes

- The current web server backend is nginx. Apache is referenced in the flags but is not yet implemented.
- Make sure the firewall allows HTTP and HTTPS: `sudo abstrax firewall allow 80` and `sudo abstrax firewall allow 443 --protocol=tcp`.
- On systems with SELinux enforcing, projects under `/home` may need additional file contexts. Abstrax prints a warning when this may apply.
- Supervisor daemons and other project processes should run as the project user in isolated mode (`daemon add --user=mike`).

## Related

- [Creating a user](/docs/guides/creating-a-user)
- [Projects](/docs/commands/projects)
- [Web server](/docs/commands/web)
- [Certificates (SSL)](/docs/commands/certificates)
- [Firewall](/docs/commands/firewall)
- [Troubleshooting](/docs/reference/troubleshooting)
