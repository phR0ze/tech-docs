# Repl <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

The Nix REPL is a powerful tool for working with the nix expression language and allows you to 
dynamnically investigate and learn the language and interrogate your system.

### Quick links
* [.. up dir](..)
* [Overview](#overview)
  * [Launch the repl](#launch-the-repl)
 
## Overview

### Launch the repl
```bash
$ nix repl -f '<nixpkgs>'
```

### Print expression result
* Evaluate an expression and store as variable then print out
  ```
  nix-repl> foo = map (x: x // { foo = x.name; } ) [ { name = "foo"; value = 2; } ]
  nix-repl> :p foo
  [ { foo = "foo"; name = "foo"; value = 2; } ]
  ```
* Evaluate and print directly
  ```
  nix-repl> :p map (x: x // { foo = x.name; } ) [ { name = "foo"; value = 2; } ]
  [ { foo = "foo"; name = "foo"; value = 2; } ]
  ```

### Evaluate expressions
* Add a new attribute to an attribute list then print it out
  ```
  nix-repl> foo = map (x: x // { foo = x.name; } ) [ { name = "foo"; value = 2; } ]
  nix-repl> :p foo
  [ { foo = "foo"; name = "foo"; value = 2; } ]
  ```
* Build a vim plugin from github and check the file structure
  ```
  nix-repl> miniPairs = vimUtils.buildVimPlugin {
    pname = "mini-pairs";
    version = "2025-10-08";
    src = fetchFromGitHub {
      owner = "nvim-mini";
      repo = "mini.pairs";
      rev = "b9aada8c0e59f2b938e98fbf4eae0799eba96ad9";
      sha256 = "sha256-KFWpyITaKc9AGhvpLkeq9B3a5MELqed2UBo9X8oC6Ac=";
    };
  }
  nix-repl> miniPairs.outPath
  "/nix/store/402i1c4dmksqziv317m2hwn2dcw9j0x3-vimplugin-mini-pairs-2025-10-08"
  ```

### Load flake from disk
* Load from the current directory
  ```
  nix-repl> :lf .
  ```
* Load from a a full path
  ```
  nix-repl> :lf /etc/nixos
  ```

### Print flake variables
Note, in my flake configuation `target` always refers to the target host under evaluation defined by 
the `clu repl` command.

* Print out shallowly a local variable
  ```
  nix-repl> :lf .
  nix-repl> nixosConfigurations
  ```
* Print out system configuration for a particular option
  ```
  nix-repl> :lf .
  nix-repl> nixosConfigurations.target.config.networking.firewall.enable
  ```
