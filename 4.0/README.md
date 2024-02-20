<p>
  <a href="https://github.com/helsingborg-stad/municipio-deploy">
    <img src="../images/hbg-github-logo-combo.png" alt="Logo" width="300">
  </a>
</p>

# 4.0 deploy action
This action will setup the build machine with php and node install all composer packages and run `php ./build.php`.  
The `build.php` is expected to perform a complete build of the site.  
All outcome from the build script is the pushed to a host with rsync based on below parameters.  
Lastly it will clear all nginx, object and blade cache.  

## Parameters

| Parameter name              | Description                                                                  | Default    | Required |
|-----------------------------|------------------------------------------------------------------------------|------------|----------|
| deploy-host                 | Host domain or ip                                                            |            | true     |
| deploy-port                 | Host ssh port                                                                | 22         | false    |
| deploy-host-path            | Host deployment path                                                         |            | true     |
| deploy-host-backup-path     | Host rsync backup path                                                       |            | true     |
| deploy-host-user            | Host deploy ssh user name                                                    |            | true     |
| deploy-host-user-key        | Host deploy ssh user key                                                     |            | true     |
| deploy-host-web-server-user | Host web server user                                                         |            | true     |
| github-token                | Github token for github npm package usage, use built in secrets.GITHUB_TOKEN |            | true     |
| acf-url                     | A url where a zip-file with ACF PRO can be found (ACF provides a url).       |            | true     |
| php-version                 | PHP Version                                                                  | 8.2        | false    |

## Backups
The rsync command is run with the --backup flag, any changed files on each deploy will be compresses and stored in the path selected in the deploy-host-backup-path parameter as {{github SHA}}.tar.gz.
A delete step will remove any backup older than 7 days. As the step is running from this action no files will be deleted unless this action is being triggered.

## Secrets
All parameters are sensitive information and should ONLY use [Github secrets.](https://docs.github.com/en/actions/security-guides/encrypted-secrets).  
Secrets can be added on organisation level spanning all repositories in the org or on a repository level where Github actions in the current repository will be the only one able to read it.

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

    - uses: helsingborg-stad/municipio-deploy/4.0@master
      with:
        deploy-host: ${{ secrets.DEPLOY_REMOTE_HOST_DOMAIN_SE }}
        deploy-host-path: ${{ secrets.DEPLOY_REMOTE_PORT_DOMAIN_SE }}
        deploy-host-path: ${{ secrets.DEPLOY_REMOTE_PATH_DOMAIN_SE }}
        deploy-host-backup-path: ${{ secrets.DEPLOY_REMOTE_BACKUP_DIR_DOMAIN_SE }}
        deploy-host-user: ${{ secrets.DEPLOY_REMOTE_USER }}
        deploy-host-user-key: ${{ secrets.DEPLOY_KEY }}
        deploy-host-web-server-user: ${{ secrets.WEB_SERVER_USER }}
        github-token: ${{ secrets.GITHUB_TOKEN }}
        acf-url: ${{ secrets.ACF_URL }}
  ```
- Make sure branchname is matching the branch you would want to deploy.
- Replace secret names with macthing secrets.
