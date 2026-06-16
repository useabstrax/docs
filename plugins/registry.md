# Registry

The public plugin registry is hosted at [plugins.useabstrax.com](https://plugins.useabstrax.com). Browse plugins there, or install them with `abstrax plugin install`.

For HTTP endpoints, request parameters, response shapes, error codes, and release manifest format, see [Registry API](/docs/plugins/registry-api).

## CLI configuration

Override the registry API base URL in `/etc/abstrax/config.json`:

```json
{
  "plugins": {
    "registry_url": "https://plugins.example.com/api/v1"
  }
}
```

Or set the `ABSTRAX_PLUGIN_REGISTRY` environment variable. See [Config file](/docs/configuration/config-file#plugin-settings) for details.

## How the CLI uses the registry

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

## Related

- [Plugin commands](/docs/commands/plugins)
- [Registry API](/docs/plugins/registry-api)
- [Security](/docs/plugins/security)
