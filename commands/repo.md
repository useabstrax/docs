# Repositories

The `repo` command group enables package repositories required by some Abstrax features on RHEL-family systems (Rocky Linux, AlmaLinux, RHEL, CentOS Stream, Oracle Linux).

Debian/Ubuntu systems do not need this command for current Abstrax features.

```text
abstrax repo <action>
```

## Permissions

`repo enable` requires root.

## Global flag

| Flag | Description |
|---|---|
| `--enable-required-repos` | Allow enabling policy-sensitive third-party repositories (EPEL, Remi, etc.) from other commands such as `project add` or `ssl install` |

On RHEL and Oracle Linux, `abstrax repo enable` for EPEL/Remi also requires `--enable-required-repos` or `--yes` to confirm intent.

## `repo enable`

Enable a named repository.

```bash
sudo abstrax repo enable epel
sudo abstrax repo enable crb
sudo abstrax repo enable remi --enable-required-repos
```

| Repository | Purpose |
|---|---|
| `epel` | Extra Packages for Enterprise Linux (Certbot; Remi dependency) |
| `crb` | CodeReady Builder / CRB (Remi dependency on EL9) |
| `remi` | Remi repository for multi-version PHP (Software Collections) |

### Behaviour by distro

- **Rocky / AlmaLinux / CentOS Stream:** EPEL and CRB may be enabled when a command needs them (EPEL for Certbot automatically; Remi still requires `--enable-required-repos` or this command).
- **RHEL / Oracle Linux:** EPEL and Remi require explicit consent via `abstrax repo enable … --enable-required-repos` or by passing `--enable-required-repos` to the command that needs the repo.

Abstrax prints clear output when enabling repositories and will not silently add third-party repositories on enterprise distributions.

### Remi and PHP

Multi-version PHP on RHEL-family systems uses Remi SCL packages. Before installing PHP via `project add` / `project modify`, either:

```bash
sudo abstrax repo enable remi --enable-required-repos
sudo abstrax project add app --php --php-version=8.3
```

or:

```bash
sudo abstrax project add app --php --php-version=8.3 --enable-required-repos
```

## Related

- [Supported platforms](/docs/reference/supported-platforms)
- [Projects](/docs/commands/projects)
- [Certificates](/docs/commands/certificates)
