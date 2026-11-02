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

Before the rule is added, Abstrax tries to allow your current client IP on the SSH port. That only happens in an interactive SSH session. See [Automatic SSH lockout protection](#automatic-ssh-lockout-protection).

On RHEL-family systems, ports `80`/`http` and `443`/`https` map to firewalld services when `--from` is omitted. With `--from`, Abstrax adds a rich rule that accepts traffic from that source only, including for ports 80 and 443. An IPv6 source uses `family="ipv6"`. Rules are added permanently and firewalld is reloaded afterward.

## `firewall deny`

Deny traffic on a port.

```bash
sudo abstrax firewall deny 23
sudo abstrax firewall deny 3306 --from=203.0.113.0/24 --protocol=tcp
```

| Flag | Default | Description |
|---|---|---|
| `--protocol` | | Protocol (`tcp` or `udp`) |
| `--from` | | Deny only from this IP or CIDR |
| `--force` | `false` | Deny the port even if it would block SSH |

On firewalld, `firewall deny` adds IPv4 and IPv6 reject rich rules for the port. `--from` limits the reject rule to that source, using `family="ipv6"` for an IPv6 address.

Denying the SSH port is refused when the firewall is active and the change would leave no TCP path to SSH. `--yes` does not skip that check. See [Automatic SSH lockout protection](#automatic-ssh-lockout-protection).

## `firewall allow-ip`

Allow all traffic from an IP or CIDR.

```bash
sudo abstrax firewall allow-ip 203.0.113.10
sudo abstrax firewall allow-ip 203.0.113.10 --port=3306 --protocol=tcp --comment="mysql from app"
```

| Flag | Description |
|---|---|
| `--to` | Destination IP |
| `--port` | Limit the allow to this port |
| `--protocol` | Protocol (`tcp` or `udp`). When a port is set and this is omitted, both protocols are allowed |
| `--comment` | Rule comment (UFW) |

Before the rule is added, Abstrax tries to allow your current client IP on the SSH port. That only happens in an interactive SSH session. See [Automatic SSH lockout protection](#automatic-ssh-lockout-protection).

## `firewall deny-ip`

Deny all traffic from an IP or CIDR.

```bash
sudo abstrax firewall deny-ip 198.51.100.0/24 --comment="blocked"
```

| Flag | Description |
|---|---|
| `--comment` | Rule comment (UFW) |

Before the rule is added, Abstrax tries to allow your current client IP on the SSH port. That only happens in an interactive SSH session. See [Automatic SSH lockout protection](#automatic-ssh-lockout-protection).

## `firewall rule list`

List current firewall rules. Each rule has a stable `id` (the first 8 characters of a hash of the rule) and a `number`, which is its current position in the list. The position can change when other rules are added or removed. Prefer `id` when you remove a rule.

Listed rules include protocol, direction, source, destination, comment, and whether the rule is IPv6. On firewalld, rich rules are included and can be removed by id.

```bash
abstrax firewall rule list
```

```text
ID        NUM   PORT   PROTOCOL   ACTION   DIR   FROM           TO
3f2a91c0  1     22     tcp        ALLOW    IN
9c10ab44  2     443    tcp        ALLOW    IN    203.0.113.10
```

An empty `FROM` means any source. The ids above are examples; use the ids printed on your server.

## `firewall rule remove`

Remove a rule by the `id` from `firewall rule list`.

```bash
sudo abstrax firewall rule remove 3f2a91c0
```

The id identifies the rule by its specification, so a later removal cannot hit a different rule that shifted into an old position.

A numeric position still works and prints a warning, because that position can change:

```bash
sudo abstrax firewall rule remove 3
```

- **UFW:** an id deletes the rule by its allow/deny specification. A numeric position deletes that numbered UFW rule.
- **firewalld:** an id removes the matching service, port, or rich rule permanently and reloads firewalld. Rich rules created with `--from` are removed this way.

| Flag | Default | Description |
|---|---|---|
| `--force` | `false` | Remove the rule even if it would block SSH |

The command is refused when this would remove the last TCP path to the SSH port and the firewall is active. `--yes` does not skip that check. See [Automatic SSH lockout protection](#automatic-ssh-lockout-protection).

## `firewall remove service`

Remove a firewalld service by name (RHEL-family).

```bash
sudo abstrax firewall remove service http
sudo abstrax firewall remove service https
```

| Flag | Default | Description |
|---|---|---|
| `--force` | `false` | Remove the service even if it would block SSH |

`firewall remove service ssh` is refused when that service is the last rule allowing TCP to the SSH port and the firewall is active. `--yes` does not skip that check.

## `firewall remove port`

Remove a firewalld port (RHEL-family).

```bash
sudo abstrax firewall remove port 8080/tcp
sudo abstrax firewall remove port 9090/udp
```

| Flag | Default | Description |
|---|---|---|
| `--force` | `false` | Remove the port even if it would block SSH |

Removing the SSH port is refused when it is the last rule allowing TCP to that port and the firewall is active. `--yes` does not skip that check.

## Automatic SSH lockout protection

Abstrax tries not to cut off the SSH session you are using, and it refuses changes that would leave an active firewall with no way to reach SSH.

### Client IP protection

`firewall enable`, `allow`, `deny`, `allow-ip`, and `deny-ip` allow your current client IP on the SSH port before they change anything else, when Abstrax can see that IP.

That detection only works in an interactive SSH session. Abstrax reads `SSH_CONNECTION`, then `SSH_CLIENT`, then `who`. From a script, a systemd unit, or any other non-interactive run, it does nothing. It does not print an error.

`--allow-ssh` on `firewall enable` is still the way to open SSH for every client. Client-IP protection does not replace it, and it does not run for `firewall rule remove`, `firewall remove service`, or `firewall remove port`.

### Refusing a change that would block SSH

Before `firewall deny`, `firewall rule remove`, `firewall remove service`, and `firewall remove port`, Abstrax reads the SSH port from the sshd configuration and simulates the change against the current rules:

- **remove** uses the rule list without the rule, service, or port being deleted
- **deny** inserts the new deny before any allow, because UFW uses the first matching rule

If the firewall is active and, after that change, no rule allows TCP to the SSH port, the command stops:

```text
This would block SSH (port 22), and you could lose access to this server. Add another rule that allows SSH first, or pass --force to do it anyway.
```

The message uses the actual SSH port. An allow limited to one IP still counts as access, so removing one of two SSH allows is permitted when the other remains. A deny of the SSH port is placed before existing allows, so `firewall deny 22` is refused while the firewall is active even if an allow is already present.

`--force` skips the check and applies the change. `--yes` does not. `--yes` only skips confirmation prompts.

When the firewall is inactive, the change is allowed. Abstrax prints a warning that `firewall enable` will need `--allow-ssh`, because enabling later would apply the rules that no longer let SSH in.

Successful `--json` results for these commands include the check in `data` when it ran:

```json
"ssh_check": {"port": 22, "blocked": false}
```

`blocked` is true when the resulting rules would not allow TCP to that port. If the firewall is active in that case, the command errors instead of returning the result. `--force` skips the check, so `ssh_check` is omitted.

## RHEL-family notes

- firewalld changes use `--permanent` and are followed by `--reload`
- firewalld rule ids are a stable hash. `number` is the current list position. Remove a service, port, or rich rule by id, or use `firewall remove service` and `firewall remove port`
- SELinux is separate from firewalld; Abstrax detects and warns about SELinux but never disables it
- Prefer `abstrax firewall allow 80` / `allow 443` for web traffic so firewalld services are used where possible. Pass `--from` when the port should be limited to one source; that writes a rich rule instead of opening the service to every address

## Notes

- Always ensure SSH access is allowed before enabling the firewall on a remote host
- Use `--verbose` to see the underlying `ufw` or `firewall-cmd` invocations
