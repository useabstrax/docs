# Deploy plugin

Zero-downtime GitHub deployments for Abstrax projects. `abstrax project add` creates the project (user, path, nginx, runtime). This plugin deploys **application code** into a release layout under that path.

| Item | Value |
|---|---|
| Binary | `abstrax-deploy` |
| CLI | `abstrax deploy …` |
| Trust level | `official` |
| Platforms | Debian/Ubuntu and RHEL-family (Rocky/Alma/RHEL) |
| Web server | Nginx |

## Install

```bash
sudo abstrax plugin install deploy
abstrax deploy version
```

## How it works

Layout under the project path from `abstrax project inspect --json`:

```text
{project.path}/
  deploy.json
  releases/{YYYYMMDDHHMMSS}/
  current -> releases/{id}
  shared/
```

Each `deploy now`:

1. Shallow-clones the repository into a new release directory
2. Writes `.abstrax-release.json` and deletes `.git`
3. Symlinks configured shared paths from `shared/`
4. Runs `after_clone`, then `before_activate` hooks
5. Checks that the release and `public_dir` exist
6. Atomically flips `current`
7. Runs `after_activate` hooks
8. Prunes old releases (`keep_releases`, default 5)

There is no in-place `git pull`. Nginx PHP blocks already use `$realpath_root`, so flipping `current` does not reload PHP-FPM. Services are not restarted automatically — add restarts to `after_activate` if you need them.

If a deploy fails before activate, the incomplete release is deleted and `current` is left unchanged.

## Quick start

```bash
sudo abstrax project add example.com \
  --domains=example.com \
  --php --public-dir=public

sudo abstrax deploy setup example.com \
  --repository=git@github.com:acme/app.git \
  --branch=main \
  --preset=laravel \
  --no-first-deploy

# Add the printed public key: GitHub → Settings → Deploy keys (read-only)
abstrax deploy key example.com --show

sudo abstrax deploy now example.com --yes

abstrax deploy status example.com
abstrax deploy list example.com
```

## Commands

Mutating commands require root. `list`, `status`, and `key --show` / `--fingerprint` do not, if the files are readable.

Global flags: `--json`, `--json-stream`, `--yes`, `--dry-run`, `--verbose`, `--quiet`, `--no-color`.

### `deploy setup <project>`

Init, write config, create a deploy key, and optionally run the first deploy.

```bash
sudo abstrax deploy setup example.com \
  --repository=git@github.com:acme/app.git \
  --branch=main \
  --preset=laravel \
  --public-dir=public \
  --keep=5 \
  --no-first-deploy
```

| Flag | Description |
|---|---|
| `--repository` | Git URL (required unless prompted on a TTY) |
| `--branch` | Default branch (`main`) |
| `--preset` | `laravel`, `node`, `ruby`, `static`, or `none`. If omitted, inferred from the project runtime (`php` → `laravel`, `node` → `node`, and so on) |
| `--public-dir` | Public directory inside each release |
| `--keep` | Releases to keep (default 5) |
| `--no-first-deploy` | Skip the first deploy |
| `--yes` | Non-interactive; also runs the first deploy unless `--no-first-deploy` |

On a TTY without `--yes`, missing values are prompted. The first deploy is also prompted unless `--no-first-deploy` or `--yes`.

### `deploy init <project>`

Create `releases/`, `shared/`, and `deploy.json`. Sets the project public dir to `current/{public_dir}`. Does not clone.

```bash
sudo abstrax deploy init example.com \
  --preset=laravel \
  --repository=git@github.com:acme/app.git
```

Same flags as setup except `--no-first-deploy`. Pass `--yes` to overwrite an existing `deploy.json`.

### `deploy configure <project>`

Show or update `deploy.json`. Writes require root.

```bash
abstrax deploy configure example.com
sudo abstrax deploy configure example.com --branch=production --keep=8
sudo abstrax deploy configure example.com --shared=.env,storage
```

