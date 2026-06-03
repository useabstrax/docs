# Creating a project

- [Creating a user](/docs/guides/creating-a-user)
- [Adding SSH access](/docs/guides/adding-ssh-access)
- [Installing packages](/docs/guides/installing-packages)
- [Creating a project](/docs/guides/creating-a-project)
- [Managing cron jobs](/docs/guides/managing-cron-jobs)
- [Managing daemons](/docs/guides/managing-daemons)

This guide creates an nginx-backed web project, then adds a TLS certificate. It uses the [`project`](/docs/commands/projects), [`web`](/docs/commands/projects#web-server-commands), and [`ssl`](/docs/commands/certificates) commands.

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
  --php --php-version=8.2 --public-dir=public
```

### Node.js or Ruby application (reverse proxy)

```bash
sudo abstrax project add myapp \
  --path=/var/www/myapp \
  --domains=myapp.com \
  --node --proxy-port=3000
```

Expected output:

```text
Project myapp created.
  Path:       /var/www/myapp
  Web server: nginx
  Domains:    myapp.com, www.myapp.com
  Vhost:      /etc/nginx/sites-available/abstrax-myapp
```

If you omit `--path`, it defaults to `/var/www/<name>`.

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

- [Projects and web server](/docs/commands/projects)
- [Certificates (SSL)](/docs/commands/certificates)
- [Firewall](/docs/commands/system#firewall-commands)
