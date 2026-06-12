# Web server

The `web` group operates on the web server directly. The current backend is nginx.

```text
abstrax web <action>
```

The group accepts a `--nginx` flag for selecting the backend. nginx is the only supported backend and is used by default, so you do not normally need to set it.

## Permissions

`web test` does not require root. `web reload` and `web restart` require root.

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
