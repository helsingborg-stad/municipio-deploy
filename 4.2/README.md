<p>
  <a href="https://github.com/helsingborg-stad/municipio-deploy">
    <img src="../images/hbg-github-logo-combo.png" alt="Logo" width="300">
  </a>
</p>

# 4.2 deploy action
This action will setup the build machine with php and node install all composer packages and run `php ./build.php`.  
The `build.php` is expected to perform a complete build of the site.  
All outcome from the build script is the pushed to a host with rsync based on below parameters.  
Lastly it will clear all nginx, object and blade cache.  

## Parameters

| Parameter name              | Description                                                                  | Default    | Required | Notes                                                      |
|-----------------------------|------------------------------------------------------------------------------|------------|----------|------------------------------------------------------------|
| deploy-host                 | Host domain or ip                                                            |            | true     |                                                            |
| deploy-port                 | Host ssh port                                                                | 22         | false    |                                                            |
| deploy-host-path            | Host deployment path                                                         |            | true     |                                                            |
| deploy-host-backup-path     | Host rsync backup path                                                       |            | true     |                                                            |
| deploy-host-user            | Host deploy ssh user name                                                    |            | true     |                                                            |
| deploy-host-user-key        | Host deploy ssh user key                                                     |            | true     |                                                            |
| deploy-host-web-server-user | Host web server user                                                         |            | true     |                                                            |
| deploy-host-pagecache-path  | The path for LS PageCache. If set, files in here will be deleted on deploy.  |            | false    |                                                            |
| kill-lsphp                  | Whether or not to kill all LSPHP processes.                                  | false      | false    |                                                            |
| github-token                | Github token for github npm package usage, use built in secrets.GITHUB_TOKEN |            | true     |                                                            |
| acf-key                     | ACF PRO license key used to authenticate Composer.                           |            | true     |                                                            |
| php-version                 | PHP Version                                                                  | 8.2        | false    |                                                            |
| newrelic-app-id             | App ID for deployment tracking with newrelic                                 |            | false    | Needs to be set with newrelic-api-key to execute the step. |
| newrelic-api-key            | API Key for deployment tracking with newrelic                                |            | false    | Needs to be set with newrelic-app-id to execute the step.  |
| sentry-url                  | Base url to sentry instance                                                  |            | false    | Needs to be set with other sentry vars to execute the step |
| sentry-org                  | The Organization Slug the project is at                                      |            | false    | Needs to be set with other sentry vars to execute the step |
| sentry-version              | The version identifier of the release to track the deploy for                |            | false    | Needs to be set with other sentry vars to execute the step |
| sentry-api-key              | The API key to use with sentry                                               |            | false    | Needs to be set with other sentry vars to execute the step |
| sentry-env                  | The enviroment to track the deploy for                                       |            | false    | Needs to be set with other sentry vars to execute the step |
| tideways-api-key            | The API key to use with sentry                                               |            | false    | Needs to be set with other sentry vars to execute the step |
| tideways-env                | The enviroment to track the deploy for                                       |            | false    | Needs to be set with other tideways vars to execute the step |


## Backups
The rsync command is run with the --backup flag, any changed files on each deploy will be compresses and stored in the path selected in the deploy-host-backup-path parameter as {{github SHA}}.tar.gz.
A delete step will remove any backup older than 7 days. As the step is running from this action no files will be deleted unless this action is being triggered.

## Secrets
All parameters are sensitive information and should ONLY use [Github secrets.](https://docs.github.com/en/actions/security-guides/encrypted-secrets).  
Secrets can be added on organisation level spanning all repositories in the org or on a repository level where Github actions in the current repository will be the only one able to read it.

## Sentry

For information on configuring the Sentry ping, please refer to [Sentrys own documentation](https://docs.sentry.io/api/releases/create-a-new-deploy-for-an-organization/)

## Tideways

For information on configuring the Tideway ping, please refer to [Tideways own documentation](https://support.tideways.com/documentation/setup/configuration/releases.html)

## Usage
In your Municipio 4.0 site repository:
- Create a file in `.github/workflows/` named ex. `deploy-branchname.yml`.
- Use the following template.
```
name: Build and deploy beta.

on:
  push:
    branches: [ branchname ]
    paths-ignore:
      - .github/**

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2

    - uses: helsingborg-stad/municipio-deploy/4.2@master
      with:
        deploy-host: ${{ secrets.DEPLOY_REMOTE_HOST_DOMAIN_SE }}
        deploy-host-path: ${{ secrets.DEPLOY_REMOTE_PORT_DOMAIN_SE }}
        deploy-host-path: ${{ secrets.DEPLOY_REMOTE_PATH_DOMAIN_SE }}
        deploy-host-backup-path: ${{ secrets.DEPLOY_REMOTE_BACKUP_DIR_DOMAIN_SE }}
        deploy-host-user: ${{ secrets.DEPLOY_REMOTE_USER }}
        deploy-host-user-key: ${{ secrets.DEPLOY_KEY }}
        deploy-host-web-server-user: ${{ secrets.WEB_SERVER_USER }}
        github-token: ${{ secrets.GITHUB_TOKEN }}
        acf-key: ${{ secrets.ACF_PRO_KEY }}
  ```
- Make sure branchname is matching the branch you would want to deploy.
- Replace secret names with macthing secrets.
