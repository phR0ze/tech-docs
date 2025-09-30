# Build Package
Nix packages are the foundation of the Nix ecosystem. Although the NixOS operating system is entirely 
composed of Nix packages, Nix packages can be installed on non-NixOS systems as well. Regardless of 
their eventual deployment target Nix packages are built using the Nix expression language and managed 
using the Nix package manager.

### Quick links
* [Overview](#overview)
  * [Organization](#organization)
  * [Show build dependencies](#show-build-dependencies)
* [Packaging local files](#packaging-local-files)
* [Build packages](#build-packages)
  * [Build phases](#build-phases)
  * [Build nixpkgs package locally](#build-nixpkgs-package-locally)
  * [Build nixpkgs package locally in phases](#build-nixpkgs-package-locally-in-phases)
  * [Override package unwrapped](#override-package-unwrapped)
* [Package overrides](#package-overrides)
  * [override vs overrideAttrs](#override-vs-overrideattrs)
* [Patch packages](#patch-packages)

## Overview

* [Nix Pills](https://nixos.org/guides/nix-pills/09-automatic-runtime-dependencies)
* [Customizing packages](https://bobvanderlinden.me/customizing-packages-in-nix/)
* [Nix REPL](https://aldoborrero.com/posts/2022/12/02/learn-how-to-use-the-nix-repl-effectively/)

### Organization
In the Nix packaging world there are usually two tasks that need to be performed. First the software 
needs to be built to work with NixOS. Second the software needs to have a mechanism to apply options 
for it to enable the end user to customize the install.

* `packages/kasmvnc/default.nix` - nix expression to actually build the software
* `options/service/raw/kasmvnc.nix` - nix expression to provide options for configuring the softare

### Show Build Dependencies
```
$ nix-instantiate
```

## Packaging local files
The key to building derivations is `stdenv.mkDerivation` and its various builder helpers that wrap it 
for different convenience purposes e.g. `runCommandLocal`.

Coercion of paths to strings such as in `"${./.}"`  will copy the whole directory into the Nix store 
on evaluation.

* [Option types](https://nlewo.github.io/nixos-manual-sphinx/development/option-types.xml.html)
* [Trivial builders - ryantm](https://ryantm.github.io/nixpkgs/builders/trivial-builders/)
* [Workiung with local files - nix.dev](https://nix.dev/tutorials/working-with-local-files.html)

1. Open the repl
   ```bash
   $ nix repl -f '<nixpkgs>'
   ```
2. Get a simple var `fs` pointing to the `lib.fileset` functions
   ```
   nix-repl> fs = lib.fileset
   ```
3. View a file's properties
   ```
   nix-repl> fs.trace ./include/home/.dircolors
   trace: /etc/nixos/include/home
   trace: - .dircolors (regular)
   «lambda @ /nix/store/lwyjz70qh12nq6cb7fixl85vryzxqm3c-source/lib/fileset/default.nix:225:8»
   ```
* Adding a new attribute to an attribute list then print it out
   ```
   nix-repl> foo = map (x: x // { foo = x.name; } ) [ { name = "foo"; value = 2; } ]
   nix-repl> :p foo
   [ { foo = "foo"; name = "foo"; value = 2; } ]

   nix-repl> :p map (x: x // { foo = x.name; } ) [ { name = "foo"; value = 2; } ]
   [ { foo = "foo"; name = "foo"; value = 2; } ]
   ```

```nix
# Rename the set using the value.target field
(attrsets.mapAttrs' (name: value: nameValuePair (value.target) value) config.files.root)
```

## Build packages
Final built packages are located in `/nix/store`. The nix store is a globally readable directory of 
all build inputs and outputs.

* [nixpkgs contributing guide](https://github.com/NixOS/nixpkgs/blob/master/pkgs/README.md)
* [Building a Nix Package](https://gist.github.com/CMCDragonkai/41593d6d20a5f7c01b2e67a221aa0330)
* [Building a stdenv package in a nix-shell](https://nixos.org/manual/nixpkgs/stable/#sec-building-stdenv-package-in-nix-shell)

### Build phases
Nix provides a number of environment variables for controlling which build phasese to run:;
* [Build phases](https://nixos.org/manual/nixpkgs/stable/#sec-stdenv-phases)

The `phases` environment variable controlls which phases to run and defaults to:
* `unpackPhase` - unpacks the code
* `patchPhase`
* `configurePhase`
* `buildPhase`
* `checkPhase`
* `installPhase`
* `fixupPhase`
* `installCheckPhase`
* `distPhase`

```bash
phases="${prePhases[*]:-} unpackPhase patchPhase ${preConfigurePhases[*]:-} \
    configurePhase ${preBuildPhases[*]:-} buildPhase checkPhase \
    ${preInstallPhases[*]:-} installPhase ${preFixupPhases[*]:-} fixupPhase installCheckPhase \
    ${preDistPhases[*]:-} distPhase ${postPhases[*]:-}";
```

### Build nixpkgs package locally
1. Drop into a nix-shell build environment. This will download the target package's build 
   dependencies and list them. Additionally the `nix-shell` will persist as environment variables a 
   number of build values like the package `name=hello-2.12.1`.
   ```bash
   $ nix-shell '<nixpkgs>' -A prismlauncher-unwrapped
   ```

2. You can then simply build the entire package right here
   ```bash
   $ genericBuild
   ```
 
### Build nixpkgs package locally in phases
Using `Prism Launcher` for my example

1. Drop into a nix-shell build environment
   ```bash
   $ nix-shell '<nixpkgs>' -A prismlauncher-unwrapped
   ```

2. Next invoke the `unpack phase` to get a working copy of the sources and auto change into that dir
   ```bash
   $ phases="${prePhases[*]:-} unpackPhase" genericBuild
   ```

3. Patch the source with your patch file, or modify it directly. Note this is from within the source directory.
   ```bash
   $ patch --no-backup-if-mismatch -p1 < ~/Projects/nixos-configs/patches/prismlauncher/v9.1/offline.patch
   ```
   
4. Next invoke the rest of the build process
   ```bash
   $ phases="${preConfigurePhases[*]:-} configurePhase ${preBuildPhases[*]:-} buildPhase checkPhase \
    ${preInstallPhases[*]:-} installPhase ${preFixupPhases[*]:-} fixupPhase installCheckPhase \
    ${preDistPhases[*]:-} distPhase ${postPhases[*]:-}" genericBuild
   ```

5. You will be left in the `build` directory


Create a patch with git
```
$ git diff -a > nixpkgs/pkgs/the/package/0001-changes.patch
```

Keep the temp build directory in case something fails during the build
```
nix-build -K
```

   1. Open the repl
      ```bash
      $ nix repl -f '<nixpkgs>'
      ```
   2. Load the package in your editor
      ```
      nix-repl> :e pkgs.prismlauncher
      ```
   3. Write out the file to another path
      ```vim
      :w ~/Downloads/default.nix
      ```
   4. Exit the repl
      ```
      nix-repl> :q
      ```

2. Build the package
   ```bash
   $ nix-build -K -E 'with import <nixpkgs> {}; callPackage ./default.nix {}'
   ```

2. Clone the source
   ```bash
   $ git clone --depth 1 https://github.com/PrismLauncher/PrismLauncher -b 8.0
   ```

### Build derivation for testing
* [elatov's building a nix package](https://elatov.github.io/2022/01/building-a-nix-package)

1. Save off the hello derivation
   1. Open the repl
      ```bash
      $ nix repl -f '<nixpkgs>'
      ```
   2. Load the package in your editor
      ```
      nix-repl> :e pkgs.hello
      ```
   3. Write out the file to another path
      ```vim
      :w ~/Downloads/default.nix
      ```
   4. Exit the repl
      ```
      nix-repl> :q
      ```

2. Build the package
   ```bash
   $ nix-build -K -E 'with import <nixpkgs> {}; callPackage ./default.nix {}'
   ```

3. Once the build is finished it prints out its location in the store and drop out the `result` link
   ```bash
   ...
   result -> /nix/store/zmdim73z9r135i6rjg1n2qbwj2q3hkvy-hello-2.10
   ```

4. Listing out that directory will give you
   ```bash
   $ tree -L 2 result
   result
   ├── bin
   │   └── hello
   └── share
       ├── info
       ├── locale
       └── man
   ```

## Package overrides
When declaring a package to be installed with the familiar `pkgs.git` syntax there is an alternate 
syntax that also allows for modifying the install arguments as well e.g.
`pkgs.git.override { guiSupport = true })`.

Package overrides are used in 3 different places in the system:
1. ***environment.systemPackages*** for an installation. This doesn't alter other packages that depend on 
   git and thus you will get multiple versions of git installed.
   ```nix
   {
     environment.systemPackages = [
       (pkgs.git.override { guiSupport = true; })
     ];
   }
   ```

2. ***services.<service>.package*** option overrides for `programs`
   ```nix
   {
     services.pipewire.package = pkgs.pipewire.override { x11Support = false; };
   }
   ```

3. ***overlays*** for global package replacements and additions. If you have the overlay in place you 
   won't need the `services.<service>.package` or `environment.systemPackages` versions in place.
   ```nix
   {
     nixpkgs.overlays = [
       (final: prev: {
         pipewire = prev.pipewire.override { x11Support = false; };
       })
     ];
   }
   ```

### overrideAttrs
The overrideAttrs syntax supports to function arguments. If only one is specified then the argument 
has the meaning of the previous attributes.

* [manual overrideAttrs](https://nixos.org/manual/nixpkgs/stable/#sec-pkg-overrideAttrs)

```nix
helloBar = pkgs.hello.overrideAttrs (finalAttrs: previousAttrs: {
  pname = previousAttrs.pname + "-bar";
});
```

### Determine sha of package
In order to determine the sha of a target package you can use `prefetch`

Example:
```
$ nix-prefetch-git --url "https://github.com/GarCoSim/TraceFileGen.git" --rev "4acf75b142683cc475c6b1c841a221db0753b404"
```

### Override package unwrapped
When attempting to make override changes to the derivation you need to be careful to get the right 
package name as many packages will be bundled as both a wrapped or options enabled version and an 
unwrapped or raw package. The raw or unwrapped package can be identified by the call to 
`stdenv.mkDerivation` and references to `src = fetchFromGitHub`. Patches and src changes need to 
occur on this package to take effect else the system will just quietly ignore your overrides.

* [Override not taking 
effect](https://discourse.nixos.org/t/overrideattrs-not-taking-effect-how-to-override-fetchfromgithub-and-more/32298/4)

```nix
wayland.windowManager.sway = {
  package = pkgs.sway.override (previous: {
    sway-unwrapped = previous.sway-unwrapped.overrideAttrs (previousAttrs: {
      src = previousAttrs.src.override {
        rev = ...
      };
    });
  });
```

### override vs overrideAttrs
`override` allows for controlling package options during installation while `overrideAttrs` provides 
deep level attribute overrides for patches and src changes.

***override*** example
```nix
environment.systemPackages = [
  (git.override { guiSupport = true; curl = curlFull; })
];
```

***overrideAttrs*** Patch example
```nix
environment.systemPackages = [
  (curl.overrideAttrs (previousAttrs: {
    patches = previousAttrs.patches ++ [ ./my-patch.patch ];
  }))
];
```


## Patch packages
Often packages will need a small change here or there to the `source`. Nix provides a way to apply 
patch files to the downloaded source and rebuilding with the original configuration.

### Patching using overrides
You can customize a Nix Package beyond simple options by using the `override` function to change the 
derivation.

* [NixOS and Flakes Overrides](https://nixos-and-flakes.thiscute.world/nixpkgs/overriding)
* [Customizing packages](https://bobvanderlinden.me/customizing-packages-in-nix/)

Using `Prism Launcher` for my example
1. Review the target derivation
   1. Open the repl
      ```bash
      $ nix repl -f '<nixpkgs>'
      ```
   2. Load the package in your editor
      ```
      $ :e pkgs.prismlauncher-unwrapped
      ```
   3. Note the `version = "9.1";` in the `fetchFromGitHub` block

2. Create the desired patch file using the `9.1` tag from above
   ```bash
   $ git clone --depth 1 https://github.com/PrismLauncher/PrismLauncher -b 9.1
   $ cp -a PrismLauncher a
   $ cp -a PrismLauncher b
   $ cd b
   # make desired changes in 'b'
   $ cd ..
   $ diff -ruN a b > ~/Projects/nixos-config/patches/prismlauncher/v9.1/offline.patch
   ```

3. Override the systemPackages to use the modified target package. Originally as much as I tried and 
   double checked my syntax a million times I couldn't get any patch changes to take affect. Finally 
   I realized that I was attempting to patch the `prismlauncher` package which has no source. It is 
   the underlying `prismlauncher-unwrapped` that actually needs to have the patch applied to.
   ```nix
   environment.systemPackages = with pkgs; [
     jdk17
     (prismlauncher.override (prev: {
       prismlauncher-unwrapped = prev.prismlauncher-unwrapped.overrideAttrs (o: {
         patches = (o.patches or [ ]) ++ [ ./patches/prismlauncher/offline.patch ];
       });
     }))
   ];
   ```

### Patching using overlays
Overlays are Nix functions which accept two arguments, conventionally called `final` and `prev` 
(formerly also `self` and `super`), and return a set of packages. They allow for overriding 
derivations globally.

* [NixOS Overlays](https://nixos.wiki/wiki/Overlays)
* [NixOs and Flake - Overlays](https://nixos-and-flakes.thiscute.world/nixpkgs/overlays)
* [How to patch in overlay?](https://discourse.nixos.org/t/how-to-patch-in-an-overlay/3678/3)

1. Review the target derivation
   1. Open the repl
      ```bash
      $ nix repl -f '<nixpkgs>'
      ```
   2. Load the package in your editor
      ```
      $ :e pkgs.prismlauncher
      ```

2. Create the desired patch file
   ```bash
   $ git clone https://github.com/PrismLauncher/PrismLauncher
   $ cp -a PrismLauncher a
   $ cp -a PrismLauncher b
   $ cd b
   # make desired changes in 'b'
   $ cd ..
   $ diff -ruN a b > /etc/nixos/patches/offline.patch
   ```

3. Add an over lay section to your nixpkgs configuration in your `/etc/nixos/flake.nix`
  * NOT SURE ON THE SYNTAX HERE - need to make it match systemPackages example above
   ```nix
   pkgs = import nixpkgs {
     inherit system;
     config.allowUnfree = true;
     overlays = [
       (final: prev: {
         prismlauncher.override (prev: {
           prismlauncher-unwrapped = prev.prismlauncher-unwrapped.overrideAttrs (o: {
             patches = (o.patches or [ ]) ++ [ ./patches/prismlauncher/offline.patch ];
           });
         });
       })
     ];
   };
   ```

* NOT SURE ON THE SYNTAX HERE
   ```nix
   pkgs = import nixpkgs {
     inherit system;
     config.allowUnfree = true;
     overlays = [
       (final: prev: {
         prismlauncher = prev.prismlauncher.overrideAttrs (o: {
           patches = (o.patches or [ ]) ++ [
             ./patches/prismlauncher/offline.patch
           ];
         });
       })
     ];
   };
   ```
