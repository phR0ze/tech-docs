# Home Manager
Home Manager is the defacto standard for managing user applications and dot files. It was built as a 
companion to Nix but focused on user configuration. Although Nix and Home Manager's focus on a cross 
platform, multi-distro, multi-user type paradigm is super useful for those that use case fits it is 
less than ideal for the typical single user consumer type system. 

Single user consumer systems, my use case, typically want system wide user configuration where their 
single user and root user mostly mirror a similar configuration and most importantly installed 
applications are uniformly available to all users.

### Quick links
* [Getting Started](#getting-started)
  * [Activate Home Manager deeper in your structure](#activate-home-manager-deeper-in-your-structure)
  * [Activate Home Manager at the top level](#activate-home-manager-at-the-top-level)
* [Configuration](#configuration)

## Configuration
As with Nix, Home Manager works best when configuration is broken out into modules. Home Manager 
works much like Nix in that you can arrange your configuration into modules with options that only 
Home Manager understands. As such it is best to sort all Nix only modules in one directory and all 
Home Manager dependent modules in another directory. After you load and activate Home Manager via 
your Nix entry point flake then use Home Manager to load your user configuration modules.

* [Home Manager Options](https://mipmip.github.io/home-manager-option-search/?query=neovim)
* [Modular configuration](https://librephoenix.com/2024-01-28-program-a-modular-control-center-for-your-config-using-special-args-in-nixos-flakes.html)
* [Good example config](https://gitlab.com/hmajid2301/dotfiles/-/blob/main/flake.nix?ref_type=heads)
* [Home Manager example](https://github.com/nix-community/home-manager/blob/master/templates/nixos/flake.nix)
* [How to add home manager to nixos](https://drakerossman.com/blog/how-to-add-home-manager-to-nixos)

### Package installation location
Home Manager installs programs using the `programs.<program>.enable = true;` syntax as links in 
`/home/<user>/.nix-profile/bin/` while Nix installs them as links in `/run/current-system/sw/bin/` as 
shown by `which`. This means that you can easily get mixed up and have multiple versions of the same 
application installed and using different versions for different users.

## Getting started
Home Manager can be installed in two forms. The standalone case is suited for the cross-platform 
non-NixOS type use cases. Or it can be installed as a Nix module that is driven by Nix system 
configuration and more useful for full system installs with NixOS.

### Activate Home Manager deeper in your structure
Configuration is really the same as the top level version except we move the home-manager activation 
and `users.nixos` configuration lower in the module tree for convenience.

1. Declare your home manager as in input in your `flake.nix`
   ```
   { 
     description = "NixOS configuration";

     inputs {
       nixpkgs.url = "github:nixos/nixpkgs/nixos-23.11"; 
   
       # the master branch of home manager would be for the unstable nixpkgs
       home-manager.url = "github:nix-community/home-manager/release-23.11";
   
       # Configure home manager to use the same nixpkgs
       home-manager.inputs.nixpkgs.follows = "nixpkgs";
     };
   ```

2. Include home manager as an output argument in your `flake.nix`
   ```
     outputs = { nixpkgs, home-manager, ... }@inputs: let
       system = "x86_64-linux";
       lib = nixpkgs.lib // home-manager.lib;
       args = inputs // { inherit systemSettings homeSettings; };
       specialArgs = { inherit args; };
     in {
   ```

3. Load your nix configuation in `nixConfigurations`
   ```
       nixosConfigurations = {
         myuser = lib.nixosSystem {
           inherit pkgs system specialArgs;
           modules = [ ./configuration.nix ];
         };
       };
     };
   }
   ``` 

4. Load and activate home manager in `configuration.nix`
   ```
   { lib, pkgs, args, ... }:
   {
     imports = [
       # Import and activate home-manager
       args.home-manager.nixosModules.home-manager
     ];
   
     # Configure home-manager
     home-manager = {
       extraSpecialArgs = { inherit args; };
       users.myuser = {
         imports = [
           ./dircolors.nix
           ./git.nix
         ];

         home = {
           username = "myuser";
           homeDirectory = "/home/myuser";
           stateVersion = "23.11";
         };
       };
     };
   }
   ``` 


### Activate Home Manager at the top level
1. Declare your home manager as an input in your `flake.nix`
   ```
   { 
     description = "NixOS configuration";

     inputs {
       nixpkgs.url = "github:nixos/nixpkgs/nixos-23.11"; 
   
       # the master branch of home manager would be for the unstable nixpkgs
       home-manager.url = "github:nix-community/home-manager/release-23.11";
   
       # Configure home manager to use the same nixpkgs
       home-manager.inputs.nixpkgs.follows = "nixpkgs";
     };
   ```

2. Include home manager as an output argument in your `flake.nix`
   ```
     outputs = { nixpkgs, home-manager, ... }@inputs: let
       lib = nixpkgs.lib // home-manager.lib;
     in {
   ```

3. Load your home manager module in your `nixConfigurations`
   ```
       nixosConfigurations = {
         nostname = lib.nixosSystem {
           system = "x86_64-linux";
           modules = [
             ./configuration.nix
             home-manager.nixosModules.home-manager
             {
               home-manager.users.nixos = import ./home.nix;
             }
           ];
         };
       };
     };
   }
   ``` 
4. `home.nix` is your home-manager module that will understand and execute home-manager options

<!-- 
vim: ts=2:sw=2:sts=2
-->
