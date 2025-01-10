# Secrets <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

### Quick links
- [.. up dir](..)
- [Overview](#overview)

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
at large. It is enterprise ready and battle tested.

## sops-nix
sops-nix brings SOPS support to NixOS configurations.

Note:
* SOPS will automatically look for `$XDG_CONFIG_HOME/sops/age/keys.txt` for the private key to decrypt.
* Age requires Ed25519 keys rather than the venerable RSA
* It is safe to give out your public key to anyone on the internet
* Best practice to encrypt your home directory or better yet your whole machine
* `~/.ssh/authorized_keys` is the public keys of systems that are allowed to login
* `ssh-copy-id root@192.168.1.3` is an easy method for installing your ssh pub key on an target 
system

1. Setup SOPS independent of nix to start with
   1. Generate a new modern SSH key, (DSA, RSA, ECDSA are all suss), `Ed25519` is recommended option. 
      Use the `-a 100` to increase the randomness of the key generation.
      ```bash
      $ cd ~/.ssh
      $ ssh-keygen -a 100 -t ed25519 -f ~/.ssh/id_ed25519 -C "email@gmail.com"
      ```
   2. Generate an age key based on your SSH key
      ```bash
      $ nix run nixpkgs#ssh-to-age -- -private-key -i ~/.ssh/id_ed25519 -o ~/.config/sops/age/keys.txt
      ```
   3. Generate a public key based on your private key
      ```bash
      $ nix shell nixpkgs#age -c age-keygen -y ~/.config/sops/age/keys.txt
      ```
   4. Create a `.sops.yaml` and `secrets` dir alongside your `flake.nix` and `configuration.nix`
      ```yaml
      keys:
        # Public key extracted via `nix shell nixpkgs#age -c age-keygen -y ~/.config/sops/age/keys.txt`
        - &primary age12hd2w2xuh70a0yrz72v79p6x8juajcn2ntt26enrm5utkjgueq0qxzdy3w
      creation_rules:
        # Defines where to find your secrets relative to your .sops.yaml file and keys to use
        - path_regex: secrets.yaml$
          key_groups:
            - age:
              - *primary
      ```
   5. Add secrets using your preferred editor i.e. `$EDITOR`
      ```bash
      $ nix-shell -p sops --run "sops secrets.yaml"
      ```
   6. For this example my `secrets.yaml` will be the sops-nix example
      ```yaml
      # Files must always have a string value
      example-key: example-value

      # Nesting the key results in the creation of directories.
      # These directories will be owned by root:keys and have permissions 0751 by default
      myservice:
        my_subdir:
          my_secret: password1
      ```
   7. Editing can be done the same way
      ```bash
      $ nix-shell -p sops --run "sops secrets.yaml"
      ```
2. Integrate with your NixOS configuration
   1. Add the following to your flake
      ```nix
      inputs = {
        inputs.sops-nix.url = "github:Mic92/sops-nix";
        # Optionally
        inputs.sops-nix.inputs.nixpkgs.follows = "nixpkgs";
      };
      outputs = { self, nixpkgs, ... }@inputs:
        let
          system = "x86_64-linux";
          pkgs = nixpkgs.legacyPackages.${system};
        in
        {
          nixosConfigurations = {
            hostname"= nixpkgs.lib.nixosSystem {
              # pass the inputs to special args
              specialArgs = { inherit inputs; };
              modules = [ ./configuration.nix ];
            };
          };
        };
      ```
   2. Open `configuration.nix` and add sops integration

      ```nix
      { pkgs, inputs, config, ... }: {
        imports = [
          inputs.sops-nix.nixosModules.sops
        ];

        # Adds the encrypted secrets.yaml to the nix store
        sops.defaultSopsFile = ./secrets.yaml;

        # yaml is the default already so this is not really needed
        sops.defaultSopsFormat = "yaml";

        # Make sure your keys.txt only contains the key no extra comments
        sops.age.keyFile = "/home/<user>/.config/sops/age/keys.txt";

        # Specify which secrets to decrypt to /run/secrets 
        sops.secrets.example-key = {
          mode = "0440";
          owner = config.users.users.<user>.name:
          group = config.users.users.<user>.group:
        };
        sops.secrets."myservice/my_subdir/my_secret" = {};
      }
      ```
   3. `nixos-rebuild switch` will decrypt the secrets to `/run/secrets` with sub fields falling into 
      different directories e.g. `myservice.my_subdir` is `myservice/my_subdir`
      ```nix
      { config, ... }: {
        services.example = {
          tokenPath = config.sops.secrets.example-key.path;
        };
      }
      ```

   4. sops-nix has to run after NixOS creates users (in order to specify what users own a secret). 
      Thus `users.users.<name>.hashedPasswordFile` won't work, but if you use the sops.nix 
      `neededForUsers = true;` in a secret it will decrypt to `/run/secrets-for-users` instead of 
      `/run/secrets` and before NixOS creates users thus you can set the user's password based on
      ```nix
      { ... }: {
        sops.secrets.userpass.neededForUsers = true;
        users.users.my-user = {
          isNormalUser = true;
          hashedPasswordFile = config.sops.secrets.userpass.path;
        };
      }

      ```
