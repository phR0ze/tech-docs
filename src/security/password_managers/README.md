# Password Managers <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

### Quick links
* [.. up dir](..)
# [Bitwarden](#bitwarden)
# [Buttercup](#buttercup)
# [KeepassXC](#keepassxc)
  # [TOTP Support](#totp-support)

## Bitwarden

**Pros**
* Free for personal use
* Cross-platform including Linux
* Command-line tools
* Ability to host is on your own servers

## Buttercup

**Pros**
* Free, with no premium options
* Cross-platform
* Browser-extensions
* Strictly offline usage

## KeePassXC
Maintained fork of KeePassX which was the original port of KeePass to Linux.

**Pros**
* Cross-platform includeing Linux and Android
* Offline only
* Dead simple

**Cons**
* A bit dated looking

### TOTP Support
KeePassXC now supports TOTP.

1. Create a password entry in KeePassXC
2. Right click on it an select `TOTP >Set up TOTP...`
3. On the next screen entry the QR Code translated to a text code
   1. You can use a screen capture tool like `Share X`
   2. From the decoded URL copy out the value to the `secret` property
   3. Alternatively most will have can't scan QR code and display text
