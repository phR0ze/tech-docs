# sops-nix <img style="margin: 6px 13px 0px 0px" align="left" src="../../../../data/images/logo_36x36.png" />

[sops-nix](https://github.com/Mic92/sops-nix) brings [SOPS](https://github.com/getsops/sops) 
support to NixOS configurations, keeping secrets encrypted at rest in git and in the nix store 
while allowing multiple secrets to be grouped in a single file.

### Quick links
* [.. up dir](..)
* [Overview](#overview)
* [Sops-nix Setup](#sops-nix-setup)
  * [SOPS Setup](#sops-setup)
  * [NixOS Integration](#nixos-integration)
* [Checking Secret Installation Status](#checking-secret-installation-status)
  * [First Boot on a Fresh Machine](#first-boot-on-a-fresh-machine)

## Overview
* SOPS will automatically look for `$XDG_CONFIG_HOME/sops/age/keys.txt` for the private key to decrypt.
* Age requires Ed25519 keys rather than the venerable RSA
* It is safe to give out your public key to anyone on the internet
* `~/.ssh/authorized_keys` is the public keys of systems that are allowed to login
* `ssh-copy-id root@<TARGET-IP>` is an easy method for installing your ssh pub key on an target
system

## Sops-nix Setup

### SOPS Setup
Setup SOPS independent of nix to start with

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

### NixOS Integration
Integrate with your NixOS configuration

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

## Checking Secret Installation Status
By default `sops.useSystemdActivation` is `false` (it's only auto-enabled when
`systemd.sysusers.enable` or `services.userborn.enable` is set). In this default mode secrets are
installed via a plain NixOS **activation script** (`system.activationScripts.setupSecrets`
running `sops-install-secrets`), not a systemd service — so there is nothing to
`systemctl status`/`systemctl restart`. Confirmed directly in `modules/sops/default.nix` in the
sops-nix source.

* If you just ran `nixos-rebuild switch` interactively, the activation output (including any
  `sops-install-secrets` errors) prints straight to your terminal in real time — that's the
  primary feedback.
* On boot (not a manual switch), that same activation output goes to the kernel log buffer instead
  of a systemd journal unit:
  ```bash
  $ journalctl -k -b | grep -i sops
  # or
  $ dmesg | grep -i sops
  ```
* The most reliable check regardless of when activation ran is just to look at the result:
  ```bash
  $ ls -la /run/secrets/
  $ cat /run/secrets/example-key
  ```
  If the secret exists there with the right content, activation succeeded. If it's missing,
  activation either hasn't run yet or failed silently from your perspective (check the journal/
  dmesg for why — most likely a missing or unreadable `sops.age.keyFile`).

If you want a real, independently restartable unit instead (useful once you have many secrets and
don't want to force a full `nixos-rebuild switch` just to reinstall them), opt in explicitly with
`sops.useSystemdActivation = true;`. This creates `systemd.services.sops-install-secrets`
(`Type = "oneshot"`, `RemainAfterExit = true`, `wantedBy = [ "sysinit.target" ]`), which you can
then query and restart:
```bash
$ systemctl status sops-install-secrets
$ sudo systemctl restart sops-install-secrets
```
Note the unit name is `sops-install-secrets`, not `sops-nix`.

**Caveat when enabling this manually** (i.e. without `systemd.sysusers.enable` or
`services.userborn.enable`, which normally auto-enable it): the unit runs at `sysinit.target`,
which is *before* NixOS's traditional `users.users.*`/`users.groups.*` activation step. A secret
with `owner`/`group` set to a non-root user created the normal NixOS way may fail to `chown`
because that user doesn't exist yet at that point in boot. Secrets left at the default owner
(`root:root`) are unaffected, since `root` always exists. If you hit this, either enable
`systemd.sysusers.enable`/`services.userborn.enable` (which create users declaratively, early
enough for `sysinit.target` ordering to work), or leave the affected secret on the default
activation-script path.

### First Boot on a Fresh Machine
A common bootstrap scenario: you install from an ISO that has no age key, boot the new machine,
then copy the age key over from an existing machine afterward. This does **not** fail the boot:

1. Fresh install boots with `sops.age.keyFile` missing → the `setupSecrets` activation script
   errors out (visible via `dmesg`/`journalctl -k`), but activation-script failures are
   non-fatal in NixOS — the rest of boot proceeds normally, you still get a working login/SSH
   session, `/run/secrets/*` is just empty.
2. Copy the age key onto the new machine (e.g. `scp` it into place at the path configured in
   `sops.age.keyFile`).
3. Re-trigger secret installation — no reboot required. Simplest option is to just rerun
   `nixos-rebuild switch`, which reruns all activation scripts including `setupSecrets` now that
   the key is present. Any service wired to the secret via `sops.secrets.<name>.restartUnits`
   will also be restarted as part of that.
