# How plugins work

When you run:

```bash
abstrax <command> [arguments]
```

Abstrax:

1. Checks whether `<command>` is a built-in command (built-ins always win).
2. If not, searches for an executable named `abstrax-<command>`.
3. If found, runs the plugin with remaining arguments unchanged.
4. Attaches stdin, stdout, and stderr directly to the plugin process.
5. Passes through the current environment plus Abstrax-specific variables.
6. Returns the plugin's exit code to the shell.

Example:

```bash
abstrax deploy production
abstrax example hello --name Mike
```

## Plugin naming convention

Plugin binaries must be named:

```text
abstrax-<plugin-name>
```

Examples:

```text
abstrax-deploy
abstrax-backup
abstrax-example
```

The plugin name (used on the command line) is the part after `abstrax-`.

## Plugin directories

Abstrax searches for plugins in this order (first match wins):

| Order | Directory |
|---|---|
| 1 | Plugin install directory (where `abstrax plugin install` places binaries) |
| 2 | `/usr/local/lib/abstrax/plugins/` |
| 3 | `/usr/lib/abstrax/plugins/` |
| 4 | `~/.local/share/abstrax/plugins/` (non-root only) |
| 5 | Each directory in `PATH` (looks for `abstrax-<name>` only) |

Abstrax never searches the current working directory.

When running as root, `abstrax plugin install` installs to `/usr/local/lib/abstrax/plugins/`. Installation records and caches are stored under `/var/lib/abstrax/plugins/`.

## Related

- [Plugin commands](/docs/commands/plugins)
- [Registry](/docs/plugins/registry)
- [Integrating with Abstrax](/docs/plugins/integrating-with-abstrax)
