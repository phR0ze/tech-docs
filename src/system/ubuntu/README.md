# Ubuntu <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

`Ubuntu` is a Debian based Linux distribution maintained by Canonical, commonly used for desktops,
servers and cloud images. This section covers Ubuntu specific configuration, administration and
security topics.

### Quick links
* [.. up dir](..)
* [Overview](#overview)
* [Package Management](#package-management)
* [Hardening](hardening)

### Linked pages
* [Hardening](hardening/README.md)

## Overview
Ubuntu ships in several flavors:
* ***Desktop*** — GNOME based desktop image intended for workstations
* ***Server*** — minimal image intended for headless server deployments
* ***Cloud images*** — pre-built images optimized for cloud providers e.g. AWS, GCP, Azure

**References**
* [Ubuntu Documentation](https://help.ubuntu.com/)
* [Ubuntu Server Guide](https://ubuntu.com/server/docs)

## Package Management
Ubuntu uses `apt` as its front end package manager on top of `dpkg`.

**Update package lists and upgrade installed packages**
```bash
$ sudo apt update
$ sudo apt upgrade
```

**Search for a package**
```bash
$ apt search PACKAGE
```

**Show info about a package**
```bash
$ apt show PACKAGE
```

**Remove a package and its config files**
```bash
$ sudo apt purge PACKAGE
```

**Remove unused dependencies**
```bash
$ sudo apt autoremove
```
