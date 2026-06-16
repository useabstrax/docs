# Metadata protocol

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

## Required fields

| Field | Description |
|---|---|
| `protocol_version` | Must be `1` |
| `name` | Plugin name (matches the `abstrax-<name>` binary suffix) |
| `display_name` | Human-readable name |
| `description` | Short description |
| `version` | Plugin semver |
| `requires_abstrax` | Semver constraint for the Abstrax CLI |
| `commands` | List of subcommands, each with `name` and `description` |

Abstrax reads this metadata after installation and when you run `abstrax plugin info`.

## Semantic versioning

Plugin versions must use valid semantic versions. Abstrax compatibility is expressed as a semver constraint in `requires_abstrax` (for example `>=0.1.0`).

The plugin `name` must match the `abstrax-<name>` binary suffix. See [How plugins work](/docs/plugins/how-it-works#plugin-naming-convention).

## Related

- [Creating a plugin](/docs/plugins/creating-a-plugin)
- [Registry submission](/docs/plugins/registry-submission)
- [Registry API](/docs/plugins/registry-api#release-manifest-format)
