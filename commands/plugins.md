# Plugins

Abstrax supports extensibility through **standalone executable plugins**. Each plugin is a separate binary that Abstrax discovers and delegates commands to. Plugins are not Go shared libraries and do not use Go's native `plugin` package.

## How command delegation works

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

## Plugin management commands

```bash
abstrax plugin list
abstrax plugin info <name>
abstrax plugin search <query>
abstrax plugin install <name>
abstrax plugin update <name>
abstrax plugin remove <name>
```

### Example: install and run the example plugin

```bash
sudo abstrax plugin install example
abstrax example hello --name Mike
abstrax plugin info example
```

Example output from `abstrax plugin list`:

```text
NAME      VERSION  PUBLISHER   TRUST       STATUS  UPDATE
deploy    1.2.0    useabstrax  official *  active  -
example   0.1.0    useabstrax  official *  active  0.2.0
```

Official plugins are marked with `official *` in the trust column. The `publisher` field is a publisher slug.

### Direct manifest installation

Install from a manifest URL (not the official registry):

```bash
abstrax plugin install myplugin --manifest https://example.com/manifest.json
```

This shows a warning and requires confirmation unless `--yes` is supplied. Manifest-installed plugins are recorded with `source: manifest` and default to community trust when `trust_level` is omitted.

Example manifest:

```json
{
  "name": "example",
  "version": "0.1.0",
  "protocol_version": 1,
  "requires_abstrax": ">=0.1.0",
  "channel": "stable",
  "publisher": "useabstrax",
  "trust_level": "official",
  "display_name": "Example Plugin",
  "description": "Reference plugin and SDK example for Abstrax",
  "platforms": {
    "linux-amd64": {
      "url": "https://example.com/abstrax-example_0.1.0_linux_amd64.tar.gz",
      "sha256": "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
      "size": 123456
    }
  }
}
```

## Registry

The public registry website is [plugins.useabstrax.com](https://plugins.useabstrax.com). The API base URL is:

```text
https://plugins.useabstrax.com/api/v1
```

Override via `/etc/abstrax/config.json`:

```json
{
  "plugins": {
    "registry_url": "https://plugins.example.com/api/v1"
  }
}
```

Or the environment variable `ABSTRAX_PLUGIN_REGISTRY`.

Registry endpoints:

| Endpoint | Description |
|---|---|
| `GET /registry` | Registry metadata and supported platforms |
| `GET /plugins` | Paginated plugin list |
| `GET /plugins/{name}` | Plugin details |
| `GET /plugins/{name}/versions/latest` | Latest compatible version |
| `GET /plugins/{name}/versions/{version}` | Specific version |

The canonical API contract is defined in `openapi/plugin-registry.openapi.yml`. See [Plugin registry API](/docs/reference/plugin-registry-api) for request parameters, response shapes, and error codes.

The Abstrax CLI caches registry responses locally for one hour. Installed plugins continue to work when the registry is unavailable.

When installing from the registry, the CLI calls the latest-version endpoint with `abstrax_version`, `platform`, and `channel=stable` query parameters.

## Trust levels

| Level | Meaning |
|---|---|
| `official` | Published by Abstrax (marked clearly in CLI output) |
| `verified` | Verified third-party publisher |
| `community` | Community-published plugin |

Trust levels are display and policy metadata. They are not proof that a plugin is harmless.

## Plugin status

Registry records include a `status` field:

| Status | Behaviour |
|---|---|
| `active` | Normal operation |
| `deprecated` | Warning shown when running; installation allowed |
| `blocked` | Installation blocked; execution blocked unless `--allow-blocked-plugin` |

Blocked status is cached locally so enforcement does not require live registry access.

## Release channels

Version records include a `channel` field:

| Channel | Meaning |
|---|---|
| `stable` | Default production channel |
| `beta` | Pre-release channel |
| `alpha` | Early pre-release channel |

## Plugin metadata protocol

Every plugin must support:

```bash
abstrax-<name> plugin metadata
```

This must print JSON only to stdout (protocol version 1):

```json
{
  "protocol_version": 1,
  "name": "example",
  "display_name": "Example Plugin",
  "description": "Reference plugin and SDK example for Abstrax",
  "version": "0.1.0",
  "requires_abstrax": ">=0.1.0",
  "homepage": "https://plugins.useabstrax.com/plugins/example",
  "commands": [
    {
      "name": "hello",
      "description": "Print a greeting"
    },
    {
      "name": "project",
      "description": "Read project information from Abstrax"
    },
    {
      "name": "version",
      "description": "Display plugin version information"
    }
  ]
}
```

Required fields: `protocol_version`, `name`, `display_name`, `description`, `version`, `requires_abstrax`, and command names.

## Environment variables passed to plugins

When Abstrax launches a plugin, it sets:

| Variable | Value |
|---|---|
| `ABSTRAX_PLUGIN` | `1` |
| `ABSTRAX_PLUGIN_PROTOCOL` | `1` |
| `ABSTRAX_BINARY` | Absolute path to the current `abstrax` binary |
| `ABSTRAX_VERSION` | Current Abstrax semver |

## Machine-readable project commands

Plugins should call supported JSON commands rather than reading Abstrax internal storage files.

Inspect a project using the stable v1 API:

```bash
abstrax project inspect <project> --json
```

Example output:

```json
{
  "api_version": "v1",
  "project": {
    "name": "example",
    "path": "/var/www/example.com",
    "user": "example",
    "runtime": {
      "type": "php",
      "version": "8.5"
    },
    "domains": ["example.com"],
    "services": [
      {
        "name": "example-worker",
        "type": "worker"
      }
    ]
  }
}
```

This API exposes non-secret information only. The existing `abstrax project info --json` command remains available for the full internal project state envelope.

### Project service commands

Restart or reload project-owned supervisor services:

```bash
sudo abstrax project service restart <project> <service> --json
sudo abstrax project service reload <project> <service> --json
```

Services must belong to the project. These commands do not wrap arbitrary systemctl calls.

## Building and publishing a plugin

1. Create a standalone binary named `abstrax-<name>`.
2. Implement the `plugin metadata` subcommand returning protocol v1 JSON.
3. Implement your plugin commands.
4. Publish platform binaries (`linux-amd64`, `linux-arm64`) with SHA-256 checksums and a release manifest.

The full reference implementation, SDK, and publishing guide live in the `plugin-example/` repository:

```bash
cd plugin-example
make build
./bin/abstrax-example plugin metadata
./bin/abstrax-example hello --name Mike
```

A minimal embedded example also exists at `cli/cmd/abstrax-example/` for quick local testing.

## Security implications

Plugins are executable programs and may be dangerous:

- Built-in commands always override plugin names.
- Plugin binaries are verified with SHA-256 before installation.
- Plugins are never executed before checksum verification.
- The default registry uses HTTPS.
- Plugins are launched with `exec.CommandContext` and argument arrays (no shell).
- Registry inclusion is not a complete security audit.
- Plugins should not read private Abstrax storage formats directly.
- Plugins should use supported JSON commands and narrow service operations.
- Privileged operations are available only through validated core commands.

Use `--allow-blocked-plugin` only when you understand the risk.

## Related

- [Plugin registry API](/docs/reference/plugin-registry-api)
- [Configuration overview](/docs/configuration/)
- [Projects](/docs/commands/projects)
- [Security](/docs/reference/security)
- [Exit codes](/docs/reference/exit-codes)
