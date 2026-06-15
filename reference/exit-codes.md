# Exit codes and output

This page describes the exit codes Abstrax returns, the structure of its JSON output, and the error codes it uses.

## Exit codes

Abstrax uses two exit codes:

| Code | Meaning |
|---|---|
| `0` | The command completed successfully |
| `1` | The command failed for any reason |

There is no finer-grained set of exit codes for most commands. Any error - invalid input, missing permissions, a failed underlying command, an unsupported platform - results in exit code `1`. To distinguish between causes in a script, read the error message, or use `--json` and inspect the `error_code` and `message` fields.

**Exception:** When Abstrax delegates to a plugin command (for example `abstrax deploy`), the plugin process exit code is propagated to the shell. A plugin that exits with code `2` will cause Abstrax to exit with `2`.

When a command requires confirmation and you decline the prompt, the command stops without making changes and exits `0`.

## JSON result shape

Add `--json` to any command for structured output. Successful results have this shape:

```json
{
  "status": "success",
  "action": "user.add",
  "summary": "User deploy created.",
  "data": { "...": "..." }
}
```

| Field | Description |
|---|---|
| `status` | `success` or `error` |
| `action` | The stable action name, for example `user.add` |
| `summary` | A short human-readable summary (success only) |
| `data` | Command-specific data (optional) |

Failure results have this shape:

```json
{
  "status": "error",
  "action": "user.add",
  "error_code": "command_error",
  "message": "user deploy already exists"
}
```

| Field | Description |
|---|---|
| `status` | `error` |
| `action` | The action name, which may be empty for top-level errors |
| `error_code` | A short error category |
| `message` | The error detail |

## Error codes

The error codes currently produced by the application are:

| Error code | When it appears |
|---|---|
| `command_error` | The default code for any error surfaced at the top level (validation failures, permission errors, failed underlying commands) |
| `config_invalid` | Returned by `web test` when the web server configuration test fails |
| `plugin_not_installed` | Plugin binary or installation record not found |
| `registry_unavailable` | Plugin registry could not be reached |
| `incompatible_abstrax_version` | Plugin requires a different Abstrax version |
| `unsupported_platform` | No plugin binary for the current platform |
| `checksum_mismatch` | Downloaded plugin binary failed SHA-256 verification |
| `malformed_plugin_metadata` | Plugin metadata JSON is invalid |
| `unsupported_plugin_protocol` | Plugin uses an unsupported metadata protocol version |
| `blocked_plugin` | Plugin is blocked by registry policy |
| `plugin_process_failure` | Plugin process failed to start |

Most failures use `command_error`. The `message` field carries the specific detail.

## Text output conventions

Without `--json`, Abstrax prints human-readable text:

- Success lines are printed to standard output (green, unless `--no-color`).
- Warnings are printed to standard error, prefixed with `WARNING:` (yellow).
- Errors are printed to standard error, prefixed with `ERROR:` (red).
- `--verbose` adds `[verbose]` lines and prints the underlying commands being run.
- `--dry-run` prints `[dry-run] would run: ...` lines instead of running commands.
- `--quiet` suppresses informational messages.

## Related

- [Troubleshooting](/docs/reference/troubleshooting)
- [Command reference](/docs/reference/index)
