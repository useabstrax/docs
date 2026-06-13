# Config

The `config` group manages Abstrax settings stored in `/etc/abstrax/config.json`. These settings apply server-wide and are used by other commands (for example, PHP extension packages installed when creating a PHP project).

```text
abstrax config <action> [arguments] [flags]
```

## Permissions

`set`, `add`, `remove`, and `reset` require root. The read-only commands `show`, `list`, and `get` do not require root.

## Configuration file

Settings are stored as JSON:

```text
/etc/abstrax/config.json
```

If the file does not exist, Abstrax uses built-in defaults. See [Config file](/docs/configuration/config-file) for details.

## Keys

Keys use dot notation. Only known keys are accepted.

| Key | Type | Default |
|---|---|---|
| `php.extensions` | list of strings | `mysql`, `xml`, `curl`, `mbstring`, `zip`, `bcmath`, `gd` |

Values for `php.extensions` are apt package **suffixes**, not full package names. When PHP 8.5 is installed for a project, Abstrax expands `mysql` to `php8.5-mysql`, and so on.

## `config show`

Show the effective configuration (defaults merged with saved settings).

```bash
abstrax config show
```

`config list` is an alias for `config show`.

```text
  php.extensions:
    - mysql
    - xml
    - curl
    - mbstring
    - zip
    - bcmath
    - gd
```

## `config get`

Get a single configuration value.

```bash
abstrax config get php.extensions
```

```text
mysql xml curl mbstring zip bcmath gd
```

## `config set`

Replace a list configuration value.

```bash
sudo abstrax config set php.extensions mysql xml curl mbstring zip intl
```

## `config add`

Append a value to a list configuration key.

```bash
sudo abstrax config add php.extensions redis
```

## `config remove`

Remove a value from a list configuration key.

```bash
sudo abstrax config remove php.extensions gd
```

## `config reset`

Reset configuration to defaults. Asks for confirmation unless `--yes` is given.

```bash
# Reset one key
sudo abstrax config reset php.extensions

# Reset all settings (removes the config file)
sudo abstrax config reset
```

## Related

- [Config file](/docs/configuration/config-file)
- [Projects](/docs/commands/projects)
