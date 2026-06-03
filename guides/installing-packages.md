# Installing packages

- [Creating a user](/docs/guides/creating-a-user)
- [Adding SSH access](/docs/guides/adding-ssh-access)
- [Installing packages](/docs/guides/installing-packages)
- [Creating a project](/docs/guides/creating-a-project)
- [Managing cron jobs](/docs/guides/managing-cron-jobs)
- [Managing daemons](/docs/guides/managing-daemons)

This guide covers the common package workflow on Debian and Ubuntu systems. The package commands use `apt` and require root.

## 1. Update the package lists

Refresh the list of available packages first:

```bash
sudo abstrax package update
```

## 2. Search for a package

```bash
abstrax package search nginx
```

```text
NAME       DESCRIPTION
nginx      small, powerful, scalable web/proxy server
nginx-core nginx web/proxy server (standard version)
```

## 3. Install a package

```bash
sudo abstrax package install nginx
```

To install a specific version:

```bash
sudo abstrax package install nginx --version=1.24.0-1ubuntu1
```

## 4. Check what is installed

```bash
abstrax package info nginx
```

```text
  Name:          nginx
  Version:       1.24.0-1ubuntu1
  Architecture:  amd64
  Status:        installed
  Description:   small, powerful, scalable web/proxy server
```

List all installed packages:

```bash
abstrax package list
```

## 5. Apply upgrades

Upgrade everything:

```bash
sudo abstrax package upgrade
```

Apply security updates only:

```bash
sudo abstrax package upgrade --security-only
```

You can preview an upgrade without applying it:

```bash
sudo abstrax package upgrade --dry-run
```

## 6. Remove a package

```bash
sudo abstrax package remove nginx
```

To remove its configuration files as well:

```bash
sudo abstrax package remove nginx --purge
```

## Notes

- These commands only work where `apt` is available. Run `abstrax doctor` to confirm the detected package manager is `apt`.
- Use `--verbose` to see the underlying `apt` command Abstrax runs.

## Related

- [Packages command reference](/docs/commands/packages)
- [Cache (Redis, Memcached)](/docs/commands/system#cache-commands)
- [Troubleshooting](/docs/reference/troubleshooting)
