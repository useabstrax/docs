# Registry submission

To list a plugin in the [Abstrax plugin registry](https://plugins.useabstrax.com), open a pull request in the [useabstrax/plugin-registry](https://github.com/useabstrax/plugin-registry) repository.

Each plugin is a single YAML file under the repository's `plugins/` directory. The repository README, `examples/`, and `schema.json` describe the expected format.

## How to submit

1. Fork [useabstrax/plugin-registry](https://github.com/useabstrax/plugin-registry).
2. Copy one of the files from `examples/` as a starting point.
3. Create `plugins/<slug>.yml` with your plugin metadata and version records.
4. Verify your plugin locally:
   ```bash
   abstrax-<name> plugin metadata
   ```
5. Open a pull request against `main`.

Put your YAML in `plugins/`, not `examples/`. The `examples/` directory is reference material only.

## Required information

Your submission should include:

- Plugin name and display name
- Short and long description
- Publisher details and public source repository
- Licence and documentation URL
- Release manifest URL
- Supported platforms and semantic version
- Required Abstrax version constraint
- SHA-256 checksums for each binary
- Security contact
- Confirmation that `plugin metadata` works

Release manifest format and checksum requirements are documented in [Registry API](/docs/plugins/registry-api#release-manifest-format).

## Review expectations

Submissions are reviewed manually. Acceptance is not guaranteed.

Registry inclusion is not a full security audit. Trust levels (`official`, `verified`, `community`) and status values (`active`, `deprecated`, `blocked`) are policy metadata, not proof that a plugin is safe. See [Security](/docs/plugins/security).

## After approval

When your pull request is merged, Abstrax imports your plugin definition into the live registry. The plugin then appears at `https://plugins.useabstrax.com/plugins/<slug>` and can be installed with:

```bash
sudo abstrax plugin install <name>
```

To publish a new version later, open another pull request that adds or updates the relevant entry under `versions:` in your plugin file.

## Related

- [Creating a plugin](/docs/plugins/creating-a-plugin)
- [Metadata protocol](/docs/plugins/metadata-protocol)
- [Registry API](/docs/plugins/registry-api#release-manifest-format)
- [plugin-registry repository](https://github.com/useabstrax/plugin-registry)
