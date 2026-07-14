# Web server

The `web` group installs and manages the web server. The default backend is nginx.

```text
abstrax web <action> [flags]
```

Most subcommands accept a `--nginx` flag for selecting the backend. nginx is the only supported backend and is used by default, so you do not normally need to set it.

## Permissions

`web install`, `web reload`, and `web restart` require root. `web test` does not require root.

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

After installing nginx, use [Projects](/docs/commands/projects) to add virtual hosts for your applications. On Debian-family systems Abstrax writes site configs to `/etc/nginx/sites-available` and enables them via symlinks in `/etc/nginx/sites-enabled`. On RHEL-family systems configs are written to `/etc/nginx/conf.d/{site}.conf`.


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
