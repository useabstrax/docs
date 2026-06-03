# Updating

- [Installation](/docs/getting-started/installation)
- [Updating](/docs/getting-started/updating)
- [Uninstalling](/docs/getting-started/uninstalling)

Abstrax does not have a built-in self-update command. The `abstrax agent update` command exists but is a placeholder and is not yet implemented; it only prints a message. You update Abstrax the same way you installed it.

## Check your current version

```bash
abstrax version
```

## Update from a release archive

Download the newer archive from the [releases page](https://github.com/useabstrax/abstrax/releases), extract it, and replace the existing binary:

```bash
# Replace <version> with the release version
tar -xzf abstrax_<version>_linux_amd64.tar.gz
chmod +x abstrax
sudo mv abstrax /usr/local/bin/abstrax
```

Then confirm the new version:

```bash
abstrax version
```

## Update a package installation

If you installed the `.deb` package, install the newer package version over the top:

```bash
sudo dpkg -i abstrax_<version>_<arch>.deb
```

Your configuration in `/etc/abstrax` and state in `/var/lib/abstrax` are left in place.

## Update a source build

Pull the latest code and rebuild:

```bash
cd abstrax/cli
git pull
go build -o abstrax ./cmd/abstrax
sudo mv abstrax /usr/local/bin/abstrax
```

## Notes

- Updating the binary does not change anything you have already configured on the server (users, firewall rules, projects, cron jobs, and so on). Those live in the system, not in the binary.
- Abstrax state files in `/var/lib/abstrax` and config in `/etc/abstrax` are not modified by an update.
