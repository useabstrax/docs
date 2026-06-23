# Creating a plugin

This page covers the end-to-end workflow. For individual topics, see [How plugins work](/docs/plugins/how-it-works) (naming and discovery), [Metadata protocol](/docs/plugins/metadata-protocol) (the `plugin metadata` command), and [Registry submission](/docs/plugins/registry-submission) (publishing to the official registry).

## What you will need

- A plugin binary compiled for supported platforms (`linux-amd64`, `linux-arm64`)
- A working `plugin metadata` command
- Semantic versioned releases with SHA-256 checksums
- A public source repository

## Reference implementation

The full reference implementation, SDK, and publishing guide live in the `plugin-example/` directory:

```bash
cd plugin-example
make build
./bin/abstrax-example plugin metadata
./bin/abstrax-example hello --name Mike
```

See `plugin-example/CREATING_A_PLUGIN.md` for a step-by-step guide to forking the example into a new plugin.

A minimal embedded example also exists at `cli/cmd/abstrax-example/` in the main Abstrax repository.

## Local testing without the registry

Build the embedded example from the `cli/` directory:

```bash
go build -o abstrax-example ./cmd/abstrax-example
./abstrax-example plugin metadata
```

Place the binary on your `PATH` or in a plugin directory to test delegation through the main `abstrax` binary.

## Publishing to the registry

To submit a plugin to the official registry, open a pull request in the [useabstrax/plugin-registry](https://github.com/useabstrax/plugin-registry) repository. See [Registry submission](/docs/plugins/registry-submission) for the full process, required metadata, and review expectations. Release manifest format and checksum requirements are documented in [Registry API](/docs/plugins/registry-api#release-manifest-format).

## Related

- [Metadata protocol](/docs/plugins/metadata-protocol)
- [Integrating with Abstrax](/docs/plugins/integrating-with-abstrax)
- [Registry submission](/docs/plugins/registry-submission)
- [Security](/docs/plugins/security)
