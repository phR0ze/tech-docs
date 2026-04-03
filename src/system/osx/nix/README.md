# Nix <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

### Quick links
- [.. up dir](..)
- [Overview](#overview)

## Overview

### Approaches
* ***Option A*** — Nix + home-manager (recommended starting point): Installs Nix alongside your existing
  homebrew setup. home-manager manages your dotfiles/configs (nvim, starship, etc.) declaratively and
  is the least disruptive.

* ***Option B*** — nix-darwin + home-manager: Manages your entire macOS system config via Nix — homebrew,
  system settings, everything. Much bigger commitment, but fully reproducible.

## Installation


### Install Nix
The Determinate Systems installer is the way to go. They have built a super clean and easy way to
deal with all the confusing OSX quirks so that it just works.

1. Install via the Determinate Systems installer
   ```bash
   $ curl --proto '=https' --tlsv1.2 -sSf -L https://install.determinate.systems/nix | sh -s -- install
   ```
2. Read through the explanation of what will happen then press `Y`

3. Start the nix daemon
   ```bash
   $ . /nix/var/nix/profiles/default/etc/profile.d/nix-daemon.sh
   ```

### Install Home Manager
Home manager does for user configuration what Nix does for system configuration. Configuration is
stored in `~/.config/home-manager` including the `flake.nix` and `home.nix`

1. Install home manager using nix and build configuration
   ```bash
   $ nix run home-manager/master -- switch --flake ~/.config/home-manager
   ```

3. Apply home manager configuration
   ```bash
   $ home-manager switch --flake ~/.config/home-manager
   ```

4. Alternatively from `~/.config/home-manager` run
   ```bash
   $ home-manager switch --flake .#YOUR_USER
   ```
