# Users

The `user` command group manages Linux user accounts: creating and removing users, managing their groups, granting or revoking sudo, setting the login shell, locking or unlocking accounts, and listing or ending interactive login sessions.

```text
abstrax user <action> [arguments] [flags]
```

## Permissions

Commands that change accounts (`add`, `remove`, `grant-sudo`, `revoke-sudo`, `set-groups`, `add-groups`, `remove-groups`, `set-shell`, `lock`, `unlock`, `logout`) require root. The read-only commands `info`, `list`, and `sessions` do not require root.

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
| `--grant-sudo` | `false` | Grant passwordless sudo (adds the user to the `sudo` group and installs a sudoers drop-in) |
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

Add or remove passwordless sudo for a user. Granting sudo adds the user to the `sudo` group and installs `/etc/sudoers.d/abstrax-<name>` with `NOPASSWD`. Revoking sudo removes both.

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

`user lock` locks the password and expires the account, so new SSH key logins are refused as well as passwords. Processes already running as that user, including daemons and cron jobs, keep running.

`user unlock` unlocks the password and restores the account expiry from before the lock. An account locked by a release older than 2.4.0 has no saved expiry, so unlock clears the expiry.

```bash
sudo abstrax user lock deploy
sudo abstrax user lock deploy --password-only
sudo abstrax user unlock deploy
```

| Flag | Description |
|---|---|
| `--password-only` | Lock the password only. SSH keys can still log in |

Locking an account does not end sessions that are already open. Use `user logout` to drop those interactive login sessions.

## `user sessions`

List a user's interactive login sessions. Does not require root. Daemons and cron jobs are not included. When the user has no login sessions, the command succeeds and prints an empty list.

```bash
abstrax user sessions deploy
abstrax user sessions deploy --json
```

Each session includes the session id, TTY, source IP, login time, and idle time. Idle is empty when the system does not report it.

Abstrax reads `loginctl list-sessions` when logind returns JSON, and `who -u` otherwise.

### Example output

```text
SESSION   TTY     SOURCE         LOGIN                  IDLE
3         pts/1   203.0.113.5    2026-10-31 12:00:00    00:05
```

### JSON

```json
{
  "status": "success",
  "action": "user.sessions",
  "data": [
    {
      "id": "3",
      "tty": "pts/1",
      "source_ip": "203.0.113.5",
      "login_time": "2026-10-31 12:00:00",
      "idle": "00:05"
    }
  ]
}
```

`idle` is `null` when it was not reported.

## `user logout`

End interactive login sessions for a user. Requires root. Asks for confirmation unless `--yes` is given.

```bash
sudo abstrax user logout deploy --session 3
sudo abstrax user logout deploy --all
sudo abstrax user logout deploy --all --yes
```

Pass `--session` or `--all`. Passing both is an error, and so is passing neither.

`--session` ends that one login session. `--all` ends each of the user's interactive login sessions. Existing SSH connections drop. Daemons and cron jobs running as that user keep running. Logout does not kill every process the user owns.

With logind, Abstrax runs `loginctl terminate-session` for each selected session. It does not run `loginctl terminate-user`. Without logind, it sends SIGHUP and then SIGKILL only to that session's `sshd: <user>@pts/N` process.

### Flags

| Flag | Description |
|---|---|
| `--session <id>` | Session id to end |
| `--all` | End every interactive login session for this user |

Lock the account, then run `user logout --all`, to refuse new logins and drop the ones already open.

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
  Expired:     no
  Login blocked: no
```

`Locked` means the password is locked. `Expired` means the account expiry date has passed, which also refuses SSH key logins. `Login blocked` is yes when both are true. JSON uses `locked`, `expired`, and `login_blocked`.

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
USERNAME   UID    HOME            SHELL       SUDO   LOCKED   EXPIRED   BLOCKED
deploy     1001   /home/deploy    /bin/bash   yes
www-data   33     /var/www        /usr/sbin/nologin
```

`LOCKED`, `EXPIRED`, and `BLOCKED` are blank unless that state is set. They match `locked`, `expired`, and `login_blocked` in JSON.

## Related

- [Adding SSH access](/docs/guides/adding-ssh-access)
- [SSH keys](/docs/commands/ssh-keys)
- [Creating a user](/docs/guides/creating-a-user)
- [Permissions](/docs/configuration/permissions)
