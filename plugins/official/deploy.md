# Deploy plugin

The **Deploy** plugin adds zero-downtime GitHub deployments for Abstrax projects. It is an official plugin: install it from the registry, then use `abstrax deploy …`.

`abstrax project add` sets up infrastructure (path, user, nginx, runtime). Deploy deploys **application code** into a release layout under the project path. It does not replace project creation.

| Item | Value |
|---|---|
| Binary | `abstrax-deploy` |
| CLI | `abstrax deploy …` |
| Trust level | `official` |
| Platforms | Debian/Ubuntu and RHEL-family (Rocky/Alma/RHEL) |
| Web server | Nginx only (Apache is not supported yet) |

## Install

```bash
sudo abstrax plugin install deploy
abstrax deploy version
```

For local development builds, see the plugin README in `plugins/deploy/` in the Abstrax repository.

## How it works

Deploy uses a Capistrano/Deployer-style layout under the project path from `abstrax project inspect --json`:

```text
{project.path}/
  deploy.json              # plugin-owned config
  releases/
    {YYYYMMDDHHMMSS}/      # immutable release directories
  current -> releases/{id} # atomic symlink
  shared/                  # persistent files/dirs linked into each release
```

Each `deploy now`:

1. Creates `releases/{id}/` and shallow-clones the configured repository
2. Writes `.abstrax-release.json`, then deletes the release `.git` directory
3. Symlinks configured shared paths from `shared/`
4. Runs `after_clone` and `before_activate` hooks
5. Health-checks that the release and `public_dir` exist
6. Atomically flips `current`
7. Runs `after_activate` hooks
8. Prunes old releases (`keep_releases`, default 5)

There is **no** in-place `git pull`, no bare/mirror git cache in v1, no PHP-FPM reload after activate, and no automatic supervisor restarts. Restart services only through hooks if you need to.

Abstrax PHP nginx blocks already use `$realpath_root` for `SCRIPT_FILENAME` / `DOCUMENT_ROOT`, so flipping `current` does not require reloading PHP-FPM.

## Quick start

Typical flow from an existing Abstrax project to a first deploy:

```bash
# 1. Create the project (infra only)
sudo abstrax project add example.com \
  --domains=example.com \
  --php --public-dir=public

# 2. One-shot deploy setup
sudo abstrax deploy setup example.com \
  --repository=git@github.com:acme/app.git \
  --branch=main \
  --preset=laravel \
  --no-first-deploy

# 3. Add the printed public key as a GitHub Deploy Key (read-only)
abstrax deploy key example.com --show

# 4. Deploy
sudo abstrax deploy now example.com --yes

# 5. Inspect
abstrax deploy status example.com
abstrax deploy list example.com
```

## Commands

Mutating commands require root (same spirit as `abstrax project add`). `list`, `status`, and read-only key/config views work without root when permissions allow.

Global flags that apply where relevant: `--json`, `--json-stream`, `--yes`, `--dry-run`, `--verbose`, `--quiet`, `--no-color`.

There is **no** `deploy release` command. Use `deploy now`.

### `deploy setup <project>`

Guided one-shot: init + configure + deploy key + optional first deploy.

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
| `--repository` | Git repository URL (required non-interactively) |
| `--branch` | Default branch (default `main`) |
| `--preset` | `laravel`, `node`, `ruby`, `static`, or `none` |
| `--public-dir` | Public directory inside each release |
| `--keep` | Number of releases to keep |
| `--no-first-deploy` | Skip the optional first deploy |
| `--yes` | Non-interactive; with setup, also runs the first deploy unless `--no-first-deploy` |

On a TTY without `--yes`, missing values are prompted.

### `deploy init <project>`

Scaffold `releases/`, `shared/`, and `deploy.json`. Sets the project public dir to `current/{public_dir}` through Abstrax (`project modify`). Does not clone.

```bash
sudo abstrax deploy init example.com --preset=laravel --repository=git@github.com:acme/app.git
```

### `deploy configure <project>`

Show or update `deploy.json`.

```bash
abstrax deploy configure example.com
sudo abstrax deploy configure example.com --branch=production --keep=8
sudo abstrax deploy configure example.com --shared=.env,storage
```

### `deploy key <project>`

Create a deploy key for the project user, store the path in config, and print GitHub Deploy Keys instructions. Updates `known_hosts` for `github.com`.

```bash
sudo abstrax deploy key example.com
abstrax deploy key example.com --show
abstrax deploy key example.com --fingerprint
sudo abstrax deploy key example.com --rotate --yes
```

| Flag | Description |
|---|---|
| `--show` | Print the public key |
| `--fingerprint` | Print the key fingerprint |
| `--rotate` | Replace the key (requires `--yes`) |

### `deploy now <project>`

Full release pipeline.

```bash
sudo abstrax deploy now example.com --yes
sudo abstrax deploy now example.com --ref=v1.2.3 --yes
sudo abstrax deploy now example.com --ref=abc1234 --skip-hooks --yes
sudo abstrax deploy now example.com --no-activate --yes
```

