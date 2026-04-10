# NeoVim <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Nix can be used to configure and build applications for your OSX system. It can also be used for
custom user configuration. To do so you simply need to install home-manager drop in your flake
configuration and then run `home-manager switch --flake <target>`.

Note: I created a private repo for my configuration `Projects/osx-nix` but its essentially just the
configs below.

### Quick links
- [.. up dir](..)
- [Install Nix](#install-nix)
- [Deploy Neovim](#deploy-neovim)
  - [Configs](#configs)
    - [flake.nix](#flakenix)
    - [home.nix](#homenix)
  - [Install Home Manager](#install-home-manager)
- [Upgrade Neovim](#upgrade-neovim)

## Install Nix
The easiest way to get started is by using the ***Determinate Systems Installer***

1. Navigate to [Determinate Systems](https://determinate.systems/nix-installer/)

2. Copy the curl bash line and execute
   ```
   curl --proto '=https' --tlsv1.2 -sSf -L https://install.determinate.systems/nix | sh -s -- install
   ```

## Deploy Neovim

### Configs
In the configuration files below you'll want to change `my-user` to your user's name to make this
work correctly.

Key files:
- ~/.config/home-manager/flake.nix
- ~/.config/home-manager/home.nix

#### flake.nix
```nix
{
  description = "Home Manager configuration";
  inputs = {
    nixpkgs.url = "github:nixos/nixpkgs/nixpkgs-unstable";
    home-manager = {
      url = "github:nix-community/home-manager";
      inputs.nixpkgs.follows = "nixpkgs";
    };
    phR0ze-config = {
      url = "github:phR0ze/nixos-config";
      flake = false;
    };
  };
  outputs = { nixpkgs, home-manager, phR0ze-config, ... }:
    let
      system = "aarch64-darwin";
      pkgs = nixpkgs.legacyPackages.${system};
    in {
      homeConfigurations."my-user" = home-manager.lib.homeManagerConfiguration {
        inherit pkgs;
        modules = [ ./home.nix ];
        extraSpecialArgs = { inherit phR0ze-config; };
      };
    };
}
```

#### home.nix
```nix
{ config, pkgs, phR0ze-config, ... }:
  let
    customNeovim = pkgs.callPackage ./neovim/package.nix {
      configSrc = "${phR0ze-config}/options/apps/system/neovim";
    };
  in
  {
    home.username = "my-user";
    home.homeDirectory = "/Users/my-user";
    home.stateVersion = "24.11";
    programs.home-manager.enable = true;
    programs.tmux = {
      enable = true;
      prefix = "C-Space";
      keyMode = "vi";
      mouse = true;
      terminal = "tmux-256color";
      extraConfig = ''
        set -ag terminal-overrides ",xterm-256color:RGB"

        # TokyoNight Night status bar
        set -g status-style "bg=#24283b,fg=#a9b1d6"
        set -g status-left-style "bg=#7aa2f7,fg=#24283b"
        set -g status-right-style "bg=#7aa2f7,fg=#24283b"
        set -g window-status-current-style "bg=#3b4261,fg=#7aa2f7,bold"
        set -g window-status-style "bg=#24283b,fg=#565f89"
        set -g pane-border-style "fg=#3b4261"
        set -g pane-active-border-style "fg=#7aa2f7"
      '';
    };

    home.packages = [
      customNeovim
    ];

    home.sessionVariables = {
      EDITOR = "nvim";
      VISUAL = "nvim";
      KUBE_EDITOR = "nvim";
    };

    # stylua formatter config
    home.file.".config/stylua/stylua.toml".text = ''
      indent_type = "Spaces"
      indent_width = 2
    '';
  }
```

### Install Home Manager
We can use the freshly installed `nix` to now install home manager via a flake

```bash
cd ~/.config/home-manager
nix run home-manager -- switch --flake .#$USER
```

The result is a single nvim binary that carries all its plugins and LSPs with it.
- `~/.nix-profile/bin/nvim`

## Upgrade Neovim
After the first run that install home manager we can then work with home manager directly rather
than having to use nix to boot it.

1. You could just roll to the latest available but if you not ready for it you might end up needing
   changes to the upstream nixos-config repo. Better to follow the upstream repo.
   1. [Setting the flake unstable](https://github.com/phR0ze/nixos-config/blob/389fe147aff1b47ed6563edcf36b93fb2630b911/base.nix#L4)
   2. run `nix flake update nixpkgs`
2. Upgrade the vim deployment
   ```bash
   cd ~/.config/home-manager
   home-manager switch --flake .#$USER
   ```
