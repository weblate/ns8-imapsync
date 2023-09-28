# ns8-imapsync
## Install

Instantiate the module with:

    add-module ghcr.io/nethserver/imapsync:latest 1

The output of the command will return the instance name.
Output example:

    {"module_id": "imapsync1", "image_name": "imapsync", "image_url": "ghcr.io/nethserver/imapsync:latest"}

## Configure

Let's assume that the imapsync instance is named `imapsync1`.

We need to bind imapsync to a mail server inside the cluster, we use the MODULE_UUID of mail server to configure imapsync, reveal the vmail master secret of local users and start the container waiting for cron tasks or individual user tasks

Launch `configure-module`, by setting the following parameters:
- `mail_server`: module uuid of the mail server
- `mail_hostname`: mail hostname of the mail server

Example:

    api-cli run module/imapsync1/configure-module --data '{
        "mail_server":"8dd3b3fe-609c-42f7-a2d1-cecba9461bea",
        "mail_hostname":"foo.domain.com"
    }'

The above command will:
- start and configure the imapsync instance

## start to sync a remote imap account to local user account

security : "tls" or "ssl" or ""
trash_sync: "enabled","disabled"
exclude: folder1,folder2,folder3

Example:

    api-cli run module/imapsync1/create-task --data '{
        "local_user":"administrator",
        "remote_hostname":"imap.foo.com",
        "remote_port":"143",
        "security":"tls",
        "delete":"enabled",
        "trash_sync":"disabled",
        "exclude":"folder1,folder2",
        "remote_username":"username",
        "remote_password":"password"
    }'

## delete env and stop a running synchronisation

Example:

    api-cli run module/imapsync1/delete-task --data '{
        "local_user":"administrator"
    }'

## stop a running synchronisation

Example:

    api-cli run module/imapsync1/stop-task --data '{
        "local_user":"administrator"
    }'

## start all configured tasks
Read all `imapsync/*.env` files and start the synchronization

Example:

    api-cli run module/imapsync1/start-all-tasks

## stop all configured tasks
Read all `imapsync/*.env` files and stop the synchronization

Example:

    api-cli run module/imapsync1/stop-all-tasks

## read configuration
Read configuration from `environment` and from `imapsync/*.{env,pwd}`
Example:

    api-cli run module/imapsync1/get-configuration

Answer:

```json
{"mail_server": "ae2ef222-7a53-449e-aa5a-b03736e51a6a", "mail_server_URL": [{"name": "mail1", "label": "mail1 (R3.rocky9-3.org)", "value": "ae2ef222-7a53-449e-aa5a-b03736e51a6a,R3.rocky9-3.org"}]}
```

## Uninstall

To uninstall the instance:

    remove-module --no-preserve imapsync1

## Testing

Test the module using the `test-module.sh` script:


    ./test-module.sh <NODE_ADDR> ghcr.io/nethserver/imapsync:latest

The tests are made using [Robot Framework](https://robotframework.org/)

## UI translation

Translated with [Weblate](https://hosted.weblate.org/projects/ns8/).

To setup the translation process:

- add [GitHub Weblate app](https://docs.weblate.org/en/latest/admin/continuous.html#github-setup) to your repository
- add your repository to [hosted.weblate.org]((https://hosted.weblate.org) or ask a NethServer developer to add it to ns8 Weblate project
