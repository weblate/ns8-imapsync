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
trashsync: true, false
-  delete mails and folder on local account not in remote account
delete: true, false
- exclude folders prepend by `^` and end by `$`
exclude: folder1,folder2,^folder3$
- the remote imap account credentials (you can use the password of the imap master administrator if you know it, for example vmail. use the login user*vmail and the relevant password). Otherwise use the login and the password of the remote account.

Example:

    api-cli run module/imapsync1/create-task --data '{
        "localuser":"administrator",
        "remotehostname":"imap.foo.com",
        "remoteport":"143",
        "security":"tls",
        "delete":"enabled",
        "trashsync":"disabled",
        "exclude":"folder1,folder2",
        "remoteusername":"username",
        "remotepassword":"password"
    }'

## delete env and stop a running synchronisation

Example:

    api-cli run module/imapsync1/delete-task --data '{
        "localuser":"administrator"
    }'

## stop a running synchronisation

Example:

    api-cli run module/imapsync1/stop-task --data '{
        "localuser":"administrator"
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
{
  "mail_server": "e8a6177c-9ae5-4356-826b-0a5f93b2dbaf",
  "mail_server_URL": [
    {
      "name": "mail2",
      "label": "mail2 (R3.rocky9-3.org)",
      "value": "e8a6177c-9ae5-4356-826b-0a5f93b2dbaf,R3.rocky9-3.org"
    }
  ],
  "user_properties": [
    {
      "props": {
        "localuser": "administrator",
        "remoteusername": "username",
        "remotehostname": "imap.foo.com",
        "remoteport": "143",
        "security": "tls",
        "delete": true,
        "trashsync": true,
        "exclude": "folder1|folder2",
        "remotepassword": "password"
      },
      "mailbox": "administrator",
      "service_running": false
    },
    {
      "props": {
        "localuser": "stephane",
        "remoteusername": "",
        "remotehostname": "",
        "remoteport": "",
        "security": "",
        "delete": false,
        "trashsync": false,
        "exclude": "",
        "remotepassword": ""
      },
      "mailbox": "stephane",
      "service_running": false
    }
  ]

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
