# Services

The `service` group manages systemd services.

```text
abstrax service <action> <name>
```

## Permissions

`start`, `stop`, `restart`, `reload`, `enable`, and `disable` require root. `status` does not require root.

## Service names

A service name may contain letters, digits, and the characters `_ @ : . -`, up to 64 characters.

## Lifecycle commands

```bash
sudo abstrax service start <name>
sudo abstrax service stop <name>
sudo abstrax service restart <name>
sudo abstrax service reload <name>
sudo abstrax service enable <name>
sudo abstrax service disable <name>
```

- `enable` configures the service to start at boot.
- `disable` stops it starting at boot.

Note: `stop` and `restart` interrupt the service. Restarting a service that handles live traffic (such as a web server or database) causes a brief outage. Prefer `reload` where the service supports it.

## `service status`

```bash
abstrax service status nginx
```

```text
  Service:      nginx
  Active:       active
  Sub:          running
  Enabled:      enabled
  PID:          12345
  Description:  A high performance web server
```
