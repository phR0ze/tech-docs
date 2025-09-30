# Patch binary <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Using the venerable [PatchELF](https://github.com/NixOS/patchelf) tool we can update dynamically 
linked binaries that are dependent on the Standard Linux FHS to instead use the correct NixOS 
external library paths. This means you can take prebuilt Ubuntu or Debian or other distro packages 
and repackage them to work with NixOS.

### Quick links
* [.. up dir](..)
* [Overview](#overview)

## Overview
I finally got this to work see [KasmVNC package](https://github.com/phR0ze/nixos-config/blob/main/packages/kasmvnc/default.nix)
