# Plugin security

Plugins are executable programs and may be dangerous. Treat third-party plugins like any binary you install on your server.

## Built-in protections

- Built-in commands always override plugin names.
- Plugin binaries are verified with SHA-256 before installation.
- Plugins are never executed before checksum verification.
- The default registry uses HTTPS.
- Plugins are launched with `exec.CommandContext` and argument arrays (no shell).
- Registry inclusion is not a complete security audit.
- Project configuration files cannot trigger automatic plugin installation.

## What plugin authors should do

- Do not read private Abstrax storage formats directly.
- Use supported JSON commands and narrow service operations (see [Integrating with Abstrax](/docs/plugins/integrating-with-abstrax)).
- Privileged operations are available only through validated core commands.

## Trust levels and registry policies

Trust levels (`official`, `verified`, `community`) and plugin status values (`active`, `deprecated`, `blocked`) are policy metadata, not proof that a plugin is safe. See [Registry](/docs/plugins/registry) for what each value means.

The registry distributes metadata only. It does not execute plugin code on your server. Production release URLs must use HTTPS, and every binary must include a SHA-256 checksum verified by the CLI before installation.

## Blocked plugins

Blocked plugins cannot be installed through the registry. On the server, execution is blocked unless you pass `--allow-blocked-plugin` (repeatable global flag) or list the plugin in `allow_blocked` in `/etc/abstrax/config.json`.

If a plugin is compromised, registry operators set `status: blocked` in its YAML definition in the [plugin-registry repository](https://github.com/useabstrax/plugin-registry). Once that change is merged, the live registry is updated and installation is blocked for new users.

Use `--allow-blocked-plugin` only when you understand the risk.

## Related

- [Registry](/docs/plugins/registry)
- [How plugins work](/docs/plugins/how-it-works)
- [Security](/docs/reference/security#plugins)
