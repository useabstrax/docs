# Plugins

The `plugin` command group installs, updates, and removes plugins from the Abstrax registry. Installed plugins extend the CLI with additional top-level commands (for example `abstrax deploy`).

For how plugins work, the registry, and plugin development, see the [Plugins](/docs/plugins/) section.

```text
abstrax plugin <action> [arguments] [flags]
```

## Permissions

| Command | Root required |
|---|---|
| `plugin list` | No |
| `plugin info` | No |
| `plugin search` | No |
| `plugin install` | Yes (registry installs; `--manifest` and `--path` do not enforce root) |
| `plugin update` | Yes |
| `plugin remove` | Yes |

Registry installs write to `/usr/local/lib/abstrax/plugins/` when run as root. Installation records and caches are stored under `/var/lib/abstrax/plugins/`.

## `plugin list`

List installed plugins.

```bash
abstrax plugin list
abstrax plugin list --json
```

Example output:

```text
NAME      VERSION  PUBLISHER   TRUST       STATUS  UPDATE
deploy    1.2.0    useabstrax  official *  active  -
example   0.1.0    useabstrax  official *  active  0.2.0
```

Official plugins are marked with `official *` in the trust column.

## `plugin info`

Show detailed information for an installed plugin, including the subcommands it provides.

```bash
abstrax plugin info <name>
abstrax plugin info example --json
```

## `plugin search`

Search the plugin registry. Does not require root.

```bash
abstrax plugin search <query>
abstrax plugin search deploy --json
```

## `plugin install`

Install a plugin from the registry, a direct manifest URL, or a local binary.

```bash
sudo abstrax plugin install <name>
sudo abstrax plugin install example
abstrax plugin install example --json
```

| Flag | Description |
|---|---|
| `--manifest` | Install from a direct manifest JSON URL instead of the registry |
| `--path` | Install from a local plugin binary path (creates a symlink; does not copy) |

`--manifest` and `--path` are mutually exclusive.

### Direct manifest installation

```bash
abstrax plugin install myplugin --manifest https://example.com/manifest.json
```

This shows a warning and requires confirmation unless `--yes` is supplied. Manifest-installed plugins are recorded with `source: manifest` and default to community trust when `trust_level` is omitted.

See [Release manifest format](/docs/plugins/registry-api#release-manifest-format) for the expected JSON shape.

### Local binary installation

Install a plugin that is not in the registry by pointing at an existing binary on the server:

```bash
abstrax plugin install myplugin --path /opt/custom/abstrax-myplugin
abstrax plugin install /opt/custom/abstrax-myplugin
```

When the positional argument looks like a filesystem path, Abstrax treats it as the binary path and reads the plugin name from `plugin metadata`.

Local installs:

- Print a warning that the plugin is not official or verified from the registry, and require confirmation (skipped with `--yes`)
- Create a symlink in the plugin install directory to the given binary (the original file is never copied)
- Record `source: local` and community trust
- Do not require root (same as `--manifest`); root still writes under `/usr/local/lib/abstrax/plugins/`

`plugin update` cannot update local plugins; reinstall with `--path` instead.

## `plugin update`

Update an installed plugin to the latest compatible version from the registry.

```bash
sudo abstrax plugin update <name>
sudo abstrax plugin update example
```

## `plugin remove`

Remove an installed plugin. Prompts for confirmation unless `--yes` is supplied.

```bash
sudo abstrax plugin remove <name>
sudo abstrax plugin remove example
```

For registry and manifest installs, this deletes the installed binary and the installation record. For local (`--path`) installs, only the symlink in the plugin install directory and the installation record are removed; the original binary on disk is left intact.
## Running installed plugins

After installation, a plugin's commands are available as top-level Abstrax commands:

```bash
sudo abstrax plugin install example
abstrax example hello --name Mike
abstrax deploy production
```

Built-in commands always take priority over plugin names. See [How plugins work](/docs/plugins/how-it-works) for discovery order and delegation behaviour.

## Related

- [Plugins overview](/docs/plugins/)
- [Registry](/docs/plugins/registry)
- [Configuration overview](/docs/configuration/)
- [Projects](/docs/commands/projects)
