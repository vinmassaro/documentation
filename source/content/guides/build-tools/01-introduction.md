---
title: Build Tools
subtitle: Introduction
description: Create a site that manages its files using Composer, and uses a GitHub PR workflow with Behat tests run via Circle CI.
tags: [automate, composer]
contributors: [greg-1-anderson, stevector, ataylorme, rachelwhitton]
layout: guide
type: guide
anchorid: build-tools
buildtools: true
generator: pagination
pagination:
    provider: data.buildtoolspages
use:
    - buildtoolspages
permalink: docs/guides/build-tools/
nexturl: guides/build-tools/create-project/
editpath: build-tools/01-introduction.md
image: buildToolsGuide-thumb
multidev: true
---
This guide describes how to use build tools such as GitHub and CircleCI with Composer to implement a collaborative, team-based Continuous Integration workflow using Pull Requests for Drupal 8 sites on Pantheon. While this guide demonstrates [Drupal 8](https://github.com/pantheon-systems/example-drops-8-composer), the same workflow can be applied to [WordPress](https://github.com/pantheon-systems/example-wordpress-composer) and [Drupal 7](https://github.com/pantheon-systems/example-drops-7-composer) sites.

<div class="flex-panel-group">
  <div class="flex-panel-item">
    <div class="flex-panel-body">
      <div class="flex-panel-title">
        <h4 class="info" style="margin-top:10px;font-size:larger">GitHub</h4>
        <div class="pantheon-official">
          <img alt="GitHub Logo" src="/source/docs/assets/images/github-logo.svg" class="main-topic-info__plugin-image" style="max-width:40px;margin-bottom:10px!important;" />
          <p class="pantheon-official"></p>
        </div>
      </div>
      <p class="topic-info__description" >[GitHub](https://github.com) is an online service that provides cloud storage Git repositories that may be cloned and used locally, or edited directly through their web-based management interface. These features are very useful to teams collaborating on a project together.</p>
    </div>
  </div>
  <div class="flex-panel-item">
    <div class="flex-panel-body">
      <div class="flex-panel-title">
        <h4 class="info" style="margin-top:10px;font-size:larger">CircleCI</h3>
        <div class="pantheon-official">
          <img alt="CircleCI Logo" src="/source/docs/assets/images/circleci-logo.svg" class="main-topic-info__plugin-image" style="max-width:40px;margin-bottom:10px!important;" />
          <p class="pantheon-official"></p>
        </div>
      </div>
      <p class="topic-info__description" >[CircleCI](https://circleci.com) provides hosted services to run automated tests for a project, and GitHub provides an integration to run these tests to whenever a change is submitted. The process of testing each set of changed files prior to merging them into the main branch is called continuous integration.</p>
    </div>
  </div>
  <div class="flex-panel-item">
    <div class="flex-panel-body">
      <div class="flex-panel-title">
        <h4 class="info" style="margin-top:10px;font-size:larger">Composer</h3>
        <div class="pantheon-official">
          <img alt="Composer Logo" src="/source/docs/assets/images/composer-logo.svg" class="main-topic-info__plugin-image" style="max-width:40px;margin-bottom:10px!important;" />
          <p class="pantheon-official"></p>
        </div>
      </div>
      <p class="topic-info__description"><a href="/docs/composer/">Composer</a> is a PHP dependency manager that provides an alternative, more modern way to manage the external code used by a project. For example, Composer may be used to install the plugins, modules and themes used by a Drupal or WordPress site.</p>
    </div>
  </div>
</div>

<Enablement title="Automation Training" link="https://pantheon.io/agencies/learn-pantheon?docs">

Master Composer concepts with help from our experts. Pantheon delivers custom workshops to help development teams master the platform and improve internal DevOps.

</Enablement>

## Artifact Deployment
Only files unique to the project are tracked as part of the project's main "source" repository on GitHub, which requires an abstraction layer to compile dependencies and deploy an entire "artifact" to the site repository on Pantheon. The abstraction layer is facilitated by CircleCI in the Pantheon maintained examples, but the principles are the same for other continuous integration service providers.

Composer is used to fetch dependencies declared by the project as part of a CircleCI build step. This ensures that the final composed build results are installed on Pantheon:

<p class="text-center" >![Artifact Deployment](/source/docs/assets/images/artifact-deployment.png)</p>

<Accordion title={"Pull Requests"} id={"understand-pr"} icon={"lightbulb"}>
One advantage of managing code this way is that it keeps the change sets (differences) for pull requests as small as possible. If a pull request upgrades several dependencies, only the dependency metadata file will change; the actual code changes in the upgraded dependencies themselves are not shown.

GitHub pull requests (PRs) are a formalized way of reviewing and merging a proposed set of changes to the source repository. When one member of a development team makes changes to a project, all of the files modified to produce the feature are committed to a separate branch, and that branch becomes the basis for the pull request. GitHub allows other team members to review all of the differences between the new files and their original versions, before merging the PR to accept changes.
</Accordion>

## Before You Begin

1. Install [Composer](https://getcomposer.org).
2. Install the most recent release of [Terminus](/docs/terminus/):

    <div class="copy-snippet">
      <button class="btn btn-default btn-clippy" data-clipboard-target="#terminus-installer">Copy</button>
      <figure><pre id="terminus-installer"><code class="command bash" data-lang="bash">curl -O https://raw.githubusercontent.com/pantheon-systems/terminus-installer/master/builds/installer.phar && php installer.phar install</code></pre></figure>
    </div>

3. [Add an SSH key](/docs/ssh-keys/) within your User Dashboard to enable passwordless access and avoid authentication prompts. Otherwise, provide your Pantheon Dashboard credentials when prompted.

4. [Generate a Machine Token](https://dashboard.pantheon.io/machine-token/create), then authenticate Terminus:

      <div class="copy-snippet">
        <button class="btn btn-default btn-clippy" data-clipboard-target="#mac-mt-auth">Copy</button>
        <figure><pre id="mac-mt-auth"><code class="command bash" data-lang="bash">terminus auth:login --machine-token=&lsaquo;machine-token&rsaquo;</code></pre></figure>
      </div>

5. Create the `$HOME/.terminus/plugins` directory if it does not already exist:

      <div class="copy-snippet">
        <button class="btn btn-default btn-clippy" data-clipboard-target="#terminus-plugin-install-mkdir">Copy</button>
        <figure><pre id="terminus-plugin-install-mkdir"><code class="command bash" data-lang="bash">mkdir -p $HOME/.terminus/plugins</code></pre></figure>
      </div>

6. Install the [Terminus Composer Plugin](https://github.com/pantheon-systems/terminus-composer-plugin):

    <div class="copy-snippet">
      <button class="btn btn-default btn-clippy" data-clipboard-target="#composer-plugin">Copy</button>
      <figure><pre id="composer-plugin"><code class="command bash" data-lang="bash">composer create-project -n -d $HOME/.terminus/plugins pantheon-systems/terminus-composer-plugin:~1</code></pre></figure>
    </div>

7. Install the [Terminus Drupal Console Plugin](https://github.com/pantheon-systems/terminus-drupal-console-plugin):

    <div class="copy-snippet">
      <button class="btn btn-default btn-clippy" data-clipboard-target="#console-plugin">Copy</button>
      <figure><pre id="console-plugin"><code class="command bash" data-lang="bash">composer create-project -n -d $HOME/.terminus/plugins pantheon-systems/terminus-drupal-console-plugin:~1</code></pre></figure>
    </div>

8. Install the [Terminus Build Tools Plugin](https://github.com/pantheon-systems/terminus-build-tools-plugin):

    <div class="copy-snippet">
      <button class="btn btn-default btn-clippy" data-clipboard-target="#build-tools-plugin">Copy</button>
      <figure><pre id="build-tools-plugin"><code class="command bash" data-lang="bash">composer create-project -n -d $HOME/.terminus/plugins pantheon-systems/terminus-build-tools-plugin:~1</code></pre></figure>
    </div>

    <Alert title="Note" type="info">
    The Terminus Build Tools Plugin does not support private repositories.</p>
    </Alert>

9. [Authorize CircleCI on GitHub](https://github.com/login/oauth/authorize?client_id=78a2ba87f071c28e65bb).

    If you are redirected to the CircleCI homepage, you have already authorized the service for your GitHub account. Nice! Way to be ahead of the game.

<Alert title="Note" type="info">
Pantheon's [support team](/docs/support/) cannot troubleshoot issues with third-party services like GitHub or CircleCI.

If you need help configuring external systems, consider joining the [Community Forum](https://discuss.pantheon.io/) or posting in our [Pantheon Power Users](https://slackin.pantheon.io/) Slack channel.
</Alert>


### Access Tokens (Optional)

The Build Tools plugin will prompt you to create access tokens for both [GitHub](https://github.com/settings/tokens) and [CircleCI](https://circleci.com/account/api), which are stored as environment variables. The GitHub token needs the **repo** (required) and **delete-repo** (optional) scopes. You may optionally generate these tokens ahead of time and manually export them to the local variables `GITHUB_TOKEN` and `CIRCLE_TOKEN`, respectively:

```bash
export GITHUB_TOKEN=yourGitHubToken
export CIRCLE_TOKEN=yourCircleCIToken
```

If you need to replace a token, navigate to your [project settings page in CircleCI](https://circleci.com/docs/2.0/env-vars/#adding-environment-variables-in-the-app).