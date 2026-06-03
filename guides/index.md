# Guides

These guides walk through common tasks from start to finish, using only commands that exist in Abstrax today. Each one assumes you have Abstrax installed (see [Installation](/docs/getting-started/installation)) and that you can use `sudo`.

| Guide | What it covers |
|---|---|
| [Creating a user](/docs/guides/creating-a-user) | Add a user, set a shell, grant sudo, manage groups |
| [Adding SSH access](/docs/guides/adding-ssh-access) | Add an authorised key and harden the SSH server |
| [Installing packages](/docs/guides/installing-packages) | Update lists, install, upgrade, and remove packages |
| [Creating a project](/docs/guides/creating-a-project) | Set up an nginx-backed web project and add a certificate |
| [Managing cron jobs](/docs/guides/managing-cron-jobs) | Schedule, list, and modify cron jobs |
| [Managing daemons](/docs/guides/managing-daemons) | Run long-lived processes under Supervisor |

Before making changes, you can preview most commands with `--dry-run`, and you can inspect the system first with `abstrax doctor`.
