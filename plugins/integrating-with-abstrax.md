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
    "web_server": "nginx",
    "runtime": {
      "type": "php",
      "version": "8.5"
    },
    "public_dir": "public",
    "domains": ["example.com"],
    "vhost": {
      "present": true,
      "path": "/etc/nginx/sites-available/abstrax-example",
      "enabled": true
    },
    "shared_logs": false,
    "access_log_path": "/var/log/nginx/example-access.log",
    "error_log_path": "/var/log/nginx/example-error.log",
    "ssl": {
      "enabled": true
    },
    "services": [
      {
        "name": "example-worker",
        "type": "worker"
      }
    ],
    "created_at": "2026-11-01T10:00:00Z",
    "updated_at": "2026-11-02T12:30:00Z"
  }
}
```

This API exposes non-secret information only. New fields are additive within `api_version: v1` (see [Projects](/docs/commands/projects#project-inspect)). The existing `abstrax project info --json` command remains available for the full internal project state envelope.

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
