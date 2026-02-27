# Strategy

### Quick links

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
