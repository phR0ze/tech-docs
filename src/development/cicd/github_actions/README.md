# Github Actions

Actions is a completely free feature for github open source projects. Actions has the advantage of
being 100% programmatically driven which is a slick feature.

References:
* [Workflows](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/workflow-syntax-for-github-actions)
* [Trigger Events](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/events-that-trigger-workflows#webhook-events)

### Quick links
* [Terms](#terms)
* [Runners](#runners)
* [Workflows](#workflows)
* [Add a Status Badge](#add-a-status-badge)
* [Dump all contexts](#dump-all-contexts)

## Terms
* ***Steps*** are the most primitive types and run a command or an action.
* ***Actions*** are precanned composed of multiple commands that combine to form jobs.
* ***Jobs*** - each job runs in a specified container environment and combine to form workflows.
* ***Workflows*** are defined in YAML and composed of one or more jobs.

## Runners
Github actions provide a number of public runners to execute your code on. These are automatically 
provisioned VMs based on the runner template you choose. The runners are chosen by specifying a 
runner name. Unfortunately the runner names are not very well documented and not very intuitive to 
follow when it comes to MacOS choices.

| Runner            | Purpose
| ----------------- | --------------------------------------------------------------------------
| macos-13          | MacOS 13 based on x86 hardware
| macos-13-xlarge   | MacOS 13 based on ARM M1 hardware
| ubuntu-latest     | Ubuntu VM versioned at Github Action's latest offering
| windows-latest    | Windows VM versioned at Github Action's latest offering

## Workflows
When a job begins, Github automatically provisions a new VM for that job. All steps in the job 
execute on the VM, allowing the steps in that job to share information using the runner's filesystem.
You can run workflows directly on the VM or in a Docker container. When the job is finished, the VM 
is automatically decommissioned.

* People with write or admin permissions to a repository can create, edit or view workflows. Workflows
  are custom automated process that you can set up in your repository to build, test, package, release
  or deploy any project on GitHub. GitHub has pre-built Actions store similar to CodeFresh. These
  reusable steps seem to be the next wave of CI/CD hotness. TravisCI and older platforms don't have
* Each `workflow` must have at least one `job` and a job must contain at least one `step`. Steps run
  commands or use an `action`.

**Useful actions to use in steps:**
* [Setup Go Env](https://github.com/marketplace/actions/setup-go-environment)

## Add a Status Badge
https://help.github.com/en/actions/automating-your-workflow-with-github-actions/configuring-a-workflow#adding-a-workflow-status-badge-to-your-repository

You must call out the workflow that the status should be associated with and if your workflow uses
the `name` keyword you need to use that else use the full path of the file. And if you have spaces in
the name you need to encode that.

Example:
```
# template
![](https://github.com/<OWNER>/<REPOSITORY>/workflows/<WORKFLOW_NAME>/badge.svg)

# template example
![](https://github.com/my-owner/hello-world/workflows/.github/workflows/build.yaml/badge.svg)

# Working with workflow name called out from `name` field
https://github.com/phR0ze/alpine-base/workflows/Build%20Docker%20Image/badge.svg

# Working example simply using the root filename of workflow
![build](https://github.com/phR0ze/witcher/workflows/build/badge.svg?branch=main)

[![build-badge](https://github.com/phR0ze/alpine-base/workflows/build/badge.svg)](https://github.com/phR0ze/alpine-base/actions)
```

## Dump all contexts
```
on: push

jobs:
  one:
    runs-on: ubuntu-latest
    steps:
      - name: Dump GitHub context
        env:
          GITHUB_CONTEXT: ${{ toJson(github) }}
        run: echo "$GITHUB_CONTEXT"
      - name: Dump job context
        env:
          JOB_CONTEXT: ${{ toJson(job) }}
        run: echo "$JOB_CONTEXT"
      - name: Dump steps context
        env:
          STEPS_CONTEXT: ${{ toJson(steps) }}
        run: echo "$STEPS_CONTEXT"
      - name: Dump runner context
        env:
          RUNNER_CONTEXT: ${{ toJson(runner) }}
        run: echo "$RUNNER_CONTEXT"
      - name: Dump strategy context
        env:
          STRATEGY_CONTEXT: ${{ toJson(strategy) }}
        run: echo "$STRATEGY_CONTEXT"
      - name: Dump matrix context
        env:
          MATRIX_CONTEXT: ${{ toJson(matrix) }}
        run: echo "$MATRIX_CONTEXT"
```
