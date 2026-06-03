# Adding SSH access

- [Creating a user](/docs/guides/creating-a-user)
- [Adding SSH access](/docs/guides/adding-ssh-access)
- [Installing packages](/docs/guides/installing-packages)
- [Creating a project](/docs/guides/creating-a-project)
- [Managing cron jobs](/docs/guides/managing-cron-jobs)
- [Managing daemons](/docs/guides/managing-daemons)

This guide adds an SSH public key for a user, then optionally hardens the SSH server. It assumes the user already exists (see [Creating a user](/docs/guides/creating-a-user)).

## 1. Add the public key

You can pass the key directly:

```bash
sudo abstrax ssh-key add deploy "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5... ci@example" \
  --name=ci-deploy --comment="CI deploy key"
```

Or read it from a file with `--from-file`:

```bash
sudo abstrax ssh-key add deploy /home/deploy/id_ed25519.pub --from-file --name=ci-deploy
```

Expected output:

```text
SSH key added for deploy.
  ID:          ci-deploy
  Name:        ci-deploy
  Fingerprint: SHA256:abc123...
```

Abstrax writes the key into the user's `authorized_keys` with a managed marker comment, so it can be listed and removed later.

## 2. Confirm the key

```bash
abstrax ssh-key list deploy
```

```text
ID          TYPE          FINGERPRINT          MANAGED
ci-deploy   ssh-ed25519   SHA256:abc123...     yes
```

## 3. Test logging in

Before changing any server settings, confirm you can actually log in with the new key from another terminal:

```bash
ssh -i /path/to/private_key deploy@your-server
```

Do not skip this step if you plan to disable password authentication in the next step.

## 4. Optionally harden the SSH server

Once key-based login works, you can tighten the SSH configuration. Abstrax writes these settings to `/etc/ssh/sshd_config.d/99-abstrax.conf` and validates the configuration with `sshd -t` before reloading.

Disable password authentication (Abstrax warns you to confirm you have a working key first):

```bash
sudo abstrax ssh config disable-password-auth
```

Disable root login over SSH:

```bash
sudo abstrax ssh config disable-root-login
```

Apply the changes:

```bash
sudo abstrax ssh reload
```

Check the current settings at any time:

```bash
abstrax ssh config show
```

```text
  Port:                        22
  PermitRootLogin:             prohibit-password
  PasswordAuthentication:      no
  ClientAliveInterval:         0
```

## 5. Changing the SSH port (optional)

If you change the SSH port, open it in the firewall at the same time so you do not lock yourself out:

```bash
sudo abstrax ssh config set-port 2222 --allow-firewall
sudo abstrax ssh reload
```

If you use the firewall separately, make sure the new port is allowed before reloading SSH:

```bash
sudo abstrax firewall allow 2222 --protocol=tcp
```

## Safety reminders

- Always keep a second logged-in session open while changing SSH settings, so you can revert if something goes wrong.
- Confirm key login works before disabling password authentication.
- Changing the port without updating the firewall can lock you out.

## Related

- [SSH keys and SSH configuration](/docs/commands/ssh-keys)
- [Firewall](/docs/commands/system#firewall-commands)
- [Security](/docs/reference/security)
