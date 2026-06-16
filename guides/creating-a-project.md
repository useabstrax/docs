# Creating a project

This guide creates an nginx-backed web project, then adds a TLS certificate. It uses the [`project`](/docs/commands/projects), [`web`](/docs/commands/web), and [`ssl`](/docs/commands/certificates) commands.

These commands change system state and require root. Make sure nginx is installed first (`abstrax doctor` will show you).

## 1. Create the project

Pick the example that matches your application.

### Static site

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

If PHP is not installed yet, Abstrax asks whether to install the requested version before continuing. Answer yes to install it, or no to abort. Pass `--yes` to install without prompting. PHP is installed with FPM, CLI, and the extension packages configured in [`abstrax config`](/docs/commands/config) (defaults include `mysql`, `xml`, `curl`, `mbstring`, `zip`, `bcmath`, and `gd`).

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
  Web server: nginx
  Domains:    myapp.com, www.myapp.com
  Vhost:      /etc/nginx/sites-available/abstrax-myapp
```

If you omit `--path`, it defaults to `/var/www/<name>`.

### Ownership

By default, `--user` and `--group` are both `www-data`. Abstrax records the intended owner in project state (see `project info`); set on-disk ownership to match your deployment workflow. To record a different owner when adding the project:

```bash
sudo abstrax project add myapp \
  --path=/var/www/myapp \
  --user=deploy --group=www-data \
  --domains=myapp.com --php --public-dir=public
```

If you deploy over SSH as a dedicated user, see [Creating a user](/docs/guides/creating-a-user). You can place the project anywhere with `--path` (for example `/home/deploy/myapp`).

### Deploy your application code

`project add` creates the directory and nginx configuration but does not put your application files on the server. Deploy source code separately — for example with CI/CD, `git clone`/`git pull`, rsync, or your existing deployment process — once the project path and virtual host are in place.

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

To also delete the files:

```bash
sudo abstrax project remove myapp --delete-files --force
```

## Notes

- The current web server backend is nginx. Apache is referenced in the flags but is not yet implemented.
- Make sure the firewall allows HTTP and HTTPS: `sudo abstrax firewall allow 80` and `sudo abstrax firewall allow 443 --protocol=tcp`.

## Related

- [Creating a user](/docs/guides/creating-a-user)
- [Projects](/docs/commands/projects)
- [Web server](/docs/commands/web)
- [Certificates (SSL)](/docs/commands/certificates)
- [Firewall](/docs/commands/firewall)
