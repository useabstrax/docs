# SSH keys and SSH configuration

- [Users](/docs/commands/users)
- [SSH keys](/docs/commands/ssh-keys)
- [Packages](/docs/commands/packages)
- [System](/docs/commands/system)
- [Cron](/docs/commands/cron)
- [Daemons](/docs/commands/daemons)
- [Projects](/docs/commands/projects)
- [Certificates](/docs/commands/certificates)

This page covers two related command groups:

- `ssh-key` – manage SSH authorised keys for users.
- `ssh` – manage the SSH server (sshd) configuration.

## SSH key management

The `ssh-key` group manages entries in a user's `authorized_keys` file. Keys added by Abstrax carry a managed marker comment so they can be identified and managed later.

```text
abstrax ssh-key <action> <user> [arguments] [flags]
```

A managed key looks like this in the file:

```text
# abstrax:key id=github-deploy name="GitHub deploy key"
ssh-ed25519 AAAAC3... user@example
```

### Permissions

The `ssh-key` commands do not enforce a root check in code. In practice, editing another user's `~/.ssh/authorized_keys` requires sufficient file permissions, which usually means running as root or as the target user. Use `sudo` when managing keys for a user other than yourself.

The username argument is validated using the same rules as the [user commands](/docs/commands/users#username-validation).

### `ssh-key add`

Add a public key for a user.

```bash
abstrax ssh-key add <user> "<key>" [flags]
abstrax ssh-key add <user> <path-to-key> --from-file [flags]
```

| Flag | Description |
|---|---|
| `--name` | Key name / ID stored in the managed marker |
| `--comment` | Comment stored in the managed marker |
| `--from-file` | Treat the key argument as a file path and read the key from it |
| `--force` | Overwrite if the key already exists |

#### Examples

```bash
sudo abstrax ssh-key add deploy "ssh-ed25519 AAAAC3... ci@example"
sudo abstrax ssh-key add deploy "ssh-ed25519 AAAAC3..." --name=github-deploy --comment="CI key"
sudo abstrax ssh-key add deploy /home/deploy/id_ed25519.pub --from-file
```

#### Example output

```text
SSH key added for deploy.
  ID:          github-deploy
  Name:        github-deploy
  Fingerprint: SHA256:abc123...
```

### `ssh-key remove`

Remove a managed key by its ID. By default only managed keys can be removed.

```bash
abstrax ssh-key remove <user> <key-id> [flags]
```

| Flag | Description |
|---|---|
| `--fingerprint` | Match the key by fingerprint instead of relying only on the ID |
| `--force` | Remove even keys that are not managed by Abstrax |

```bash
sudo abstrax ssh-key remove deploy github-deploy
sudo abstrax ssh-key remove deploy github-deploy --fingerprint=SHA256:abc123...
```

### `ssh-key list`

List the keys for a user. Does not require root if you can read the file.

```bash
abstrax ssh-key list <user>
abstrax ssh-key list <user> --managed-only
```

| Flag | Description |
|---|---|
| `--managed-only` | Show only keys managed by Abstrax |

#### Example output

```text
ID              TYPE          FINGERPRINT          MANAGED
github-deploy   ssh-ed25519   SHA256:abc123...     yes
```

### `ssh-key info`

Show details for a single key.

```bash
abstrax ssh-key info <user> <key-id>
```

```text
  ID:            github-deploy
  Name:          GitHub deploy key
  Type:          ssh-ed25519
  Fingerprint:   SHA256:abc123...
  Comment:       CI key
  Managed:       yes
```

---

## SSH server configuration

The `ssh` group manages the SSH server configuration. Rather than editing the main `sshd_config`, Abstrax writes its settings to a dedicated include file:

```text
/etc/ssh/sshd_config.d/99-abstrax.conf
```

Before reloading, the configuration is validated with `sshd -t` (when `sshd` is available). The managed include file is backed up with a timestamped suffix before it is rewritten (see [Security](/docs/reference/security)).

```text
abstrax ssh <action> [arguments] [flags]
abstrax ssh config <action> [arguments] [flags]
```

### Permissions

`ssh config show` does not require root. All other `ssh` commands change configuration or restart the service and require root.

### `ssh config show`

Show the current SSH settings managed by Abstrax. Does not require root.

```bash
abstrax ssh config show
```

```text
  Port:                        22
  PermitRootLogin:             prohibit-password
  PasswordAuthentication:      yes
  ClientAliveInterval:         0
```

### `ssh config set-port`

Change the SSH listening port. Abstrax warns that changing the port can lock you out if the firewall is not updated.

```bash
sudo abstrax ssh config set-port 2222
sudo abstrax ssh config set-port 2222 --allow-firewall
```

| Flag | Description |
|---|---|
| `--allow-firewall` | Open the new port in the firewall if a firewall backend is available |

The port value is validated as a number in the range 1–65535.

### `ssh config set-timeout`

Set the SSH client alive interval (idle timeout) in seconds.

```bash
sudo abstrax ssh config set-timeout 300
```

### Root login and password authentication

```bash
sudo abstrax ssh config disable-root-login
sudo abstrax ssh config enable-root-login
sudo abstrax ssh config disable-password-auth
sudo abstrax ssh config enable-password-auth
```

`disable-password-auth` prints a warning reminding you to confirm you have a working SSH key before disabling password logins.

### `ssh reload` and `ssh restart`

```bash
sudo abstrax ssh reload
sudo abstrax ssh restart
```

- `reload` reloads the SSH server.
- `restart` restarts it, and warns that active sessions will briefly disconnect.

## Notes and warnings

- Changing the SSH port or disabling password authentication can lock you out of the server. Make sure you have working alternative access (a key, or another open session) before applying these changes.
- Use `--dry-run` to preview the changes first.

## Related

- [Adding SSH access](/docs/guides/adding-ssh-access)
- [Users](/docs/commands/users)
- [Firewall](/docs/commands/system#firewall-commands)
- [Security](/docs/reference/security)