| Flag | Description |
|---|---|
| `--repository` | Git URL |
| `--branch` | Default branch |
| `--preset` | Re-apply a preset (replaces shared paths and hooks) |
| `--public-dir` | Public directory inside each release |
| `--keep` | Releases to keep |
| `--shared` | Comma-separated shared paths |

Changing `--public-dir` or `--preset` also updates the project public dir via Abstrax.

### `deploy key <project>`

Create an ed25519 key for the project user, store the path in config, and print GitHub instructions. Updates `known_hosts` for `github.com`.

```bash
sudo abstrax deploy key example.com
abstrax deploy key example.com --show
abstrax deploy key example.com --fingerprint
sudo abstrax deploy key example.com --rotate --yes
```

| Flag | Description |
|---|---|
| `--show` | Print the public key |
| `--fingerprint` | Print the SHA256 fingerprint |
| `--rotate` | Replace the key (requires `--yes`) |

Key path: `~/.ssh/abstrax_deploy_<project>` (`.` and `/` in the project name become `_`). Shared web users (`www-data`, `nginx`, `apache`) use `/var/www/.ssh/…`.

### `deploy now <project>`

Full release pipeline. Requires `repository` and `deploy_key` in config.

```bash
sudo abstrax deploy now example.com --yes
sudo abstrax deploy now example.com --ref=tags/v1.2.3 --yes
sudo abstrax deploy now example.com --ref=abc1234 --skip-hooks --yes
sudo abstrax deploy now example.com --no-activate --yes
```

| Flag | Description |
|---|---|
| `--ref` | Branch, tag (`tags/v1.0.0` or `refs/tags/v1.0.0`), or SHA. Default: configured branch. A bare name such as `v1.2.3` is treated as a **branch**, not a tag |
| `--keep` | Override `keep_releases` for this deploy |
| `--skip-hooks` | Skip all hooks |
| `--no-activate` | Prepare the release without flipping `current` |
| `--force` / `--yes` | Skip confirmation |

### `deploy rollback <project> [release-id]`

Point `current` at the previous release, or an explicit id. Re-runs `after_activate` hooks.

```bash
sudo abstrax deploy rollback example.com --yes
sudo abstrax deploy rollback example.com 20260814213045 --yes
sudo abstrax deploy rollback example.com --skip-hooks --yes
```

### `deploy list <project>`

List releases and mark the current one.

```bash
abstrax deploy list example.com
abstrax deploy list example.com --json
```

### `deploy status <project>`

Config, current release, symlink target, and last deploy metadata.

```bash
abstrax deploy status example.com
```

### `deploy hooks <project> [phase]`

List or edit hooks for `after_clone`, `before_activate`, or `after_activate`. Writes require root and a phase.

```bash
abstrax deploy hooks example.com
abstrax deploy hooks example.com after_activate
sudo abstrax deploy hooks example.com after_activate \
  --append='abstrax project service restart example.com example-worker --yes'
sudo abstrax deploy hooks example.com before_activate --clear
```

| Flag | Description |
|---|---|
| `--set` | Replace the phase with a single command |
| `--append` | Append a command |
| `--clear` | Remove all hooks for the phase |

## Presets

Applying a preset sets `public_dir`, `shared`, and hooks.

| Preset | `public_dir` | Shared | Default hooks |
|---|---|---|---|
| `laravel` | `public` | `.env`, `storage` | `after_clone`: `abstrax composer run --project="$ABSTRAX_PROJECT" --path="$ABSTRAX_RELEASE_PATH" install --no-dev --optimize-autoloader`. `before_activate`: `$ABSTRAX_CLI_PHP artisan migrate --force` |
| `node` | `.` | none | `after_clone`: `npm ci && npm run build` (the app must define a `build` script) |
| `ruby` | `.` | none | `after_clone`: `bundle install --deployment --without development test` |
| `static` | `.` | none | none |
| `none` | `.` | none | none |

On `setup` / `init` / `deploy now`, the Laravel preset also creates `shared/storage` (app, framework cache/sessions/views, logs) and a minimal `shared/.env` with a generated `APP_KEY` when that file is missing or empty. Existing non-empty `.env` files are never overwritten.

