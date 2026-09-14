# Provisioning <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Tools and workflows for turning a machine into a `NixOS` system, particularly useful for cloud VPS
providers that don't offer a native `NixOS` image or ISO boot option.

### Quick links
- [.. up dir](..)
- [Overview](#overview)

### Linked pages
- [nixos-consume](nixos_consume/README.md)

## Overview
Most cloud providers only offer a handful of common distro images (`Ubuntu`, `Debian`, `CentOS`, etc.)
and don't support booting from a custom `NixOS` ISO. Provisioning tools in this section work around
that by converting an already-running Linux system into `NixOS` in place, rather than requiring a
fresh install from installer media.

**References**
* [NixOS Manual: Installing from another Linux distro](https://nixos.org/manual/nixos/stable/#sec-installing-from-other-distro)
