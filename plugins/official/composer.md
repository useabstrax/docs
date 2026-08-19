# Composer plugin

Installs Composer on the server and runs it with the correct PHP binary. Abstrax already installs PHP-FPM and PHP CLI for PHP projects; this plugin installs **Composer itself**. It is optional: static, Node.js, and Ruby servers do not need it.

| Item | Value |
|---|---|
| Binary | `abstrax-composer` |
| CLI | `abstrax composer …` |
| Trust level | `official` |
| Platforms | Debian/Ubuntu and RHEL-family (Rocky/Alma/RHEL) |

## Install

```bash
sudo abstrax plugin install composer
abstrax composer version
sudo abstrax composer setup
```

## What it installs

`composer setup` downloads the latest stable Composer phar from getcomposer.org, verifies the SHA-256 checksum, and writes:

| Path | Role |
|---|---|
| `/usr/local/lib/abstrax/composer/composer.phar` | The Composer phar |
| `/usr/local/bin/composer` | Wrapper that runs the phar with the configured default PHP |
| `/etc/abstrax/composer.json` | Optional default PHP binary |

The wrapper is marked as managed by Abstrax. `setup` will not overwrite a Composer binary it did not install unless you pass `--force`.

This plugin does not wrap every Composer subcommand. Pass Composer arguments through `abstrax composer run`.

## Quick start

```bash
sudo abstrax composer setup
abstrax composer status

sudo abstrax composer configure --php=php8.2

abstrax composer run install --no-dev --optimize-autoloader
abstrax composer run --project=example.com install --no-dev
```

## PHP resolution

Composer is always invoked as `php-binary composer.phar …`.

1. `--php` on this invocation
2. `ABSTRAX_COMPOSER_PHP`
3. `--project` when the project runtime is PHP (versioned CLI such as `php8.5`, or a Remi path on RHEL-family)
4. Default in `/etc/abstrax/composer.json`
5. `php`

The global `composer` command on `PATH` uses steps 4–5. For a project's PHP version, use `abstrax composer run --project=…`, or `$ABSTRAX_CLI_PHP` in [Deploy](/docs/plugins/official/deploy) hooks.

## Commands

Global flags: `--json`, `--json-stream`, `--yes`, `--dry-run`, `--verbose`, `--quiet`, `--no-color`.

### `composer setup`

Download, verify, and install Composer. Requires root.

```bash
sudo abstrax composer setup
sudo abstrax composer setup --force
sudo abstrax composer setup --dry-run
```

| Flag | Description |
|---|---|
| `--force` | Replace an existing Composer binary that was not installed by this plugin |

### `composer self-update`

Replace the installed phar with the latest stable release (same verification as setup). Requires root. Composer must already be installed.

```bash
sudo abstrax composer self-update
```

### `composer remove`

Remove the managed wrapper and phar. Does not delete project `vendor/` directories or `auth.json`. Requires root.

```bash
sudo abstrax composer remove --yes
sudo abstrax composer remove --purge --yes
```

| Flag | Description |
|---|---|
| `--purge` | Also remove `/etc/abstrax/composer.json` |

Unmanaged binaries at `/usr/local/bin/composer` are refused; remove those by hand.

### `composer status`

Show whether Composer is installed, paths, Composer version, and the resolved PHP binary (including why it was chosen).

```bash
abstrax composer status
abstrax composer status --project=example.com
abstrax composer status --php=php8.2
abstrax composer status --json
```

### `composer configure`

Show or set the default PHP binary used when no project is given. Writes require root.

```bash
abstrax composer configure
sudo abstrax composer configure --php=php8.2
sudo abstrax composer configure --php=php
```

`--php=php` resets to the unversioned `php` command. The binary is checked with `php -r 'echo PHP_VERSION;'` before it is saved. If Composer is already installed, the wrapper is rewritten.

### `composer run [composer-args…]`

Run Composer. An Abstrax project is optional.

Put Abstrax flags **before** the Composer command:

```bash
abstrax composer run install
abstrax composer run install --no-dev --optimize-autoloader
abstrax composer run --project=example.com install --no-dev
abstrax composer run --php=php8.2 update
abstrax composer run --path=/srv/app --user=deploy install
```

| Flag | Description |
|---|---|
| `--project` | Use this Abstrax project for path, user, and PHP version |
| `--path` | Working directory (overrides the project path when both are set) |
| `--php` | PHP binary for this invocation |
| `--user` | User to run Composer as |
| `--allow-root` | Allow running Composer as root |

If you invoke the plugin with `sudo` and do not pass `--user` or `--project`, Composer runs as `SUDO_USER`. Root is refused unless `--allow-root` is set. Shared web users (`www-data`, `nginx`, `apache`) are not used as the run user.

`--verbose`, `--quiet`, and `--dry-run` are Abstrax flags. To pass those through to Composer, put them after `--`:

```bash
abstrax composer run --dry-run install
abstrax composer run -- install --dry-run
```

### `composer diagnose`

Check PHP, Composer, `git`, `unzip`, and common PHP extensions (`json`, `mbstring`, `xml`, `zip`, `curl`). When Composer is installed, also runs `composer diagnose`.

```bash
abstrax composer diagnose
abstrax composer diagnose --project=example.com
abstrax composer diagnose --php=php8.2
```

Missing `git` / `unzip` are reported with an `abstrax package install` hint.

### `composer auth`

Show or update Composer credentials for a user. Files are written to `~/.config/composer/auth.json` with mode `0600`. Tokens are never printed in full.

With no write flags, prints whether GitHub and HTTP basic credentials are configured (redacted).

```bash
abstrax composer auth --user=deploy
sudo abstrax composer auth --user=deploy --github-token=ghp_…
sudo abstrax composer auth --user=deploy --http-basic-host=repo.packagist.com \
  --username=token --password=secret
sudo abstrax composer auth --project=example.com --remove=github
sudo abstrax composer auth --user=deploy --remove=http-basic --http-basic-host=repo.packagist.com
```

| Flag | Description |
|---|---|
| `--user` | User whose `auth.json` to read or write |
| `--project` | Use the Abstrax project user |
| `--github-token` | GitHub OAuth token (empty value removes it) |
| `--http-basic-host` | Host for HTTP basic auth |
| `--username` / `--password` | HTTP basic credentials |
| `--remove` | `github` or `http-basic`. For `http-basic`, pass `--http-basic-host` to remove one host, or omit it to clear all |

Writes for another user require root. If neither `--user` nor `--project` is set, the current user (or `SUDO_USER`) is used.

## Config (`/etc/abstrax/composer.json`)

```json
{
  "version": 1,
  "php": "php8.2"
}
```

If `php` is omitted, the unversioned `php` binary is used when no project or flag applies.

## Action dispatch

```bash
sudo abstrax --json-stream --yes --action plugin.composer.setup --payload '{}'
abstrax --json --yes --action plugin.composer.run \
  --payload '{"args":["install","--no-dev"],"project":"example.com"}'
```

See [Action dispatch](/docs/plugins/how-it-works#action-dispatch).

## Related

- [Official plugins](/docs/plugins/official/)
- [Deploy](/docs/plugins/official/deploy) — Laravel hooks call `abstrax composer run`
- [Projects](/docs/commands/projects)
- [Plugin commands](/docs/commands/plugins)
- [Exit codes and output](/docs/reference/exit-codes)
