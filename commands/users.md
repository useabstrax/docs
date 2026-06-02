# Users

The `user` command group manages Linux user accounts: creating and removing users, managing their groups, granting or revoking sudo, setting the login shell, and locking or unlocking accounts.

```text
abstrax user <action> [arguments] [flags]
```

## Permissions

Commands that change accounts (`add`, `remove`, `grant-sudo`, `revoke-sudo`, `set-groups`, `add-groups`, `remove-groups`, `set-shell`, `lock`, `unlock`) require root. The read-only commands `info` and `list` do not require root.

## Username validation

Usernames are validated before any change is made. A username must:

- start with a lowercase letter or underscore,
- contain only lowercase letters, digits, underscores, and hyphens,
- be at most 32 characters.

Invalid names are rejected with a clear error.

## `user add`

Create a new user.

```bash
abstrax user add <name> [flags]
```

By default a home directory is created.

### Flags

| Flag | Default | Description |
|---|---|---|
| `--create-home` | `true` | Create the home directory |
| `--no-create-home` | `false` | Do not create the home directory |
| `--grant-sudo` | `false` | Add the user to the `sudo` group |
| `--groups` | | Additional groups, comma-separated |
| `--shell` | | Login shell (must be an absolute path) |
| `--uid` | | Custom UID |
| `--system` | `false` | Create a system user |
| `--password` | `false` | Prompt for a password securely |
| `--disabled-password` | `false` | Create the user without a password |
| `--comment` | | GECOS comment field |

The `--password` flag prompts for the password on the terminal. The password is never passed as a command-line argument.

### Examples

```bash
sudo abstrax user add deploy
sudo abstrax user add deploy --grant-sudo --groups=www-data,docker --shell=/bin/bash
sudo abstrax user add deploy --create-home --comment="Deploy user"
sudo abstrax user add svc --system --no-create-home --disabled-password
```

### Example output

```text
User deploy created.
  Home:   /home/deploy
  Shell:  /bin/bash
  UID:    1001
  Groups: deploy, sudo
  Sudo:   granted
```

If the user already exists, Abstrax reports this as a warning rather than failing, and the JSON `data` includes `"already_existed": true`.

### JSON

```bash
sudo abstrax user add deploy --grant-sudo --json
```

```json
{
  "status": "success",
  "action": "user.add",
  "summary": "User deploy created.",
  "data": {
    "username": "deploy",
    "uid": "1001",
    "home": "/home/deploy",
    "shell": "/bin/bash",
    "groups": ["deploy", "sudo"],
    "sudo": true,
    "created": true
  }
}
```

## `user remove`

Remove a user. This is a destructive command and asks for confirmation unless `--yes` is given.

```bash
abstrax user remove <name> [flags]
```

### Flags

| Flag | Default | Description |
|---|---|---|
| `--delete-home` | `false` | Delete the home directory |
| `--keep-home` | `false` | Keep the home directory |
| `--remove-cron` | `false` | Remove the user's crontab |
| `--kill-processes` | `false` | Kill the user's processes before removal |
| `--force` | `false` | Force removal |

### Examples

```bash
sudo abstrax user remove deploy
sudo abstrax user remove deploy --delete-home --kill-processes
sudo abstrax user remove deploy --yes
```

## `user grant-sudo` and `user revoke-sudo`

Add or remove a user from the `sudo` group.

```bash
sudo abstrax user grant-sudo <name>
sudo abstrax user revoke-sudo <name>
```

`revoke-sudo` asks for confirmation unless `--yes` is given.

## Group management

```bash
sudo abstrax user set-groups <name> <groups>
sudo abstrax user add-groups <name> <groups>
sudo abstrax user remove-groups <name> <groups>
```

- `set-groups` replaces the user's supplementary groups with the given list.
- `add-groups` adds the given groups.
- `remove-groups` removes the given groups.

Groups are given as a comma-separated list.

```bash
sudo abstrax user set-groups deploy www-data,deploy
sudo abstrax user add-groups deploy docker
sudo abstrax user remove-groups deploy docker
```

## `user set-shell`

Set the user's login shell. The shell must be an absolute path.

```bash
sudo abstrax user set-shell deploy /bin/bash
```

## `user lock` and `user unlock`

Lock or unlock a user account.

```bash
sudo abstrax user lock deploy
sudo abstrax user unlock deploy
```

## `user info`

Show details for a user. Does not require root.

```bash
abstrax user info deploy
```

### Example output

```text
  Username:    deploy
  UID:         1001
  GID:         1001
  Home:        /home/deploy
  Shell:       /bin/bash
  Groups:      deploy, sudo
  Sudo:        yes
  Locked:      no
```

## `user list`

List users. Does not require root.

```bash
abstrax user list
abstrax user list --sudo
abstrax user list --regular
abstrax user list --system
```

### Flags

| Flag | Description |
|---|---|
| `--system` | Show system users only |
| `--regular` | Show regular users only |
| `--sudo` | Show sudo users only |

### Example output

```text
USERNAME   UID    HOME            SHELL       SUDO
deploy     1001   /home/deploy    /bin/bash   yes
www-data   33     /var/www        /usr/sbin/nologin
```

## Related

- [Adding SSH access](guides/adding-ssh-access.html)
- [SSH keys](commands/ssh-keys.html)
- [Creating a user](guides/creating-a-user.html)
- [Permissions](configuration/permissions.html)
