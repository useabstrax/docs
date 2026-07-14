# Firewall

The `firewall` group manages the system firewall. On Debian-family systems it uses UFW. On RHEL-family systems it uses firewalld.

```text
abstrax firewall <action> [flags]
```

Run `abstrax doctor` to see which firewall strategy is active (`ufw` or `firewalld`).

## Permissions

Most firewall commands require root. `firewall status` and `firewall rule list` do not require root.

## `firewall install`

Install the platform firewall package without enabling it.

| Distro family | Package |
|---|---|
| Debian/Ubuntu | `ufw` |
| Rocky/Alma/RHEL | `firewalld` |

```bash
sudo abstrax firewall install
```

After install, enable the firewall with SSH protection:

```bash
sudo abstrax firewall enable --allow-ssh
```

`firewall enable` will also install the package automatically if it is missing.

## `firewall status`

Show whether the firewall is active and list current rules.

```bash
abstrax firewall status
```

## `firewall enable`

Enable the firewall. On Debian-family hosts this enables UFW. On RHEL-family hosts this enables and starts the `firewalld` service.

If the firewall package is not installed yet (`ufw` or `firewalld`), Abstrax installs it first, then enables the firewall. Prefer `--allow-ssh` so SSH is opened before/while enabling.

```bash
sudo abstrax firewall enable --allow-ssh
```

| Flag | Default | Description |
|---|---|---|
| `--allow-ssh` | `false` | Open the SSH port before enabling |
| `--ssh-port` | `22` | SSH port to allow when `--allow-ssh` is set |

Enabling without opening SSH can lock you out of the server.

## `firewall disable`

Disable the firewall.

```bash
sudo abstrax firewall disable
```

## `firewall allow`

Allow traffic on a port.

```bash
sudo abstrax firewall allow 80
sudo abstrax firewall allow 443 --protocol=tcp
sudo abstrax firewall allow 8080 --from=203.0.113.0/24 --comment="app"
```

| Flag | Default | Description |
|---|---|---|
| `--protocol` | | Protocol (`tcp` or `udp`) |
| `--from` | | Allow only from this IP or CIDR |
| `--comment` | | Rule comment (UFW) |

On RHEL-family systems, ports `80`/`http` and `443`/`https` map to firewalld services. Rules are added permanently and firewalld is reloaded afterward.

## `firewall deny`

Deny traffic on a port.

```bash
sudo abstrax firewall deny 23
```

| Flag | Default | Description |
|---|---|---|
| `--protocol` | | Protocol (`tcp` or `udp`) |

Port deny via firewalld is limited; prefer rich rules with `firewall-cmd` for complex deny policies on RHEL-family hosts.

## `firewall allow-ip`

Allow all traffic from an IP or CIDR.

```bash
sudo abstrax firewall allow-ip 203.0.113.10
```

## `firewall deny-ip`

Deny all traffic from an IP or CIDR.

```bash
sudo abstrax firewall deny-ip 198.51.100.0/24
```

## `firewall rule list`

List current firewall rules.

```bash
abstrax firewall rule list
```

## `firewall rule remove`

Remove a rule by the ID shown in `firewall rule list`.

```bash
sudo abstrax firewall rule remove 3
```

- **UFW:** deletes the numbered UFW rule.
- **firewalld:** Abstrax assigns list IDs to services and ports from `firewall-cmd --list-all`. Removing an ID removes the matching service or port permanently and reloads firewalld.

## `firewall remove service`

Remove a firewalld service by name (RHEL-family).

```bash
sudo abstrax firewall remove service http
sudo abstrax firewall remove service https
```

## `firewall remove port`

Remove a firewalld port (RHEL-family).

```bash
sudo abstrax firewall remove port 8080/tcp
sudo abstrax firewall remove port 9090/udp
```

## RHEL-family notes

- firewalld changes use `--permanent` and are followed by `--reload`
- firewalld does not use UFW-style numbered rules; Abstrax provides equivalent removal via list IDs or explicit service/port commands
- SELinux is separate from firewalld; Abstrax detects and warns about SELinux but never disables it
- Prefer `abstrax firewall allow 80` / `allow 443` for web traffic so firewalld services are used where possible

## Notes

- Always ensure SSH access is allowed before enabling the firewall on a remote host
- Use `--verbose` to see the underlying `ufw` or `firewall-cmd` invocations
