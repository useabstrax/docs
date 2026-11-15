# Cache

The `cache` group installs and manages Redis, Valkey, or Memcached.

```text
abstrax cache <action> [driver] [flags]
```

The driver is `redis`, `valkey`, or `memcached`.

## Permissions

`install`, `remove`, `start`, `stop`, `restart`, and `config` (when changing settings) require root. `status` and read-only `config` do not require root.

## `cache install`

```bash
sudo abstrax cache install <driver> [flags]
```

| Flag | Default | Description |
|---|---|---|
| `--version` | | Package version (not yet supported; omit it) |
| `--port` | `0` | Port to listen on (0 uses the driver default) |
| `--bind` | `127.0.0.1` | IP address to bind to |
| `--memory` | | Memory limit |
| `--enable` | `true` | Enable the service at boot |
| `--start` | `true` | Start the service after install |

If you set `--bind=0.0.0.0` for Redis or Valkey, Abstrax warns that this exposes the service on all interfaces and asks for confirmation (unless `--yes` is given).

```bash
sudo abstrax cache install redis
sudo abstrax cache install valkey
sudo abstrax cache install memcached
sudo abstrax cache install redis --bind=127.0.0.1 --memory=256mb
```

### Rocky / AlmaLinux 10+

AppStream ships **Valkey** instead of Redis. Prefer:

```bash
sudo abstrax cache install valkey
```

`cache install redis` without `--enable-required-repos` falls back to AppStream Valkey and prints a note. Pass `--enable-required-repos` (or enable Remi first) to install Redis from Remi:

```bash
sudo abstrax cache install redis --enable-required-repos
```

Rocky/Alma **9** still uses the AppStream `redis` package and does not require Remi for Redis.

## `cache remove`

```bash
sudo abstrax cache remove <driver> [flags]
```

| Flag | Description |
|---|---|
| `--purge` | Purge configuration files |

## Lifecycle commands

```bash
sudo abstrax cache start <driver>
sudo abstrax cache stop <driver>
sudo abstrax cache restart <driver>
```

## `cache status`

Show status for all drivers, or a single driver. Does not require root.

```bash
abstrax cache status
abstrax cache status redis
abstrax cache status --json
```

```text
DRIVER     RUNNING   ENABLED
redis      yes       yes
valkey     no        no
memcached  no        no
```

`--json` includes `installed` and `version` (`null` when unknown), plus `port` and `bind` when those can be read from the driver's config file.

## `cache config`

Show or update configuration for a driver.

```bash
abstrax cache config redis
sudo abstrax cache config redis --memory=128mb --maxmemory-policy=allkeys-lru
sudo abstrax cache config redis --port=6380 --bind=127.0.0.1
sudo abstrax cache config redis --requirepass
sudo abstrax cache config redis --clear-requirepass
```

Without flags, Abstrax prints the effective settings it can read from the config file (for example `/etc/redis/redis.conf`, `/etc/valkey/valkey.conf`, or the Memcached sysconfig/conf file).

| Flag | Description |
|---|---|
| `--port` | Listen port |
| `--bind` | Bind address |
| `--memory` | Memory limit (`maxmemory` on Redis/Valkey; `-m` on Memcached) |
| `--maxmemory-policy` | Redis/Valkey eviction policy |
| `--requirepass` | Prompt for a Redis/Valkey password |
| `--clear-requirepass` | Remove the Redis/Valkey password |

Changing settings restarts the service so they take effect.