The Laravel `after_clone` hook uses the [Composer](/docs/plugins/official/composer) plugin. If it is not installed, `setup` / `init` / `configure --preset=laravel` prints:

```bash
sudo abstrax plugin install composer && sudo abstrax composer setup
```

Hooks that call `abstrax` or `abstrax-*` run as root so system-installed plugins are found. Pass `--project` / `--path` so Composer still runs as the project user in the release directory. Other hooks (for example `$ABSTRAX_CLI_PHP artisan …`) run as the project user.

No preset restarts services. For workers, add restarts in `after_activate`.

## Config (`deploy.json`)

Path: `{project.path}/deploy.json`

```json
{
  "version": 1,
  "project": "example.com",
  "repository": "git@github.com:acme/app.git",
  "branch": "main",
  "provider": "github",
  "keep_releases": 5,
  "public_dir": "public",
  "preset": "laravel",
  "shared": [".env", "storage"],
  "hooks": {
    "after_clone": ["abstrax composer run --project=\"$ABSTRAX_PROJECT\" --path=\"$ABSTRAX_RELEASE_PATH\" install --no-dev --optimize-autoloader"],
    "before_activate": ["$ABSTRAX_CLI_PHP artisan migrate --force"],
    "after_activate": []
  },
  "deploy_key": "/home/example/.ssh/abstrax_deploy_example_com"
}
```

`provider` must be `github`. Shared leaf names that contain a dot (`.env`) are treated as files; others (`storage`) as directories.

Hooks are shell strings run with `bash -lc`. **cwd** is the release path.

| Variable | Meaning |
|---|---|
| `ABSTRAX_PROJECT` | Project name |
| `ABSTRAX_PROJECT_PATH` | Project root |
| `ABSTRAX_RELEASE_PATH` | Release directory |
| `ABSTRAX_CURRENT_PATH` | Path to the `current` symlink |
| `ABSTRAX_SHARED_PATH` | Path to `shared/` |
| `ABSTRAX_BRANCH` | Configured branch |
| `ABSTRAX_REF` | Ref being deployed |
| `ABSTRAX_RELEASE_ID` | Release id |
| `ABSTRAX_CLI_PHP` | Versioned PHP CLI when the project runtime is PHP (`php8.5` on Debian/Ubuntu, Remi paths such as `/opt/remi/php85/root/usr/bin/php` on RHEL-family) |

## GitHub deploy keys

1. `sudo abstrax deploy key <project>` creates the key for the project user
2. Print it: `abstrax deploy key <project> --show`
3. GitHub → repository **Settings → Deploy keys → Add deploy key** (read-only is enough)
4. `known_hosts` is updated for `github.com` when the key is created

Rotate with `sudo abstrax deploy key <project> --rotate --yes` and replace the key on GitHub.

## Git refs

`--ref` is classified as:

- **SHA** if it matches `[0-9a-f]{7,40}`
- **Tag** if it starts with `tags/` or `refs/tags/`
- **Branch** otherwise (including names like `v1.2.3`)

Clones are shallow (`--depth 1`). SHAs that are not a branch tip fall back to a deeper fetch. After metadata is written, `.git` is removed so each release is a plain tree.

## Machine-readable output

```bash
sudo abstrax deploy now example.com --yes --json-stream
```

`--json-stream` prints NDJSON progress (`type=progress`) then a final `type=result` line. `--json` prints a single result object. Do not combine them. See [Exit codes and output](/docs/reference/exit-codes#json-stream-ndjson).

Agents can dispatch the same command:

```bash
sudo abstrax --json-stream --yes \
  --action plugin.deploy.now \
  --payload '{"args":["example.com"],"ref":"main"}'
```

See [Action dispatch](/docs/plugins/how-it-works#action-dispatch).

## Related

- [Official plugins](/docs/plugins/official/)
- [Composer](/docs/plugins/official/composer) — Laravel installs go through `abstrax composer run`
- [Projects](/docs/commands/projects)
- [Plugin commands](/docs/commands/plugins)
- [Exit codes and output](/docs/reference/exit-codes)
