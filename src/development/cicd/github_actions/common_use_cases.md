# Common Use Cases

### Quick links
* [Build & Test Workflow](#build-and-test-workflow)
* [Block PRs on failing checks](#block-prs-on-failing-checks)
* [Build Docker Image](#build-docker-image)
* [Publish Docker Image](#publish-docker-image)

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
   ```
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
