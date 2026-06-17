# Uninstalling

How you remove Abstrax depends on how you installed it.

## Remove a binary installation

If you copied the binary to `/usr/local/bin`, delete it:

```bash
sudo rm /usr/local/bin/abstrax
```

## Abstrax data directories

Abstrax creates and uses these directories. They are not removed automatically when you delete the binary:

```text
/etc/abstrax        Configuration (config.json, mysql.json, projects/)
/var/lib/abstrax    Runtime state (plugin records and caches)
/var/log/abstrax    Logs
```

If you want to remove them as well:

```bash
sudo rm -rf /etc/abstrax /var/lib/abstrax /var/log/abstrax
```

## Important: changes made by Abstrax remain in place

Uninstalling the binary does not undo anything Abstrax did to your system. Users, groups, firewall rules, cron files, Supervisor configs, nginx virtual hosts, databases, and certificates remain exactly as they were.

If you want to remove those, do so before uninstalling, using the relevant Abstrax commands or the underlying tools. For example:

```bash
sudo abstrax project remove myapp --remove-vhost
sudo abstrax cron remove backup
sudo abstrax daemon remove queue-worker --stop
```

Managed cron jobs, daemons, and projects can be listed first so you know what exists:

```bash
abstrax cron list
abstrax daemon list
abstrax project list
```
