# Official plugins

Official plugins are maintained by the Abstrax team. They use trust level `official` in the [plugin registry](/docs/plugins/registry), ship as separate binaries, and extend the CLI with top-level commands (for example `abstrax deploy`).

Install them with the same workflow as any other plugin:

```bash
sudo abstrax plugin install <name>
abstrax <name> version
```

Official does **not** mean “runs with elevated privileges by default.” Mutating commands still follow the same root and confirmation patterns as core Abstrax commands. Treat every plugin binary like software you chose to install on the server. See [Plugin security](/docs/plugins/security).

## Available official plugins

| Plugin | Command | What it does |
|---|---|---|
| [Deploy](/docs/plugins/official/deploy) | `abstrax deploy` | Zero-downtime GitHub deployments for Abstrax projects |

More official plugins may appear here as they are published.

## Related

- [Plugins overview](/docs/plugins/)
- [Plugin commands](/docs/commands/plugins)
- [Registry](/docs/plugins/registry)
- [How plugins work](/docs/plugins/how-it-works)
