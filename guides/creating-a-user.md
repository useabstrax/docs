# Creating a user

This guide creates a user suitable for deploying and running an application, gives it sudo access, and adds it to a couple of groups.

These commands change system state, so run them with `sudo`. See [Permissions](../configuration/permissions.md).

## 1. Preview the change

Use `--dry-run` to see what will happen first:

```bash
sudo abstrax user add deploy --grant-sudo --create-home --shell=/bin/bash --dry-run
```

## 2. Create the user

```bash
sudo abstrax user add deploy --grant-sudo --create-home --shell=/bin/bash
```

Expected output:

```text
User deploy created.
  Home:   /home/deploy
  Shell:  /bin/bash
  UID:    1001
  Groups: deploy, sudo
  Sudo:   granted
```

If you also want a password, add `--password`, which prompts for it securely:

```bash
sudo abstrax user add deploy --grant-sudo --password
```

If the user already exists, Abstrax reports a warning rather than failing.

## 3. Add the user to extra groups

To let the user work with web files and Docker, for example:

```bash
sudo abstrax user add-groups deploy www-data,docker
```

To replace the full set of supplementary groups instead of adding to them:

```bash
sudo abstrax user set-groups deploy www-data,deploy
```

## 4. Confirm the result

```bash
abstrax user info deploy
```

```text
  Username:    deploy
  UID:         1001
  GID:         1001
  Home:        /home/deploy
  Shell:       /bin/bash
  Groups:      deploy, sudo, www-data, docker
  Sudo:        yes
  Locked:      no
```

You can also list users and filter to those with sudo:

```bash
abstrax user list --sudo
```

## Common follow-ups

- Give the user SSH access: see [Adding SSH access](adding-ssh-access.md).
- Lock the account temporarily: `sudo abstrax user lock deploy`, and later `sudo abstrax user unlock deploy`.
- Remove the user (asks for confirmation): `sudo abstrax user remove deploy --delete-home`.

## Related

- [Users command reference](../commands/users.md)
- [Adding SSH access](adding-ssh-access.md)
- [Permissions](../configuration/permissions.md)
