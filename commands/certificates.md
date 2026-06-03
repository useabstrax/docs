# Certificates (SSL)

- [Users](/docs/commands/users)
- [SSH keys](/docs/commands/ssh-keys)
- [Packages](/docs/commands/packages)
- [System](/docs/commands/system)
- [Cron](/docs/commands/cron)
- [Daemons](/docs/commands/daemons)
- [Projects](/docs/commands/projects)
- [Certificates](/docs/commands/certificates)

The `ssl` command group manages TLS certificates using [Certbot](https://certbot.eff.org/) and Let's Encrypt. Certificates are tied to projects created with the [`project`](/docs/commands/projects) commands.

```text
abstrax ssl <action> [arguments] [flags]
```

## Permissions

`add`, `remove`, and `renew` require root. The read-only command `status` does not require root.

## `ssl add`

Obtain a certificate for a project. By default HTTP is redirected to HTTPS.

```bash
sudo abstrax ssl add <project> [flags]
```

| Flag | Default | Description |
|---|---|---|
| `--domains` | | Comma-separated domain names |
| `--email` | | Email for certificate registration |
| `--staging` | `false` | Use the Let's Encrypt staging environment |
| `--redirect-http` | `true` | Redirect HTTP to HTTPS |

```bash
sudo abstrax ssl add myapp --email=admin@example.com --redirect-http
sudo abstrax ssl add myapp --email=admin@example.com --staging
```

Use `--staging` when testing. The staging environment issues certificates that are not trusted by browsers but avoids the rate limits of the production environment.

## `ssl remove`

Remove the certificate for a project.

```bash
sudo abstrax ssl remove <project>
```

## `ssl renew`

Renew certificates. With no flag, all certificates are renewed; with `--project`, only that project's certificate is renewed.

```bash
sudo abstrax ssl renew
sudo abstrax ssl renew --project=myapp
```

| Flag | Description |
|---|---|
| `--project` | Renew only this project's certificate |

## `ssl status`

Show certificate status. With no argument, all certificates are shown; with a project name, only that project. Does not require root.

```bash
abstrax ssl status
abstrax ssl status myapp
```

```text
PROJECT   DOMAINS                    EXPIRY
myapp     myapp.com, www.myapp.com   2024-04-01
```

## Notes

- These commands require Certbot to be installed. Run `abstrax doctor` to confirm Certbot is present.
- Let's Encrypt issues certificates over HTTP or DNS validation. The domains must resolve to the server and be reachable for HTTP validation to succeed.
- The production environment has rate limits. Use `--staging` while you are testing your setup.

## Related

- [Projects](/docs/commands/projects)
- [Creating a project](/docs/guides/creating-a-project)
- [Firewall](/docs/commands/system#firewall-commands) – make sure ports 80 and 443 are open