| Flag | Description |
|---|---|
| `--ref` | Branch, tag (`tags/v1.0.0`), or SHA (default: configured branch) |
| `--keep` | Override `keep_releases` for this deploy |
| `--skip-hooks` | Skip all hooks |
| `--no-activate` | Prepare the release without flipping `current` |
| `--force` | Skip confirmation |
| `--dry-run` | Preview without changing the filesystem |
| `--yes` | Skip confirmation |

On failure before activate, the incomplete release directory is removed and `current` is never changed.

### `deploy rollback <project> [release-id]`

Point `current` at the previous release, or an explicit release id. Re-runs `after_activate` hooks so user-defined restart hooks still apply.

```bash
sudo abstrax deploy rollback example.com --yes
sudo abstrax deploy rollback example.com 20260814213045 --yes
```

### `deploy list <project>`

List releases and mark the current one. Supports `--json`.

```bash
abstrax deploy list example.com
abstrax deploy list example.com --json
```

### `deploy status <project>`

Config summary, current release, symlink target, and last deploy metadata.

```bash
abstrax deploy status example.com
```

### `deploy hooks <project> [phase]`

List or edit hooks for `after_clone`, `before_activate`, or `after_activate`.

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

| Preset | Shared defaults | Default hooks |
|---|---|---|
| `laravel` | `.env`, `storage` | `after_clone`: `composer install --no-dev --optimize-autoloader`; `before_activate`: `$ABSTRAX_CLI_PHP artisan migrate --force` |
| `node` | none | `after_clone`: `npm ci && npm run build` (app must define a `build` script) |
| `ruby` | none | `after_clone`: `bundle install --deployment --without development test` |
| `static` | none | none |
| `none` | none | none |

Laravel includes migrate. No preset restarts services. For Node/Ruby apps or queue workers, add restarts in `after_activate`.

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
    "after_clone": ["composer install --no-dev --optimize-autoloader"],
    "before_activate": ["$ABSTRAX_CLI_PHP artisan migrate --force"],
    "after_activate": []
  },
  "deploy_key": "/home/example/.ssh/abstrax_deploy_example_com"
}
```

Prefer CLI flags for common edits. Hooks are shell strings run with `bash -lc` as the project user when isolated. **cwd** is the release path.

Injected environment variables include:

| Variable | Meaning |
|---|---|
| `ABSTRAX_PROJECT` | Project name |
| `ABSTRAX_PROJECT_PATH` | Project root |
| `ABSTRAX_RELEASE_PATH` | Current release directory |
| `ABSTRAX_CURRENT_PATH` | Path to the `current` symlink |
| `ABSTRAX_SHARED_PATH` | Path to `shared/` |
| `ABSTRAX_BRANCH` | Configured branch |
| `ABSTRAX_REF` | Ref being deployed |
| `ABSTRAX_RELEASE_ID` | Release id |
| `ABSTRAX_CLI_PHP` | Versioned PHP CLI when the project runtime is PHP |

`$ABSTRAX_CLI_PHP` resolves versioned binaries when possible (`php8.5` on Debian/Ubuntu, Remi paths such as `/opt/remi/php85/root/usr/bin/php` on RHEL-family).

## GitHub deploy keys

1. `sudo abstrax deploy key <project>` creates `~/.ssh/abstrax_deploy_<project>` for the project user
2. Print the public key: `abstrax deploy key <project> --show`
3. GitHub → repository **Settings → Deploy keys → Add deploy key** (read-only is enough)
4. `known_hosts` is updated for `github.com` when the key is created

Rotate with `sudo abstrax deploy key <project> --rotate --yes` and update the key on GitHub.

v1 is GitHub-oriented. A small provider seam exists so other hosts can be added later without rewriting the release engine.

## Shallow clone behaviour

- **Branches:** `git clone --depth 1 --branch <ref>`
- **Tags:** shallow clone by tag name, with fetch fallback
- **SHAs:** init + shallow fetch, with a deeper fetch if needed
- After metadata is written, `.git` is removed so each release is a plain tree

## Debian/Ubuntu and RHEL

Deploy does not hard-code distro nginx or PHP socket paths. It uses Abstrax APIs:

- `abstrax project inspect <project> --json`
- `abstrax project modify <project> --public-dir=current/{public_dir}`

Distro differences for nginx and PHP stay Abstrax’s concern. This plugin owns filesystem layout, git, hooks, and activation.

## Machine-readable output

For scripts and automation:

```bash
sudo abstrax deploy now example.com --yes --json-stream
```

`--json-stream` prints NDJSON progress lines (`type=progress`) then a final `type=result` line, matching Abstrax core. Use `--json` for a single result object. Do not combine `--json` and `--json-stream`. See [Exit codes and output](/docs/reference/exit-codes#json-stream-ndjson).

## Permissions and safety

- Mutating commands require root
- Git, hooks, and filesystem work run as the project user when isolated
- Shared `www-data` (or equivalent) ownership is respected when that is the project owner
- Failed deploys clean up incomplete releases and never flip `current`
- Rollback never invents service restarts; it only re-runs `after_activate`

## Related

- [Official plugins](/docs/plugins/official/)
- [Projects](/docs/commands/projects) — infrastructure only; Deploy deploys application code
- [Plugin commands](/docs/commands/plugins)
- [Integrating with Abstrax](/docs/plugins/integrating-with-abstrax)
- [Exit codes and output](/docs/reference/exit-codes)
