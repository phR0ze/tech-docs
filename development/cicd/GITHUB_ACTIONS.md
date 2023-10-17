<img align="left" width="40" height="40" src="../../images/logo_256x256.png">Github Actions
====================================================================================================
Github Actions docuementation for various use cases I've bumped into
<br><br>

### Quick links
* [.. up dir](..)
* [Overview](#overview)
  * [Terms](#terms)
  * [Runners](#runners)
  * [Workflows](#workflows)
  * [Add a Status Badge](#add-a-status-badge)
  * [Dump all contexts](#dump-all-contexts)
* [OSX builds](#osx-builds)
  * [Code signing](#code-signing)
* [Strategy](#strategy)
* [Common Use Cases](#common-use-cases)
  * [Build & Test Workflow](#build-and-test-workflow)
  * [Block PRs on failing checks](#block-prs-on-failing-checks)
  * [Build Docker Image](#build-docker-image)
  * [Publish Docker Image](#publish-docker-image)

# Overview
Actions is a completely free feature for github open source projects. Actions has the advantage of
being 100% programmatically driven which is a slick feature.

References:
* [Workflows](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/workflow-syntax-for-github-actions)
* [Trigger Events](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/events-that-trigger-workflows#webhook-events)

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

## Check the system specifications
```yaml
name: MacOS test
on:
  pull_request:

jobs:
  macos-test-x86:
    runs-on: macos-13

    steps:
    - run: uname -a
```

# OSX builds

**Requirements**
1. Build the code
2. Run any unit tests
3. Sign the code

***Resources***
* [Apple Silicon M1](https://github.com/actions/runner-images/issues/8439)
* [macOS 13](https://github.com/actions/runner-images/blob/main/images/macos/macos-13-Readme.md)
* [macOS 13 ARM](https://github.com/actions/runner-images/blob/main/images/macos/macos-13-arm64-Readme.md)
* [Supported runners](https://docs.github.com/en/actions/using-github-hosted-runners/about-github-hosted-runners/about-github-hosted-runners#supported-runners-and-hardware-resources)

## Code signing
OSX will block code from using networking or even running in some cases if not signed. Typically this 
isn't a problem as `brew` installed software is automatically signed locally by your machine as part 
of the install process.

WARNING: OSX web downloads will automatically be marked with an untrusted bit even if the code is 
signed. You can avoid this by downloading via `wget` or `curl` or some other mechanism other than 
your browser.

Makefile example entry to code sign
```bash
ifeq (${UNAME},Darwin)
	codesign -f --deep -s - bin/TARGET_BIN
endif
```

### Manually allow unsigned code
Alternately you can manually avoid `can't be opened because Apple can't check it for malicious software` popup

1. Navigate to `System Settings >Privacy & Security` 
2. Scroll down almost to the bottom; you'll find a section for your blocked app. Click `Allow Anyway`

## Build on new M1 hardware
The M1 hardware is a bit obfuscated in Github's runner naming. The only way I can figure out how to 
select them is with the `macos-13-xlarge` name.

```json
name: Build
on:
  pull_request:

jobs:
  build:
    runs-on: macos-13-xlarge
    permissions:
      contents: read
      id-token: write

    steps:
    - uses: actions/setup-go@v4
      with:
        go-version: '1.20'

    - uses: actions/checkout@v4
      with:
        token: ${{ secrets.GIT_TOKEN }}

    - run: make
```

# Triggers

## Release

**published**
```json
{
  ...
  "event_name": "release",
  "event": {
    "action": "published",
    ...
    "release": {
      ...
      "html_url": "https://github.com/<ORG>/<PROJECT>/releases/tag/v2.4.56",
      "id": 123972193,
      "name": "WIP testing release automation - title",
      ...
      "prerelease": true,
      ...
      "tag_name": "v2.4.56",
      ...
    },
```

# Strategy
Build for three different operating systems

```json
name: Build
on:
  pull_request:

jobs:
  build-bins:
    strategy:
      matrix:
        os: [macos-13, macos-13-xlarge, ubuntu-latest]
    runs-on: ${{ matrix.os }}
    permissions:
      contents: read
      id-token: write

    steps:
    - uses: actions/setup-go@v4
      with:
        go-version: '1.20'

    - uses: actions/checkout@v4
      with:
        token: ${{ secrets.GIT_TOKEN }}

    - run: |
        bin_path=
        arch=$(uname -p)
        if [[ "${{ matrix.os }}" == "macos-13" ]]; then
          if [[ "${arch}" != "i386" ]]; then
            echo "::error::Incorrect runner architecture ${arch}"
            exit 1
          fi
          bin_path=bin/darwin-x86-bin
        elif [[ "${{ matrix.os }}" == "macos-13-xlarge" ]]; then
          if [[ "${arch}" != "arm" ]]; then
            echo "::error::Incorrect runner architecture ${arch}"
            exit 1
          fi
          bin_path=bin/darwin-arm-bin
        else
          bin_path=bin/linux-bin
        fi
```


# Common Use Cases

## Build & Test Workflow
GitHub hosted runners as referred to in the `runs-on` call out are virutal machines hosted by GitHub.
Each job in a workflow executes in a fresh instance of the virtual machine. All steps in a job
execute on the same instance of the virtual machine, allowing the actions in that job to share
information using the filesystem.

Filesystems on GitHub hosted runners have the following [env vars](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/using-environment-variables) listed you can use:
* ***HOME*** - user related data
* ***GITHUB_WORKSPACE*** - writable directory for actions and shell commands to execute in
* ***GITHUB_EVENT_PATH*** - The `POST` payload of the event that triggered the workflow
* ...

### rust build and test
References:
* [Unofficial Github Actions](https://github.com/actions-rs/meta)
* [Clippy action](https://github.com/actions-rs/clippy-check)
* [grcov action](https://github.com/actions-rs/grcov)
* [Tarpaulin action](https://github.com/actions-rs/tarpaulin)

1. Create a new `.github/workflows` directory in your repository
````
$ mkdir -p .github/workflows
```

2. Create a new workflow file e.g. `.github/workflows/build.yaml`
```yaml
name: build

# Triggered on pushes to the repo
on: 
  push:
    branches:
    - main

# Jobs to execute when triggered
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Install stable Rust toolchain
        uses: actions-rs/toolchain@v1
        with:
          toolchain: stable

      - name: Build application
        uses: actions-rs/cargo@v1
        with:
          command: build
          args: --all-features

      - name: Test application
        uses: actions-rs/cargo@v1
        with:
          command: test
          args: --all-features

      - name: Lint application
        uses: actions-rs/clippy-check@v1
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
```

### rust code coverage
I'm using github actions with tarpaulin and codecov

References:
* [Tarpaulin github action](https://github.com/actions-rs/tarpaulin)

1. Sign in to codecov.io
2. Add repo
3. Copy the upload token:
4. Login to github and navigate to `Settings >Secretes`  
   a. Click `New repository secret`  
   b. Set name to `CODECOV_TOKEN`  
   c. Set value to the value copied from codecov.io  
   d. Click `Add secret`  
5. Add two new blocks to our previous workflow
   ```yaml
      - name: Clean
        uses: actions-rs/cargo@v1
        with:
          command: clean

      - name: Tarpaulin code coverage
        id: coverage
        uses: actions-rs/tarpaulin@v0.1

      - name: Upload to codecov.io
        uses: codecov/codecov-action@v1.0.2
        with:
          token: ${{secrets.CODECOV_TOKEN}}
   ```

### golang build and test<a name="golang-build-and-test"/></a>
1. Create a new `.github/workflows` directory in your repository

2. Create a new workflow file e.g. `.github/workflows/build.yaml`
```yaml
name: build

# Triggered on pushes to the repo
on: 
  push:
    branches:
    - master

# Jobs to execute when triggered
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Standard action to pull source
        uses: actions/checkout@v2
      - name: Setup Go build environment
        uses: actions/setup-go@v2
        with:
          go-version: 1.15.3
      - name: Build application
        run: |
          make
      - name: Test application
        run: |
          make test
```

## Build Docker Image
Create a basic docker image builder workflow `.github/workflows/build.yaml`:
```yaml
name: Build Docker Image
on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v1
    - name: Build Docker image
      run: docker build . --file Dockerfile --tag alpine-base:$(date +%s)
```

## Publish Docker Image

1. Create DockerHub secrets in Github:  
   a. Navigate to `Settings >Secrets`  
   b. Click `Add a new secret`  
   c. Fill out `DOCKERHUB_USER` and value  
   d. Fill out `DOCKERHUB_PASS` and value  

Create a basic docker image builder workflow `.github/workflows/publish.yaml`:
```yaml
name: Publish Docker Image
on:
  release:
    types: [published]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v1
    - name: Build Docker image
      run: docker build . --file Dockerfile --tag alpine-base:${GITHUB_REF}
```

<!-- 
vim: ts=2:sw=2:sts=2
-->
