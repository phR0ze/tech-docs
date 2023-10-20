# OSX Builds

### Quick links
* [Code signing](#code-signing)

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

<!-- 
vim: ts=2:sw=2:sts=2
-
