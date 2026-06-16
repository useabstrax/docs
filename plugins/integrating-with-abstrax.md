# Integrating with Abstrax

Plugins should call supported JSON commands rather than reading Abstrax internal storage files.

## Environment variables

When Abstrax launches a plugin, it sets:

| Variable | Value |
|---|---|
| `ABSTRAX_PLUGIN` | `1` |
| `ABSTRAX_PLUGIN_PROTOCOL` | `1` |
| `ABSTRAX_BINARY` | Absolute path to the current `abstrax` binary |
| `ABSTRAX_VERSION` | Current Abstrax semver |

These are set on the plugin process only. They are not used to configure Abstrax itself. See [Environment variables](/docs/configuration/environment-variables#plugin-environment-variables) for registry configuration variables.

## Project inspect API

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

## Project service commands

Restart or reload project-owned supervisor services:

```bash
sudo abstrax project service restart <project> <service> --json
sudo abstrax project service reload <project> <service> --json
```

Services must belong to the project. These commands do not wrap arbitrary systemctl calls.

## Related

- [Projects command reference](/docs/commands/projects)
- [Creating a plugin](/docs/plugins/creating-a-plugin)
- [Security](/docs/plugins/security)
