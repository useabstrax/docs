# Plugin registry API

The Abstrax plugin registry is hosted at [plugins.useabstrax.com](https://plugins.useabstrax.com). The public read-only API base URL is:

```text
https://plugins.useabstrax.com/api/v1
```

API versioning is expressed in the URL path. Responses do not include a redundant top-level `api_version` field.

## OpenAPI specification

The canonical contract is defined in the repository at:

```text
openapi/plugin-registry.openapi.yml
```

Validate the specification locally:

```bash
./openapi/validate.sh
```

Shared JSON examples used by contract tests live in:

```text
openapi/fixtures/
```

## Endpoints

| Method | Path | Description |
|---|---|---|
| `GET` | `/registry` | Registry metadata and supported platforms |
| `GET` | `/plugins` | Paginated plugin list |
| `GET` | `/plugins/{plugin}` | Plugin detail |
| `GET` | `/plugins/{plugin}/versions/latest` | Latest compatible version |
| `GET` | `/plugins/{plugin}/versions/{version}` | Specific version |

### Registry metadata

```bash
curl https://plugins.useabstrax.com/api/v1/registry
```

```json
{
  "registry": {
    "name": "Abstrax Plugin Registry",
    "supported_plugin_protocols": [1],
    "supported_platforms": ["linux-amd64", "linux-arm64"]
  }
}
```

### Plugin list

Query parameters:

| Parameter | Description |
|---|---|
| `search` | Case-insensitive search |
| `trust_level` | `official`, `verified`, or `community` |
| `status` | `active`, `deprecated`, `blocked`, or `all` (default `active`) |
| `page` | Page number (default `1`) |
| `per_page` | Page size (default `20`, max `100`) |

```json
{
  "plugins": [
    {
      "name": "example",
      "display_name": "Example Plugin",
      "description": "Reference plugin and SDK example for Abstrax",
      "publisher": "useabstrax",
      "trust_level": "official",
      "status": "active",
      "latest_version": "0.1.0",
      "homepage": "https://plugins.useabstrax.com/plugins/example"
    }
  ],
  "meta": {
    "current_page": 1,
    "last_page": 1,
    "per_page": 20,
    "total": 1
  }
}
```

The `publisher` field is a publisher slug, not a display name.

### Latest version

Query parameters:

| Parameter | Description |
|---|---|
| `abstrax_version` | Client Abstrax version for compatibility resolution |
| `platform` | Required platform, for example `linux-amd64` |
| `channel` | Release channel (default `stable`) |

```json
{
  "version": "0.1.0",
  "requires_abstrax": ">=0.1.0",
  "platforms": {
    "linux-amd64": {
      "url": "https://example.com/abstrax-example_0.1.0_linux_amd64.tar.gz",
      "sha256": "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
      "size": 123456
    }
  },
  "stable": true,
  "channel": "stable",
  "protocol_version": 1,
  "release_date": "2026-06-15T12:00:00+00:00",
  "manifest_url": "https://example.com/plugin-manifest.json"
}
```

### Error responses

All registry API errors use this structure:

```json
{
  "error": {
    "code": "plugin_not_found",
    "message": "The requested plugin could not be found."
  }
}
```

Stable error codes:

| Code | HTTP status | Meaning |
|---|---|---|
| `plugin_not_found` | 404 | Plugin does not exist |
| `version_not_found` | 404 | Version does not exist |
| `no_compatible_version` | 404 | No version matches constraints |
| `unsupported_platform` | 404 | No binary for requested platform |
| `plugin_blocked` | 403 | Plugin is blocked |
| `invalid_request` | 422 | Invalid query parameters |
| `registry_error` | 5xx | Internal registry failure |

## Release manifest format

Direct installation with `abstrax plugin install <name> --manifest=<url>` uses the same core fields as the version endpoint:

```json
{
  "name": "example",
  "version": "0.1.0",
  "protocol_version": 1,
  "requires_abstrax": ">=0.1.0",
  "channel": "stable",
  "publisher": "useabstrax",
  "trust_level": "official",
  "description": "Reference plugin and SDK example for Abstrax",
  "display_name": "Example Plugin",
  "platforms": {
    "linux-amd64": {
      "url": "https://example.com/abstrax-example_0.1.0_linux_amd64.tar.gz",
      "sha256": "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
      "size": 123456
    }
  }
}
```

## Related

- [Plugins command reference](/docs/commands/plugins)
- [Security](/docs/reference/security)
