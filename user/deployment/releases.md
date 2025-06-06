---
title: GitHub Releases Uploading
layout: en
deploy: v1

---

Travis CI can automatically upload assets to git tags on your GitHub repository.

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

This configuration will use the "GITHUB OAUTH TOKEN" to upload "FILE TO UPLOAD"
(relative to the working directory) on tagged builds.

> Make sure you have `skip_cleanup` set to `true`, otherwise Travis CI will delete all the files created during the build, which will probably delete what you are trying to upload.

GitHub Releases works with git tags, so it is important that
you understand how tags affect GitHub Releases.

## Deploy on tagged builds

With [`on.tags: true`](/user/deployment/#conditional-releases-with-on),
your Releases deployment will trigger if and only if the build is a tagged
build.

## Regular releases

When the `draft` option is not set to `true` (more on this below), a regular
release is created.
Regular releases require tags.
If you set `on.tags: true` (as the initial example in this document), this
requirement is met.

## Draft releases 

For Draft releases using `draft: true`, use the following code:

```yaml
deploy:
  provider: releases
  api_key: "GITHUB OAUTH TOKEN"
  file: "FILE TO UPLOAD"
  skip_cleanup: true
  draft: true
```
{: data-file=".travis.yml"}

the resultant deployment is a draft Release that only repository collaborators
can see.
This gives you an opportunity to examine and edit the draft release.

## Set the tag at deployment

GitHub Releases need the present commit to be tagged at the deployment time.
If you set `on.tags: true`, the commit is guaranteed to have a tag.

Depending on the workflow, however, this is not desirable.

In such cases, it is possible to postpone setting the tag until
you have all the information you need.
A natural place to do this is `before_deploy`.
For example:

```yaml
    before_deploy:
      # Set up git user name and tag this commit
      - git config --local user.name "YOUR GIT USER NAME"
      - git config --local user.email "YOUR GIT USER EMAIL"
      - export TRAVIS_TAG=${TRAVIS_TAG:-$(date +'%Y%m%d%H%M%S')-$(git log --format=%h -1)}
      - git tag $TRAVIS_TAG
    deploy:
      provider: releases
      api_key: "GITHUB OAUTH TOKEN"
      file: "FILE TO UPLOAD"
      skip_cleanup: true
```
{: data-file=".travis.yml"}

### Tag not set during deployment

If the tag is still not set at the time of deployment, the deployment
provider attempts to match the current commit with a tag from remote,
and if one is found, uses it.

This could be a problem if multiple tags are assigned to the current commit and
the one you want is not matched.
In such a case, assign the tag you need (the method will depend on your use
case) to `$TRAVIS_TAG` to get around the problem.

If the build commit does not match any tag at deployment time, GitHub creates one
when the release is created.
The GitHub-generated tags are of the form `untagged-*`, where `*` is a random
hex string.
Notice that this tag is immediately available on GitHub, and thus
will trigger a new Travis CI build, unless it is prevented by
other means; for instance, by
[blocklisting `/^untagged/`](/user/customizing-the-build/#safelisting-or-blocklisting-branches).

## Overwrite existing files on the release

If you need to overwrite existing files, add `overwrite: true` to the `deploy` section of your `.travis.yml`.

## Populate the initial deployment configuration with Travis CI

You can also use the [Travis CI command line client](https://github.com/travis-ci/travis.rb#installation) to configure your `.travis.yml`:

```bash
travis setup releases
```

Or, if you're using a private repository or the GitHub Apps integration:

```bash
travis setup releases --com
```

## OAuth token Authentication 

The recommended way to authenticate is to use a GitHub OAuth token. Instead of setting it up manually, it is highly recommended to use `travis setup releases`, which automatically creates and encrypts a GitHub OAuth token with the correct scopes.

If you can't use `travis setup releases`, you can set up the token manually with the following steps:
1. Create a personal access token on the Github account. It must have the `public_repo` or `repo` scope to upload assets.
2. Encrypt the token using Travis CLI: `travis encrypt [super_secret_token]`. Note that you must _not_ give the token a name in the encrypt command, as you might for an environment variable.
3. Add the secure encrypted token to the deploy section of your `.travis.yml`, under the `api_key`.

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

**Warning:** the `public_repo` and `repo` scopes for GitHub OAuth tokens grant write access to all of a user's (public) repositories. For security, it's ideal for `api_key` to have the write access limited to only repositories where Travis deploys to GitHub releases. The suggested workaround is to create a [machine user](https://developer.github.com/v3/guides/managing-deploy-keys/#machine-users) — a dummy GitHub account that is granted write access on a per-repository basis.

## Username and Password Authentication

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

## Deploy to GitHub Enterprise

If you wish to upload assets to a GitHub Enterprise repository, you must override the `$OCTOKIT_API_ENDPOINT` environment variable with your GitHub Enterprise API endpoint:

```
http(s)://"GITHUB ENTERPRISE HOSTNAME"/api/v3/
```

You can configure this in [Repository Settings](/user/environment-variables/#defining-variables-in-repository-settings) or via your `.travis.yml`:

```yaml
env:
  global:
    - OCTOKIT_API_ENDPOINT="GITHUB ENTERPRISE API ENDPOINT"
```
{: data-file=".travis.yml"}

## Upload Multiple Files

You can upload multiple files using the yml array notation. This example uploads two files.

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
  api_key: "GITHUB OAUTH TOKEN"
  file_glob: true
  file: directory/*
  skip_cleanup: true
  on:
    tags: true
```
{: data-file=".travis.yml"}

You can use the glob pattern to recursively find the files:

```yaml
deploy:
  provider: releases
  api_key: "GITHUB OAUTH TOKEN"
  file_glob: true
  file: directory/**/*
  skip_cleanup: true
  on:
    tags: true
```
{: data-file=".travis.yml"}

Please note that all paths in `file` are relative to the current working directory, not to [`$TRAVIS_BUILD_DIR`](/user/environment-variables/#default-environment-variables).

### Conditional releases

You can deploy only when certain conditions are met.
See [Conditional Releases with `on:`](/user/deployment/#conditional-releases-with-on).

## Run Commands Before or After Release

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

## Advanced options

The following options from `.travis.yml` are passed through to Octokit API's
[#create_release](https://octokit.github.io/octokit.rb/Octokit/Client/Releases.html#create_release-instance_method)
and [#update_release](https://octokit.github.io/octokit.rb/Octokit/Client/Releases.html#update_release-instance_method) methods.

* `tag_name`
* `target_commitish`
* `name`
* `body`
* `draft` (boolean)
* `prerelease` (boolean)

Note that formatting in `body` is [not preserved](https://github.com/travis-ci/dpl/issues/155).

## Troubleshoot Git Submodules

GitHub Releases executes a number of git commands during deployment. For this reason, it is important that the working directory is set to the one for which the release will be created, which generally isn't a problem, but if you clone another repository during the build or use submodules, it is worth double-checking.

