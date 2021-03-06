---
title: GitHub Releases Uploading
layout: en

---

Travis CI can automatically upload assets from your [`$TRAVIS_BUILD_DIR`](/user/environment-variables/#Default-Environment-Variables) to git tags on your GitHub repository.

**Note that deploying GitHub Releases works only for tags, not for branches.**

For a minimal configuration, add the following to your `.travis.yml`:

```yaml
deploy:
  provider: releases
  api_key: "GITHUB OAUTH TOKEN"
  file: "FILE TO UPLOAD"
  skip_cleanup: true
  on:
    tags: true
```
{: data-file=".travis.yml"}

> Make sure you have `skip_cleanup` set to `true`, otherwise Travis CI will delete all the files created during the build, which will probably delete what you are trying to upload.

The `on: tags: true` section at the end of the `.travis.yml` above is required to make sure that your tags get deployed.

If you need to overwrite existing files, add `overwrite: true` to the `deploy` section of your `.travis.yml`.

You can also use the [Travis CI command line client](https://github.com/travis-ci/travis.rb#installation) to configure your `.travis.yml`:

```bash
travis setup releases
```

Or, if you're using a private repository:

```bash
travis setup releases --pro
```

If you are using the [`branches.only` property](/user/customizing-the-build#Building-Specific-Branches), remember that when you push a tag, the [`$TRAVIS_BRANCH` property](/user/environment-variables/#Default-Environment-Variables) contains the name of the tag. As a result, edit the `branches.only` property to add the names of the tags you might push in the future. You can use a regular expression if you have formalized names. For example, if your release tags look like  `v1.3.15`, use the following configuration: 

```yaml
   branches:
    only:
    - master
    - /^v\d+(\.\d+)+$/
```



## Authenticating with an OAuth token

The recommended way to authenticate is to use a GitHub OAuth token. It must have the `public_repo` or `repo` scope to upload assets. Instead of setting it up manually, it is highly recommended to use `travis setup releases`, which automatically creates and encrypts a GitHub oauth token with the correct scopes.

This results in something similar to:

```yaml
deploy:
  provider: releases
  api_key:
    secure: YOUR_API_KEY_ENCRYPTED
  file: "FILE TO UPLOAD"
  skip_cleanup: true
  on:
    tags: true
```
{: data-file=".travis.yml"}

## Authentication with a Username and Password

You can also authenticate with your GitHub username and password using the `user` and `password` options. This is not recommended as it allows full access to your GitHub account but is simplest to setup. It is recommended to encrypt your password using `travis encrypt "GITHUB PASSWORD" --add deploy.password`. This example authenticates using  a username and password.

```yaml
deploy:
  provider: releases
  user: "GITHUB USERNAME"
  password: "GITHUB PASSWORD"
  file: "FILE TO UPLOAD"
  skip_cleanup: true
  on:
    tags: true
```
{: data-file=".travis.yml"}

## Deploying to GitHub Enterprise

If you wish to upload assets to a GitHub Enterprise repository, you must override the `$OCTOKIT_API_ENDPOINT` environment variable with your GitHub Enterprise API endpoint:

```
http(s)://"GITHUB ENTERPRISE HOSTNAME"/api/v3/
```

You can configure this in [Repository Settings](https://docs.travis-ci.com/user/environment-variables/#Defining-Variables-in-Repository-Settings) or via your `.travis.yml`:

```yaml
env:
  global:
    - OCTOKIT_API_ENDPOINT="GITHUB ENTERPRISE API ENDPOINT"
```
{: data-file=".travis.yml"}

## Uploading Multiple Files

You can upload multiple files using yml array notation. This example uploads two files.

```yaml
deploy:
  provider: releases
  api_key:
    secure: YOUR_API_KEY_ENCRYPTED
  file:
    - "FILE 1"
    - "FILE 2"
  skip_cleanup: true
  on:
    tags: true
```
{: data-file=".travis.yml"}

You can also enable wildcards by setting `file_glob` to `true`. This example
includes all files in a given directory.

```yaml
deploy:
  provider: releases
  api-key: "GITHUB OAUTH TOKEN"
  file_glob: true
  file: directory/*
  skip_cleanup: true
  on:
    tags: true
```
{: data-file=".travis.yml"}

### Conditional releases

You can deploy only when certain conditions are met.
See [Conditional Releases with `on:`](/user/deployment#Conditional-Releases-with-on%3A).

## Running commands before or after release

Sometimes you want to run commands before or after releasing a gem. You can use the `before_deploy` and `after_deploy` stages for this. These will only be triggered if Travis CI is actually pushing a release.

```yaml
before_deploy: "echo 'ready?'"
deploy:
  ..
after_deploy:
  - ./after_deploy_1.sh
  - ./after_deploy_2.sh
```
{: data-file=".travis.yml"}

## Pushing a specific directory

* `local_dir`: Directory to push to GitHub Releases, defaults to the current
    directory

## Advanced options

Options from `.travis.yml` are passed through to [Octokit API](https://octokit.github.io/octokit.rb/Octokit/Client/Releases.html#create_release-instance_method), so you can use any valid Octokit option.
