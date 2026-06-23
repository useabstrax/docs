# Abstrax documentation

Markdown source for the documentation served at [useabstrax.com/docs](https://useabstrax.com/docs).

## Contributing

Edit the `.md` files in this repository and open a pull request. Navigation is defined in `_menu.md`.

## Deployment

Docs are deployed automatically when changes are merged to `main`. The workflow rsyncs markdown files to the production server's shared storage directory (Deployer's `shared/storage/docs`), so updates are live immediately without a site redeploy.
