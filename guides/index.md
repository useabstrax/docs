# Guides

- [Creating a user](/guides/creating-a-user.html)
- [Adding SSH access](/guides/adding-ssh-access.html)
- [Installing packages](/guides/installing-packages.html)
- [Creating a project](/guides/creating-a-project.html)
- [Managing cron jobs](/guides/managing-cron-jobs.html)
- [Managing daemons](/guides/managing-daemons.html)

These guides walk through common tasks from start to finish, using only commands that exist in Abstrax today. Each one assumes you have Abstrax installed (see [Installation](/getting-started/installation.html)) and that you can use `sudo`.

| Guide | What it covers |
|---|---|
| [Creating a user](/guides/creating-a-user.html) | Add a user, set a shell, grant sudo, manage groups |
| [Adding SSH access](/guides/adding-ssh-access.html) | Add an authorised key and harden the SSH server |
| [Installing packages](/guides/installing-packages.html) | Update lists, install, upgrade, and remove packages |
| [Creating a project](/guides/creating-a-project.html) | Set up an nginx-backed web project and add a certificate |
| [Managing cron jobs](/guides/managing-cron-jobs.html) | Schedule, list, and modify cron jobs |
| [Managing daemons](/guides/managing-daemons.html) | Run long-lived processes under Supervisor |

Before making changes, you can preview most commands with `--dry-run`, and you can inspect the system first with `abstrax doctor`.
