# \[Site name goes here\]
A Drupal 10 website powering \[Production URL goes here\], a website about \[something\].

Dev site: http://\[site name\].pantheonsite.io/

* [Local setup](#setup)
* [Making site changes](#changes)
* [Local destruction](#removal)
* [Cron commands](#cron)
* [Deployment to Pantheon via Github actions](#deployment)

## Local site setup with DDEV <a name="setup"></a>

Install [homebrew](https://brew.sh/), [docker](https://www.docker.com/products/docker-desktop), and DDEV if you haven't already. DDEV install instructions are [here](https://ddev.readthedocs.io/en/stable/#homebrew-macoslinux), but the gist of it is to open a terminal window and run `brew install drud/ddev/ddev`

1. Start Docker desktop.

2. Get the code.
   * `git clone` (this URL)
   * `cd` (into project directory)
   * `git checkout main`

3. Configure DDEV.
   * `ddev config --project-type=drupal10 --docroot=web --project-name=mysite`

4. Start DDEV.
    * `ddev start`<br><br>

    > **Note**
    >
    > **If you get an error about ports**, manually edit .ddev/config.yaml so router_http_port and router_https_port are set to unused ports on your machine. Then run ddev start again.

5. Build with composer.
   * `ddev composer install`

6. Import site database, clear caches, and apply site configuration from code.
   * `ddev import-db --src=/mysite.sql.gz`
   * `ddev drush cr`
   * `ddev drush cim -y`

7. In a browser, visit your fresh local copy at http://mysite.ddev.site
(Include the port from step 4 if needed,  or use `ddev launch`)

### Local site setup troubleshooting/tips <a name="troubleshooting"></a>
* `ddev list` to find and selectively stop all other ddev projects with conflicting names or ports
* `ddev poweroff` to shut down ddev and all of its projects

## Making site changes <a name="changes"></a>

### Configuration changes

Drupal 10 configuration (such as content type and field definitions) is managed through yml files. These files live in the `config` directory.

After you make configuration changes locally (such as adding a field to a content type), you can then export those changes by running `ddev drush cex` in the project directory.

> **Warning**
>
> Before you change or export your site configuration, be sure to `git pull; git merge origin/main` and run `ddev drush cim` to get any recent configuration updates that were added upstream by other developers. This also will prevent conflicts when you export and commit your changes.

### Installing modules, updating Drupal and/or contributed modules <a name="ql"></a>

* To install a module: run `ddev composer require drupal/[module name]` from the project root directory.
* To run a drush command: `ddev drush [drush command]`
* Show all outdated packages: `ddev composer outdated "drupal/*"`
* Update Drupal core to latest: `ddev composer update "drupal/core-*" --with-all-dependencies`


## Local setup destruction <a name="removal"></a>
* Temporary; keep databases: `ddev stop`
* For ddev rebuild; remove databases: `ddev stop --remove-data --omit-snapshot`
* For ddev project obliteration: `ddev delete --omit-snapshot --yes`
* For total destruction, run the above command and then `rm` the project directory. Then reclone and run through these steps again (or not).


## Cron commands <a name="cron"></a>

Pantheon automatically executes cron once an hour (on non-suspended sites) without any configuration. See [Pantheon's cron documentation](https://docs.pantheon.io/drupal-cron) for more details.

## Deployment to Pantheon via Github Actions <a name="deployment"></a>

This site's main branch is automatically committed and deployed to Pantheon's dev environment with a [Github action workflow file](.github/workflows/deploy-to-pantheon.yml). The pantheon site name and ID are configured in that file.

It needs these secrets added to this repository's Github settings:
* **PANTHEON_SSH_PRIVATE_KEY**
<br>(A Private SSH key paired to a [Public SSH key added to Pantheon](https://docs.pantheon.io/ssh-keys))
* **TERMINUS_TOKEN**
<br>(A generated [Pantheon Machine token](https://docs.pantheon.io/machine-tokens))
