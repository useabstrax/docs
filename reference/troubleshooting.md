# Troubleshooting

This page lists common problems, their likely causes, and how to fix them. The symptoms are based on the errors and validation messages produced by the application.

## "this command requires root privileges; please run with sudo"

**Symptom.** A command stops immediately with this message.

**Cause.** The command changes system state and must run as root, but you ran it as a normal user.

**Fix.** Re-run the command with `sudo`:

```bash
sudo abstrax user add deploy
```

See [Permissions](../configuration/permissions.md) for which commands need root.

## "invalid username ..." and other validation errors

**Symptom.** A command fails before doing anything, with a message like `invalid username "Deploy"` or `invalid domain "example"`.

**Cause.** The argument did not pass validation. For example:

- Usernames must start with a lowercase letter or underscore and contain only lowercase letters, digits, underscores, and hyphens (max 32 characters).
- Package names allow letters, digits, and `_ . + -`.
- Database names allow letters, digits, and underscores.
- Cron IDs and daemon/project names allow letters, digits, underscores, and hyphens.
- Domains must be well-formed.
- Ports must be numbers in the range 1–65535.
- Shells must be absolute paths.

**Fix.** Correct the argument to match the rules above.

## "--command is required"

**Symptom.** `cron add` or `daemon add` fails with this message.

**Cause.** You did not provide the `--command` flag.

**Fix.** Add `--command="..."`:

```bash
sudo abstrax cron add backup --command="/usr/local/bin/backup.sh" --daily
```

## "--schedule or a frequency flag is required"

**Symptom.** `cron add` fails with this message.

**Cause.** No schedule was given.

**Fix.** Add a frequency flag (such as `--daily`) or a `--schedule` expression. A `--schedule` must have exactly 5 fields:

```bash
sudo abstrax cron add report --command="php artisan report" --schedule="0 8 * * 1"
```

## "cron expression must have 5 fields ..."

**Symptom.** A cron command rejects your schedule.

**Cause.** The expression did not have exactly 5 space-separated fields.

**Fix.** Use the standard 5-field format: minute, hour, day, month, weekday. For example `0 3 * * *`.

## Unsupported platform warning

**Symptom.** `abstrax doctor` shows a warning that the platform is not fully supported.

**Cause.** The OS `ID` in `/etc/os-release` is not one of the supported Debian/Ubuntu based systems.

**Fix.** Run Abstrax on Ubuntu or Debian (or a close derivative). On other distributions, package and service operations may not work because they assume `apt` and `systemd`. See [Supported platforms](supported-platforms.md).

## A tool is "not found" in doctor

**Symptom.** `abstrax doctor` lists a tool such as nginx, certbot, or supervisor as `not found`, and the related commands fail.

**Cause.** Abstrax calls these tools rather than bundling them. If the tool is not installed (or not on the `PATH`), the related commands cannot work.

**Fix.** Install the tool first. For example:

```bash
sudo abstrax package install nginx
sudo abstrax package install certbot
```

For Supervisor, you can install it as part of adding a daemon:

```bash
sudo abstrax daemon add worker --command="..." --install-supervisor
```

## Package commands do nothing or fail

**Symptom.** `package install` or `package update` fails.

**Cause.** The package commands use `apt`. On a system without `apt`, or where `apt` is busy/locked, they will fail.

**Fix.** Confirm `apt` is the detected package manager with `abstrax doctor`. Make sure no other package operation is running, then retry. Use `--verbose` to see the exact command.

## Service commands fail

**Symptom.** `service start`/`stop`/etc. fails.

**Cause.** Service operations assume `systemd`. If systemd is not running (for example inside some containers), these commands cannot manage services.

**Fix.** Run on a system with systemd. Check with `abstrax doctor` that the service manager is `systemd`.

## MySQL commands fail to connect

**Symptom.** `mysql test` or other MySQL commands report a connection error.

**Cause.** The saved connection config is missing or wrong, or the database is not reachable.

**Fix.** Set the connection config (run as root so the file can be written and read):

```bash
sudo abstrax mysql config set --host=127.0.0.1 --user=root --password
abstrax mysql test
```

The config is stored at `/etc/abstrax/mysql.toml` with mode 0600, so reading it generally requires root.

## "nginx configuration test failed"

**Symptom.** `web test` reports a failure and prints nginx output; the JSON `error_code` is `config_invalid`.

**Cause.** The nginx configuration is not valid.

**Fix.** Read the printed output to find the offending file and line. Fix the configuration, then run `abstrax web test` again before reloading. Avoid reloading with a broken config.

## Locked out after an SSH change

**Symptom.** You cannot reconnect after changing the SSH port, disabling password auth, or disabling root login.

**Cause.** The change removed your only way in (for example you had no working key, or the firewall did not allow the new port).

**Fix.** Use an existing open session or out-of-band console access to revert:

```bash
sudo abstrax ssh config enable-password-auth
sudo abstrax ssh config set-port 22 --allow-firewall
sudo abstrax ssh reload
```

To avoid this, always keep a second session open while changing SSH settings, test key login before disabling passwords, and open the new port in the firewall before changing the SSH port. See [Adding SSH access](../guides/adding-ssh-access.md).

## "port must be a number" when allowing a port

**Symptom.** `firewall allow 443/tcp` fails with `port must be a number, got "443/tcp"`.

**Cause.** `firewall allow` validates the port as a plain number. It does not accept the `port/protocol` form.

**Fix.** Pass the protocol as a flag instead:

```bash
sudo abstrax firewall allow 443 --protocol=tcp
```

Note that `firewall deny` does not validate the port, so it accepts a value as given.

## Firewall locked you out

**Symptom.** You lose connectivity after `firewall enable`.

**Cause.** The firewall was enabled without allowing SSH.

**Fix.** Always enable with `--allow-ssh` (and `--ssh-port` if you changed it):

```bash
sudo abstrax firewall enable --allow-ssh --ssh-port=2222
```

If you are already locked out, you will need console access to run `sudo abstrax firewall disable` or to allow the SSH port.

## No log file found

**Symptom.** `abstrax log` prints `No log file found at /var/log/abstrax/abstrax.log.`

**Cause.** The log file does not exist yet.

**Fix.** This is informational, not an error. The log directory is created by package installation; the file appears when there is something to log.

## Related

- [Permissions](../configuration/permissions.md)
- [Security](security.md)
- [Supported platforms](supported-platforms.md)
- [Exit codes and output](exit-codes.md)
