# Registry submission

For the initial registry release, plugin submissions are reviewed manually through a GitHub pull request against `resources/plugin-registry/plugins/` in the Abstrax site repository.

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

## Review expectations

Submission does not guarantee acceptance. Registry inclusion is not a full security audit. See [Security](/docs/plugins/security).

## After approval

Approved YAML definitions are imported with:

```bash
php artisan plugins:sync
```

## Related

- [Creating a plugin](/docs/plugins/creating-a-plugin)
- [Metadata protocol](/docs/plugins/metadata-protocol)
- [Registry API](/docs/plugins/registry-api#release-manifest-format)
