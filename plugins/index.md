# Plugins

Abstrax supports extensibility through **standalone executable plugins**. Each plugin is a separate binary that Abstrax discovers and delegates commands to. Plugins are not Go shared libraries and do not use Go's native `plugin` package.

Use the `abstrax plugin` command group to install, update, and remove plugins from the registry. See [Plugin commands](/docs/commands/plugins) for the full command reference.

## For users

| Page | What it covers |
|---|---|
| [How plugins work](/docs/plugins/how-it-works) | Command delegation, naming, and where binaries are found |
| [Registry](/docs/plugins/registry) | The public registry, trust levels, status, and release channels |
| [Security](/docs/plugins/security) | What to know before installing, running, or publishing a plugin |

## For plugin authors

| Page | What it covers |
|---|---|
| [Metadata protocol](/docs/plugins/metadata-protocol) | The `plugin metadata` subcommand and required JSON fields |
| [Integrating with Abstrax](/docs/plugins/integrating-with-abstrax) | Environment variables and stable JSON APIs for project data |
| [Creating a plugin](/docs/plugins/creating-a-plugin) | Build, test, and publish a plugin |
| [Registry submission](/docs/plugins/registry-submission) | Submit a plugin to the official registry |
| [Registry API](/docs/plugins/registry-api) | HTTP endpoints, request parameters, and response shapes |

## Quick example

```bash
sudo abstrax plugin install example
abstrax example hello --name Mike
abstrax plugin info example
```

The reference implementation lives in the `plugin-example/` directory in the repository. A minimal embedded example also exists at `cli/cmd/abstrax-example/` for local testing.
