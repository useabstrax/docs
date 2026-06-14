# Firewall

The `firewall` group manages the UFW firewall.

```text
abstrax firewall <action> [arguments] [flags]
abstrax firewall rule <action> [arguments]
```

## Automatic SSH lockout protection

On `allow`, `deny`, `allow-ip`, `deny-ip`, and `enable`, Abstrax automatically ensures your current IP address can reach the SSH port before applying the requested change. This helps prevent locking yourself out when editing firewall rules over SSH.

- **SSH port** is read from sshd configuration: the Abstrax managed include file (`/etc/ssh/sshd_config.d/99-abstrax.conf`) first, then the main `/etc/ssh/sshd_config`, defaulting to `22`.
- **Your IP address** is detected from the SSH session (`SSH_CONNECTION` or `SSH_CLIENT` environment variables). When running under `sudo`, Abstrax falls back to parsing `who` output, which uses the controlling terminal of your session.
- **Idempotent**: if an allow rule for your IP on the SSH port already exists, no duplicate rule is added. For example, if `enable` adds the rule, a later `allow 443` will not add it again.
- **Undetectable IP**: if your IP cannot be determined (local console, cron job, or automation), the command still runs but Abstrax prints a warning that SSH protection was skipped.

The auto-protect rule is source-restricted (your IP only). On `enable`, `--allow-ssh` still opens the SSH port globally for all IPs, which is useful on servers with multiple administrators.

## Permissions

`enable`, `disable`, `allow`, `deny`, `allow-ip`, `deny-ip`, and `rule remove` require root. `status` and `rule list` do not require root.

## `firewall status`

Show whether the firewall is active and list its rules. Does not require root.

```bash
abstrax firewall status
```

```text
  Firewall:    ufw
  Status:      active

  Rules:
    [1] 22/tcp               ALLOW
    [2] 443/tcp              ALLOW
```

## `firewall enable`

Enable the firewall. Abstrax warns that enabling the firewall without opening SSH may lock you out, and asks for confirmation unless `--yes` is given.

```bash
sudo abstrax firewall enable --allow-ssh
sudo abstrax firewall enable --allow-ssh --ssh-port=2222
```

| Flag | Default | Description |
|---|---|---|
| `--allow-ssh` | `false` | Open the SSH port before enabling |
| `--ssh-port` | `22` | SSH port to allow |

Always use `--allow-ssh` (and `--ssh-port` if you changed it) unless you have another way into the server.

## `firewall disable`

```bash
sudo abstrax firewall disable
```

Asks for confirmation unless `--yes` is given.

## Allowing and denying ports

```bash
sudo abstrax firewall allow <port> [flags]
sudo abstrax firewall deny <port> [flags]
```

`allow` flags:

| Flag | Description |
|---|---|
| `--protocol` | Protocol (`tcp` or `udp`) |
| `--from` | Allow only from this IP or CIDR |
| `--comment` | Rule comment |

`deny` supports `--protocol`.

The behaviour differs between `allow` and `deny`:

- `firewall allow <port>` validates the port as a plain number in the range 1-65535. A value such as `443/tcp` is rejected with `port must be a number`. To restrict the protocol, use the `--protocol` flag.
- `firewall deny <port>` does not validate the port, so a value is passed through to UFW as given.

```bash
sudo abstrax firewall allow 80
sudo abstrax firewall allow 443 --protocol=tcp
sudo abstrax firewall deny 23
```

## Allowing and denying IPs

```bash
sudo abstrax firewall allow-ip <ip-or-cidr> [flags]
sudo abstrax firewall deny-ip <ip-or-cidr>
```

The IP or CIDR is validated. `allow-ip` supports `--to` (destination IP) and `--port` (specific port).

```bash
sudo abstrax firewall allow-ip 192.168.1.0/24
sudo abstrax firewall deny-ip 10.0.0.5
```

## Managing rules

```bash
abstrax firewall rule list
sudo abstrax firewall rule remove <id>
```

`rule list` shows the numbered rules. `rule remove` removes a rule by its number.

```text
ID   PORT/IP     PROTOCOL   ACTION
1    22          tcp        ALLOW
2    443         tcp        ALLOW
```
