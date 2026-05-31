# Guides

These guides walk through common tasks from start to finish, using only commands that exist in Abstrax today. Each one assumes you have Abstrax installed (see [Installation](../getting-started/installation.md)) and that you can use `sudo`.

| Guide | What it covers |
|---|---|
| [Creating a user](creating-a-user.md) | Add a user, set a shell, grant sudo, manage groups |
| [Adding SSH access](adding-ssh-access.md) | Add an authorised key and harden the SSH server |
| [Installing packages](installing-packages.md) | Update lists, install, upgrade, and remove packages |
| [Creating a project](creating-a-project.md) | Set up an nginx-backed web project and add a certificate |
| [Managing cron jobs](managing-cron-jobs.md) | Schedule, list, and modify cron jobs |
| [Managing daemons](managing-daemons.md) | Run long-lived processes under Supervisor |

Before making changes, you can preview most commands with `--dry-run`, and you can inspect the system first with `abstrax doctor`.
