# Nix Language

### Quick links
* [Overview](#overview)
  * [Example configs](#example-configs)
* [nixos-rebuild](#nixos-rebuilt)
  * [Skip git add requirement](#skip-git-add-requirement)
* [nixpkgs](#nixpkgs)
  * [legacyPackages](#legacypackages)
* [Value priority enforcement](#value-priority-enforcement)
* [config](#config)
* [Modules](#modules)
* [Flakes](#flakes)

## Overview

***References***
* [Nix Dev](https://nix.dev/tutorials/nix-language.html)
* [Nix Reference Manual](https://nixos.org/manual/nix/stable/language/)
* [Flakes aren't real](https://jade.fyi/blog/flakes-arent-real/)

### Example configs
* [NobbZ](https://github.com/NobbZ/nixos-config)

## nixos-rebuild

### Skip git add requirement
NixOS flakes are stored in a git repo and nix goes out of its way to assist in helping you remember 
to commit your changes. Thus by default you are required to at least stage your changes before 
`nixos-rebuild` will recognize them. However in some cases this is not ideal. You can skip this check 
and just have nix rebuild regardless by using the `path:` prefix for your flake path.

* [Flake types](https://nixos.org/manual/nix/stable/command-ref/new-cli/nix3-flake.html#types)

```
$ cd /etc/nixos
$ sudo nixos-rebuild switch --flake path:.#install
```

## builtins
`(builtins.readFile ./bash/myscript.sh)`

### inherit
inherit is shorthand for assigning the value of a name from an existing scope to the same name in a 
nested scope. It is a convenience to avoid repeating the same name multiple times.

* [inherit](https://nix.dev/tutorials/nix-language#inherit)

**Inherit as many attributes as you'd like**
```
let
  x = 1;
  y = 2;
in
{
  inherit x y;
}
```

**Inherit from a specific attribute set**
```
let
  a = { x = 1; y = 2; };
in
{
  inherit (a) x y;
}
```

## xdg.configFile

## writeShellScriptBin
This function takes 2 arguments; the name of the script you want in your PATH and the string contents 
of the script.

**Example**
```
script = writeShellScriptBin "script-name" ''
  # content
'';
```

## nixpkgs
NixOS will call `pkgs.nixos` which injects its own instance of nixpkgs to the `nixpkgs.pkgs` option.

### configure nixpkgs for nix-shell
Flake based systems don't need the traditional channel setup. However to get the `nix-shell` command 
to work properly you need to configure the `nixPath` i.e. `NIX_PATH` to use the flake specified nixpkgs.

* [Use nixpkgs](https://discourse.nixos.org/t/correct-way-to-use-nixpkgs-in-nix-shell-on-flake-based-system-without-channels/19360)
* [Common Environment Variables](https://nixos.org/manual/nix/stable/command-ref/env-common.html)

The `NIX_PATH` environment variable ends up being the following by default. However when installing 
with the `--no-channel-copy` flag for `nixos-install` this path is no longer valid.
```
nixpkgs=/nix/var/nix/profiles/per-user/root/channels/nixos:nixos-config=/etc/nixos/configuration.nix:/nix/var/nix/profiles/per-user/root/channels
```

We need to change this to use the flake nixpkgs
```
nix.nixPath = [ "nixpkgs=${nixpkgs.outPath}" "unstable=${unstable.outPath}" ];
```

### legacyPackages
It's called legacy because it is not following the rules for packages, i.e. its not flat but a set of 
sets. This doesn't mean its legacy in the traditional sense. It is very much the thing to use.

* [legacyPackags discourse](https://discourse.nixos.org/t/using-nixpkgs-legacypackages-system-vs-import/17462/4)
* [1000 instances of nixpkgs](https://zimbatm.com/notes/1000-instances-of-nixpkgs)

**Best practice**
```
{
  inputs.nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
  inputs.other-dep.url = "github:other/dep";
  inputs.other-dep.inputs.nixpkgs.follows = "nixpkgs"; # Use the same version of nixpkgs as us

  outputs = { self, nixpkgs, other-dep }@inputs: {
      packages = nixpkgs.lib.genAttrs [ "x86_64-linux" ] (system:
        let
          p = {
            nixpkgs = inputs.nixpkgs.legacyPackages.${system};
            # other-dep would also access `inputs.nixpkgs.legacyPackages.${system}` thus only using a single instance of it.
            other-dep = inputs.other-dep.packages.${system};
          };
        in
        # your code here accessing `p.nixpkgs` and `p.other-dep`
     );
  };
}
```

## Value priority enforcement
Nix values are enforced at different levels of priority. The smaller the priority number the more 
priority an override has. All but the smallest piority value i.e. highest priority are discarded and 
the remaining value is used.

```
value = mkDefault "1000";
value = mkImageMediaOverride "60";
value = mkForce "50";
value = mkOverride 10 "10";
```

Resulting value of `value` would be `"10"`

### option defaults
Option defaults have a priority of `1500`;

### mkDefault
`mkDefault` is defined as `mkDefault = mkOverride 1000;`

### mkImageMediaOverride
`mkImageMediaOverride` is defined as `mkImageMediaOverride = mkOverride 60;` and therefore paves over 
simple values assignments and `mkDefault` but yields to `mkForce`

### mkForce
`mkForce` is defined as `mkForce = mkOverride 50;`

### mkOverride
`mkOverride` is the root function that allows for setting a custom value for the priority level.

Example setting priority to 10 which is a higher priority than even `mkForce` at `50`
```nix
services.openssh.enable = mkOverride 10 false;
```

## config
This argument gives access to the entire configuration of the system. It is computed from option 
declarations and option definitions defined inside all modules used for the system.

Effectively this means that if you've set values in `minimal.nix` for example like 
`documentation.enable = false;` then you can override this specific value using 
`config.documentation.enable = lib.mkOverride 500 true;`

* [config argument](https://nixos.wiki/wiki/NixOS:config_argument)

## Modules
NixOS modules are just fundamentally functions which are invoked in a fancy way and not a flakes 
construct. Best practice and what is done with all the NixOS profiles etc... is to create reusable 
modules that can be imported. Then you just import what you need in your top level entry `flake.nix` 
and inject dependencies from its environment.

```
{lib, config, options, pkgs, ...}:
{
  # Importing other Modules
  imports = [
    # ...
    ./xxx.nix
  ];
  for.bar.enable = true;
  # Other option declarations
  # ...
}
```

Modules have five `automatically generated`, `automatically injected`, `declaration-free` parameters 
provided by the module system.
* `lib` - a built-in function library [nixpkg lib](https://nixos.org/manual/nixpkgs/stable/#id-1.4)
* `config` - all option values in the current environment, useful for conditionals and overrides
* `options` - all options defined in all Modules in the current environment
* `pkgs` - a collection containing all nixpkgs along with utility functions
* `modulesPath` - helper referring to the `nixpkgs/nixos/modules`

### top-level error
If you something like `has an unsupported attribute ... This is caused by introducing a top-level 
config or options`.

NixOS modules allow for two syntaxes
```
{
  imports = [ <some imports> ];
  my-favorite-option = true;
}
```

and

```
{
  imports = [ <some imports> ];
  options = { <some custom options> };
  config = {
    my-favorite-option = true;
  };
}
```

The first syntax is used to set options. The second is used to define custom options. If you use the 
second syntax with a `confi` or `options` at the top-level then you need to put all configuration 
options inside those else the module system won't work. So avoid the second syntax unless you have to 
to make is simpler.

## Flakes
After digesting the Nix community blogs, docs and various sources its clear that ***Flakes*** are 
fantastic and also simply an additional feature in a rich Nix based ecosystem not necessarily a 
replacement paradigm. Their purpose is to be a top level entry point for a NixOS configuration to 
to ensure dependencies are versioned properly to be able to rebuild your configuration in a reliable 
way. So yes the feature is stable and useful but is only one small feature for a specific use case.
Perhaps because it is tied closely to the change with the nix cli 2.0 it seems to get more press than 
is warranted. There is a lot of hype around flakes and they serve a purpose but it doesn't really 
replace anything but channels for NixOS.

* [FlakesHub](https://flakehub.com/)
* [NixOS and Flakes book](https://nixos-and-flakes.thiscute.world/best-practices/run-downloaded-binaries-on-nixos)
* [Flakes wiki](https://nixos.wiki/wiki/Flakes)
* [Cache builds with Cachix](https://docs.cachix.org/)
* [Flakes with NixOS](https://www.tweag.io/blog/2020-07-31-nixos-flakes/)
* [NixOS Modules](https://nixos.wiki/wiki/NixOS_modules)
* [Nixos flakes guide](https://nixos-and-flakes.thiscute.world/nixos-with-flakes/introduction-to-flakes)
* [Convert to flakes](https://nixos.asia/en/nixos-install-flake)
* [Flakes aren't real](https://jade.fyi/blog/flakes-arent-real/)

### Lock file
Flakes introduce the `flake.nix` and the `flack.lock` similar to other programming languages to 
describe dependencies and versions to ensure your project can be reproduced.

### Basic flake commands
* `nix flake show templates`
* `nix flake init -t templates#full`

### Enabling flakes temporarily
```bash
$ nix --extra-experimental-features 'nix-command flakes' flake show .
```

### Install from flake
```bash
$ sudo nixos-install --no-root-passwd --impure --flake .#nixos
```

### Flake registry
The [nix registry](https://nixos.org/manual/nix/stable/command-ref/new-cli/nix3-registry.html) 
command allows for interacting with the flake registry. Flake registries are a convenience feature 
that allows you to refer to flakes using symbolic identifers such as `nixpkgs` rather than the full 
URLs such as `git://github.com/NixOS/nixpkgs`. They also allow you to override these mappings to 
redirect to other locations. There are multiple registries:

* the [global registry](https://github.com/NixOS/flake-registry/blob/master/flake-registry.json), 
* the system registry at `/etc/nix/registry.json`
* the user registry at `~/.config/nix/registry.json`

<!-- 
vim: ts=2:sw=2:sts=2
-->
