# Nix <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

### Quick links
- [.. up dir](..)
- [Overview](#overview)
- [Installation](#installation)
  - [Install Nix](#install-nix)
  - [Install nix-darwin](#install-nix-darwin)
  - [Integrate Home Manager](#integrate-home-manager)
- [Rebuilding](#rebuilding)

## Overview
***nix-darwin*** manages your entire macOS system config via Nix — homebrew, system settings,
everything — with ***home-manager*** layered in as a module for user-level dotfiles/configs (nvim,
starship, etc.). This is a bigger commitment than a standalone Nix + home-manager setup, but the whole
system becomes reproducible from a single flake.

## Installation

### Install Nix
nix-darwin doesn't install Nix itself — install it first using the official multi-user installer.

1. Install Nix
   ```bash
   $ sh <(curl -L https://nixos.org/nix/install)
   ```
2. Follow the prompts, then start a new shell so `nix` is on your `PATH`
   ```bash
   # Build your first package and run it
   $ nix-shell -p nix-info --run "nix-info -m"
   ```
3. Move the default nix.conf aside as nix-darwin will take over
   ```bash
   $ sudo mv /etc/nix/nix.conf /etc/nix/nix.conf.before-nix-darwin
   ```

### Install nix-darwin
Configuration is stored in `~/.config/nix/flake.nix`, alongside the `nix.conf` from the previous step.

1. Create the flake, replacing `my-host` and `aarch64-darwin` as needed
   ```nix
   {
     description = "My darwin system";
     inputs = {
       nixpkgs.url = "github:NixOS/nixpkgs/nixpkgs-unstable";
       nix-darwin = {
         url = "github:LnL7/nix-darwin";
         inputs.nixpkgs.follows = "nixpkgs";
       };
     };
     outputs = inputs@{ self, nix-darwin, nixpkgs }: {
       darwinConfigurations."my-host" = nix-darwin.lib.darwinSystem {
         modules = [
           ({ pkgs, ... }: {
             nix.settings.experimental-features = "nix-command flakes";
             system.stateVersion = 5;
             nixpkgs.hostPlatform = "aarch64-darwin";
           })
         ];
       };
     };
   }
   ```
2. Build and activate for the first time
   ```bash
   $ sudo nix --extra-experimental-features 'nix-command flakes' run nix-darwin -- switch --flake ~/.config/nix
   ```
3. After the first activation, `darwin-rebuild` is installed onto your `PATH` and can be used directly
   from then on.

### Integrate Home Manager
Add home-manager as a nix-darwin module so user configuration (dotfiles, packages) is built and
activated together with the system config, rather than as a separate `home-manager switch` step.

1. Add the input to `flake.nix`
   ```nix
   home-manager = {
     url = "github:nix-community/home-manager";
     inputs.nixpkgs.follows = "nixpkgs";
   };
   ```
2. Wire it into the `darwinConfigurations` modules list
   ```nix
   modules = [
     ./configuration.nix
     home-manager.darwinModules.home-manager
     {
       home-manager.useGlobalPkgs = true;
       home-manager.useUserPkgs = true;
       home-manager.users.my-user = import ./home.nix;
     }
   ];
   ```

## Rebuilding
After the initial `nix run nix-darwin -- switch`, apply further changes with
```bash
$ darwin-rebuild switch --flake ~/.config/nix
```
