# Dev env <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

`nix shell -p` is simple and fast, but requires you to remember what you wanted in your 
configuration. For more complicated setups use `nix develop` and a dev flake.

### Quick links
* [Create dev flake](#create-dev-flake)

## Create dev flake
Example dev env with Node.js 18

***flake.nix***
```nix
{
  description = "A Nix-flake-based Node.js development environment";
  inputs = {
    nixpkgs.url = "github:nixos/nixpkgs/nixos-unstable";
  };

  outputs = { self , nixpkgs ,... }: let
    # system should match the system you are running on
    system = "x86_64-linux";
  in {
    devShells."${system}".default = let
      pkgs = import nixpkgs {
        inherit system;
      };
    in pkgs.mkShell {
      # create an environment with nodejs_18, pnpm, and yarn
      packages = with pkgs; [
        nodejs_18
        nodePackages.pnpm
        (yarn.override { nodejs = nodejs_18; })
      ];

      shellHook = ''
        echo "node `${pkgs.nodejs}/bin/node --version`"
      '';
    };
  };
}
```
