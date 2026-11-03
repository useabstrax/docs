# Certificates (SSL)

The `ssl` command group manages TLS certificates using [Certbot](https://certbot.eff.org/) and Let's Encrypt. Certificates are tied to projects created with the [`project`](/docs/commands/projects) commands.

```text
abstrax ssl <action> [arguments] [flags]
```

## Permissions

`install`, `add`, `remove`, and `renew` require root. The read-only command `status` does not require root.

## `ssl install`

Install Certbot and the nginx plugin using the platform package manager (`apt` on Debian-family systems, `dnf` on RHEL-family systems).

```bash
sudo abstrax ssl install
```

On Rocky Linux, AlmaLinux, and CentOS Stream, Abstrax installs the EPEL repository (`epel-release`) when needed before installing Certbot. On RHEL or Oracle Linux, enable EPEL explicitly first:

```bash
sudo abstrax repo enable epel --enable-required-repos
sudo abstrax ssl install
```

or pass `--enable-required-repos` to `ssl install`.

Run this when `abstrax doctor` reports certbot as not found. `ssl add` and `ssl renew` require Certbot to be installed.


## `ssl add`

Obtain a certificate for a project and write HTTPS into that project's managed nginx vhost. By default HTTP is redirected to HTTPS.

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

Abstrax runs `certbot certonly --nginx --cert-name abstrax-<project>`, then updates the project's nginx vhost itself (Certbot does not edit the vhost). The certificate name is stored on the project and reused for later adds, renewals, status, and removal. A deploy hook reloads nginx after each renewal.

The vhost renders an HTTPS server (`listen 443 ssl`) when SSL is enabled, with an HTTP server that redirects when `redirect_http` is true. Abstrax validates with `nginx -t` before reloading. If nginx rejects the HTTPS vhost, the previous file is restored, `ssl_enabled` stays off, and `ssl.last_error` records why.

Use `--staging` when testing. The staging environment issues certificates that are not trusted by browsers but avoids the rate limits of the production environment.

## `ssl remove`

Remove HTTPS from the project's vhost, then delete the certificate.

```bash
sudo abstrax ssl remove <project>
```

Abstrax rewrites the vhost without HTTPS and runs `nginx -t` before `certbot delete`, then clears the stored SSL state. If nginx rejects the vhost, the certificate is left alone. If `certbot delete` fails, HTTPS is put back.

## `ssl renew`

Renew certificates using the stored certificate name, reload nginx, and refresh `ssl.expires_at` on the project. With no flag, all certificates are renewed; with `--project`, only that project's certificate is renewed.

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
PROJECT   CERTIFICATE        DOMAINS                    EXPIRY                STATUS
myapp     abstrax-myapp      myapp.com, www.myapp.com   2026-04-01T00:00:00Z  valid
```

Each entry reports:

| Field | Description |
|---|---|
| `project_name` | The Abstrax project name (empty for a certificate no project owns) |
| `certificate_name` | Certbot lineage name |
| `domains` | Names on the certificate |
| `expiry` | Certificate `notAfter` as RFC3339 UTC |
| `days_remaining` | Days until expiry |
| `redirect_http` | Whether HTTP redirects to HTTPS |
| `staging` | Whether the certificate was issued from staging |
| `status` | `valid`, `expiring` (within 30 days), `expired`, or `missing` |

Status is read from project state and the certificate under `/etc/letsencrypt/live`. `certbot certificates` output is only a fallback, and also lists certificates no project owns.

## Project SSL state

`project info` and the project state file expose an `ssl` object with `certificate_name`, `domains`, `redirect_http`, `staging`, `issued_at`, `expires_at`, and `last_error`. `ssl_enabled` is still written. A project that only has the older `ssl_enabled` flag is adopted the first time Abstrax needs its certificate name: the name comes from the `ssl_certificate` path in the existing vhost, then from the first domain.

Vhost rewrites from `project modify` and `project runtime set` render HTTPS from this stored state, so modifying a project no longer drops SSL.

## Notes

- These commands require Certbot to be installed. Run `abstrax doctor` to confirm Certbot is present.
- Let's Encrypt issues certificates over HTTP or DNS validation. The domains must resolve to the server and be reachable for HTTP validation to succeed.
- The production environment has rate limits. Use `--staging` while you are testing your setup.

## Related

- [Projects](/docs/commands/projects)
- [Creating a project](/docs/guides/creating-a-project)
- [Firewall](/docs/commands/firewall) - make sure ports 80 and 443 are open
