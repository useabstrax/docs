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

On firewalld, `firewall deny` adds IPv4 and IPv6 reject rich rules for the port. `--from` limits the reject rule to that source, using `family="ipv6"` for an IPv6 address.

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

## `firewall deny-ip`

Deny all traffic from an IP or CIDR.

```bash
sudo abstrax firewall deny-ip 198.51.100.0/24 --comment="blocked"
```

| Flag | Description |
|---|---|
| `--comment` | Rule comment (UFW) |

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
- firewalld rule ids are a stable hash. `number` is the current list position. Remove a service, port, or rich rule by id, or use `firewall remove service` and `firewall remove port`
- SELinux is separate from firewalld; Abstrax detects and warns about SELinux but never disables it
- Prefer `abstrax firewall allow 80` / `allow 443` for web traffic so firewalld services are used where possible. Pass `--from` when the port should be limited to one source; that writes a rich rule instead of opening the service to every address

## Notes

- Always ensure SSH access is allowed before enabling the firewall on a remote host
- Use `--verbose` to see the underlying `ufw` or `firewall-cmd` invocations
