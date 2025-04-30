# Update a remote Drupal 8+ instance

This GitHub action runs remote commands over SSH to update a Drupal 8+ instance. When a deployment is made, a Drupal instance needs to run specific commands:

1. Database updates
2. Config import
3. Cache rebuild
4. Translation updates
5. etc.

## Inputs

- `path` - (Default: `empty`) - Full absolute path on the remote server where the Drupal instance is located
- `server_name` - (Default: `server`) - Is the name of the server given in the SSH configuration (`.ssh/config`)
- `drush` - (Default: `./vendor/bin/drush`) - Name of Drush executable to use for deployments instead of default one from vendor
- `enable_extra_modules` - (Default: empty) - After the deployment is done, some extra modules can be enabled (i.e. `field_ui views_ui devel devel_generate` etc.)
- `enable_maintenance_mode` - (Default: "true") - Enables the maintenance mode.
- `statuscake_api_key` - (Default: empty) – API key used to authenticate with the StatusCake API
- `statuscake_test_id` -  (Default: empty) – Comma-separated list of StatusCake test IDs to disable/enable monitoring

Here's an example how to configure a remote SSH server `myserver` given in the example below:

```yml
      - name: "Configure SSH"
        run: |
          mkdir -p ~/.ssh/
          echo "$SSH_KEY" > ~/.ssh/server.key
          chmod 600 ~/.ssh/server.key
          cat >>~/.ssh/config <<END
          Host myserver
            HostName $SSH_HOST
            User $SSH_USER
            IdentityFile ~/.ssh/server.key
            StrictHostKeyChecking no
          END
```

## Example

```yml
on:
  workflow_dispatch:

name: Tests and code
jobs:
  update:
    runs-on: ubuntu-latest
    steps:

      - name: "Update instance"
        id: release
        uses: eaudeweb/drupal-update-action@1.x
        with:
          path: ${{ secrets.PROJECT_DIR }}/live
          server_name: myserver
          enable_extra_modules: field_ui views_ui devel devel_generate views_ui webform_ui purge_ui config
          enable_maintenance_mode: ${{ inputs.enable_maintenance_mode }}
          statuscake_api_key: ${{ secrets.STATUSCAKE_API_KEY }}
          statuscake_test_id: ${{ secrets.PROD_STATUSCAKE_ID }}
```