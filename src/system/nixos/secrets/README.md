# Secrets <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

### Quick links
- [.. up dir](..)
- [Overview](#overview)

### Linked pages
* [sops-nix](sops_nix/README.md)

## Overview
The NixOS community has a plethora of secrets management options of which `git-crypt`, `agenix` and 
`sops-nix` are often used.

**References**
- [Comparison of scretes - NixOS Wiki](https://nixos.wiki/wiki/Comparison_of_secret_managing_schemes)
- [Handling secrets](https://lgug2z.com/articles/handling-secrets-in-nixos-an-overview/)

### git-crypt
`git-crypt` is the simplest of the three but leaves secrets fully exposed in plain text when built 
and stored in the `/nix/store` which is readable to all users of a machine. This isn't a big deal 
when your the only user but definetely not best practice.

### Agenix
`agenix` ensures secrets are encrypted at rest in github as well as the nix store locally and is 
arguably simpler to configure. The only downside to Agenix is that it requires a file per secret 
which can be cumbersome.

### sops-nix
[sops-nix](https://github.com/Mic92/sops-nix) while arguably more complicated to setup solves both 
prior issues in that secrets are safely encrypted in github and the nix store and we can group 
secrets together in a single file if desired. Additionally [SOPS](https://github.com/getsops/sops), 
written in Golang, is backed by Mozilla and other notables and used outside of NixOS in the industry 
at large. It is enterprise ready and battle tested. See [sops-nix](sops_nix/README.md) for setup 
details.
