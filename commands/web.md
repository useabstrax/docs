# Web server

The `web` group installs and manages the web server. The default backend is nginx.

```text
abstrax web <action> [flags]
```

Most subcommands accept a `--nginx` flag for selecting the backend. nginx is the only supported backend and is used by default, so you do not normally need to set it.

## Permissions

`web install`, `web ip set`, `web ip clear`, `web reload`, and `web restart` require root. `web test` and `web ip show` do not require root.

## `web install`

Install and configure a web server for hosting standard websites. On Debian-family systems this installs nginx via `apt`, sets up the `sites-available` / `sites-enabled` layout, ensures `nginx.conf` includes enabled sites, and disables the default welcome site. On RHEL-family systems this installs nginx via `dnf` and uses `/etc/nginx/conf.d` (no sites-available/enabled symlinks). The configuration is validated and the service is enabled at boot.


```bash
sudo abstrax web install
```

| Flag | Default | Description |
|---|---|---|
| `--apache` | `false` | Install Apache instead of nginx (not yet implemented) |
| `--enable` | `true` | Enable the service at boot |
| `--start` | `true` | Start the service after install |

```bash
sudo abstrax web install
sudo abstrax web install --no-start
```

After installing nginx, use [Projects](/docs/commands/projects) with `--nginx` to add virtual hosts for your applications. On Debian-family systems Abstrax writes site configs to `/etc/nginx/sites-available` and enables them via symlinks in `/etc/nginx/sites-enabled`. On RHEL-family systems configs are written to `/etc/nginx/conf.d/{site}.conf`.

`web install` does **not** create a managed response for the bare server IP. Until you set one with [`web ip`](#web-ip) or create a domainless `--nginx` project, a single named site may still answer on the IP as nginx's implicit default.


## `web ip`

Control what `http://<server-ip>/` and `https://<server-ip>/` return when no project holds `default_server`. Registered as `web.ip.show`, `web.ip.set`, and `web.ip.clear`.

A project added with `project add --nginx` and no domains owns `default_server` instead (see [Projects](/docs/commands/projects#default-server--server-ip)). While that project holds the IP, the `web ip` block keeps its return code but drops `default_server`. When the project gains a domain or is removed, the IP block takes `default_server` again if a response is still configured.

### `web ip show`

Show the configured response. Does not require root.

```bash
abstrax web ip show
```

Unset is the default. The note explains whether a named site may still answer the IP as nginx's implicit default.

### `web ip set`

Write an Abstrax-managed nginx block (`abstrax-ip.conf`, not named `abstrax-<project>` so `project remove` does not delete it).

```bash
sudo abstrax web ip set --return 404
sudo abstrax web ip set --return 403
sudo abstrax web ip set --return 444
```

| Flag | Description |
|---|---|
| `--return` | Required. `404`, `403`, or `444` (`444` closes the connection with no response) |

The HTTP server listens on port 80 (and `[::]:80`) with `return <code>`. The HTTPS server listens on 443 with `ssl_reject_handshake on`, so HTTPS to the IP does not fall through to a named site and does not need a certificate.

### `web ip clear`

Remove the managed block and reload nginx.

```bash
sudo abstrax web ip clear
```

After clear, remaining named sites may answer the IP again as nginx's implicit default.


## `web test`

Test the web server configuration (runs `nginx -t`). Does not require root.

```bash
abstrax web test
```

```text
nginx configuration test passed.
```

If the test fails, Abstrax prints the error output and the JSON result has `error_code: config_invalid`.

## `web reload` and `web restart`

```bash
sudo abstrax web reload
sudo abstrax web restart
```

`reload` applies configuration changes gracefully and is usually what you want. `restart` fully restarts the web server, which briefly drops active connections. Run `abstrax web test` first to confirm the configuration is valid before reloading or restarting.

## Notes

- These commands manage nginx. Apache is referenced in the flags but is not yet implemented.
- After changing a virtual host by hand, run `abstrax web test` before reloading to catch configuration errors.

## Related

- [Projects](/docs/commands/projects)
- [Certificates (SSL)](/docs/commands/certificates)
