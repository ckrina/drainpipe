# Drainpipe

Drainpipe is a composer package which provides a number of build tool helpers
for a Drupal site, including:

- Site and database updates
- Artifact packaging for deployment to a hosting provider
- Automated testing setup with support for PHPUnit and Nightwatch tests

## Installation

```sh
composer require lullabot/drainpipe
# Includes development dependencies, but only in the `require-dev` section. This step is required for Drainpipe to provide test helpers.
composer require lullabot/drainpipe-dev --dev
# To compile Sass and Javascript via their respective tasks, add these node packages.
PACKAGES="
@lullabot/drainpipe-sass
@lullabot/drainpipe-javascript
"
if [ -f yarn.lock ]; then
  yarn add $PACKAGES
else
  npm install $PACKAGES
fi
```

Drainpipe integrates with [DDEV](https://ddev.readthedocs.io/en/stable/), but
will only add the relevant files when DDEV is detected in the repository. Either
set DDEV up first before requiring this project, or run `composer update` if
DDEV is added later.

```sh
composer create-project drupal/recommended-project drupal
cd drupal
ddev config
ddev start
ddev composer require lullabot/drainpipe
ddev composer require lullabot/drainpipe-dev --dev
# Restart is required to enable the provided Selenium containers
ddev restart
```

## Usage

Build tasks are provided as [Taskfiles](https://taskfile.dev/#/). A full list of
available tasks can be shown by running `./vendor/bin/task --list` (or
`ddev task --list` if you're using DDEV).

### Running Tests

See https://github.com/Lullabot/drainpipe-dev

### Defining Browser compatibility

The best way to specify the browsers to target in the project is a `.browserslistrc` file in the project root. See https://github.com/postcss/autoprefixer
for more info.

### CSS & JS asset automation

Drainpipe provides tasks to automate Sass & JavaScript compilation.

To enable this, first define the project variables `DRAINPIPE_SASS` and/or
`DRAINPIPE_JAVASCRIPT` in `Taskfile.yml`.

Then define the task:
```
assets:
  desc: Builds assets such as CSS & JS
  cmds:
    - if [ -f "yarn.lock" ]; then yarn; else npm install; fi
    - task: javascript:compile
    - task: sass:compile
assets:watch:
  desc: Builds assets such as CSS & JS, and watches them for changes
  deps: [sass:watch, javascript:watch]
```

You can then run the following tasks:
- `ddev task assets` Default task to compile the CSS and JS once for production.
- `ddev task assets:watch` Task to watch and build the changes while working on the theme.

#### CSS

The task provided to compile CSS assets uses [Sass](https://sass-lang.com/).

It includes SASS Glob to use glob imports to define a whole directory:

```
// Base
@use "sass/base/**/*";
```

## Validation

Your `Taskfile.yml` can be validated with JSON Schema:
```
curl -O https://json.schemastore.org/taskfile.json
npx ajv-cli validate -s taskfile.json -d Taskfile.yml
```

See [.github/workflows/validate-taskfile.yml](`.github/workflows/validate-taskfile.yml`)
for an example of this in use.

## GitLab Integration

Add the following to `composer.json` for GitLab helpers:
```json
"extra": {
  "drainpipe": {
    "gitlab": []
  }
}
```

This will import [`scaffold/gitlab/Common.gitlab-ci.yml`](scaffold/gitlab/Common.gitlab-ci.yml),
which provides helpers that can be used in GitLab CI with [includes and
references](https://docs.gitlab.com/ee/ci/yaml/yaml_specific_features.html#reference-tags).

### Pantheon
```json
"extra": {
    "drainpipe": {
        "gitlab": ["Pantheon", "Pantheon Review Apps"]
    }
}
```

This will setup Merge Request deployment to Pantheon Multidev environments. See
[scaffold/gitlab/gitlab-ci.example.yml] for an example. You can also just
include which will give you helpers that you can include and reference for tasks
such as setting up [Terminus](https://pantheon.io/docs/terminus). See
[scaffold/gitlab/Pantheon.gitlab-ci.yml](scaffold/gitlab/Pantheon.gitlab-ci.yml).

### Composer Lock Diff
```json
"extra": {
    "drainpipe": {
        "gitlab": ["ComposerLockDiff"]
    }
}
```

Updates Merge Request descriptions with a markdown table of any changes detected
in `composer.lock` using [composer-lock-diff](https://github.com/davidrjonas/composer-lock-diff).
Requires `GITLAB_ACCESS_TOKEN` variable to be set, which is an access token with
`api` scope.

## GitHub Actions Integration

## Pantheon

- Add the following the composer.json to enable deployment of Pantheon Review Apps
  ```json
  "extra": {
      "drainpipe": {
          "github": ["PantheonReviewApps"]
      }
  }
  ```
- Run `composer install`
- Add your Pantheon `site-name` and `site-id` to the last job in the new
  workflow file at `.github/workflows/PantheonReviewApps.yml`
- Add the following secrets to your repository:
  - `PANTHEON_TERMINUS_TOKEN` See https://pantheon.io/docs/terminus/install#machine-token
  - `SSH_PRIVATE_KEY` A private key of a user which can push to Pantheon
  - `SSH_KNOWN_HOSTS` The result of running `ssh-keyscan -H codeserver.dev.$PANTHEON_SITE_ID.drush.in`
