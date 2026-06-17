# Abstrax documentation

Markdown source for the documentation served at [useabstrax.com/docs](https://useabstrax.com/docs).

## Contributing

Edit the `.md` files in this repository and open a pull request. Navigation is defined in `_menu.md`.

## Deployment

Docs are deployed automatically when changes are merged to `main`. The workflow rsyncs markdown files to the production server's shared storage directory (Deployer's `shared/storage/docs`), so updates are live immediately without a site redeploy.

### Required GitHub Secrets

Configure these in the repository settings (Settings → Secrets and variables → Actions). No server details are stored in this repo.

| Secret | Description |
|--------|-------------|
| `DEPLOY_HOST` | Production server hostname or IP |
| `DEPLOY_USER` | SSH user for deployment |
| `DEPLOY_PATH` | Deployer base path on the server |
| `DEPLOY_SSH_KEY` | Private SSH key for CI deployment |

### One-time setup

1. Generate a dedicated deploy key (do not commit the key files):

   ```bash
   ssh-keygen -t ed25519 -C "github-actions-docs-deploy" -f docs-deploy -N ""
   ```

2. Add the public key to the server using Abstrax:

   ```bash
   sudo abstrax ssh-key add abstrax docs-deploy.pub --from-file \
     --name=github-actions-docs-deploy --comment="GitHub Actions docs deploy"
   ```

3. Add the GitHub Secrets listed above. Set `DEPLOY_SSH_KEY` to the full contents of the private key file (`docs-deploy`).

4. Push this workflow to `main` (or run **workflow_dispatch** from the Actions tab to deploy manually).
