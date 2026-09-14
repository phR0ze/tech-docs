# nixos-consume <img style="margin: 6px 13px 0px 0px" align="left" src="../../../../data/images/logo_36x36.png" />

`nixos-consume` is a oneliner curl-bash script that converts a freshly deployed, minimal Ubuntu VPS
into a clean, flake-managed `NixOS` system by `kexec`-ing directly into an ephemeral `NixOS`
installer — no reboot, no ISO boot, and no `nix-channel` involved at any point.

This page uses [phR0ze's nixos-consume](https://github.com/phR0ze/nixos-consume), a from-scratch
rewrite that borrows heavily from [nixos-infect](https://github.com/elitak/nixos-infect) but targets
the newer kexec-based `NixOS` unstable installer flow and is purely flake based.

### Quick links
- [.. up dir](..)
- [Overview](#overview)
- [Prerequisites](#prerequisites)
  - [Disk space](#disk-space)
  - [RAM](#ram)
  - [SSH keys](#ssh-keys)
- [Usage](#usage)
  - [Convert your VPS](#convert-your-vps)
  - [Environment variables](#environment-variables)
  - [Tested Linux distros and VPS providers](#tested-linux-distros-and-vps-providers)
- [Development](#development)
  - [Running unpublished changes](#running-unpublished-changes)
  - [Testing with a local quickemu VM](#testing-with-a-local-quickemu-vm)
- [Gotchas](#gotchas)

## Overview
`nixos-consume` interrogates the currently running Linux host to determine key system details — disk
layout, hostname, SSH keys, and networking, among other things. Using those details it builds an
ephemeral `NixOS` installer entirely within `/nix/store` on the *original* filesystem, then `kexec`s
directly into it, no reboot necessary. From that ephemeral installer it formats the original root
partition, writes out the target flake-based configuration, runs `nixos-install` unattended, and
reboots into the finished system on success.

***WARNING*** — this is a completely destructive approach to installing `NixOS` on an existing
foreign Linux system. `nixos-consume` will entirely consume the existing OS, leaving it potentially
unusable if the conversion fails and you don't have console or other out-of-band access to recover.
VPS provider tools can typically wipe and re-provision from scratch if that happens, but there is
inherent risk in overwriting a system in place.

**References**
* [phR0ze/nixos-consume on GitHub](https://github.com/phR0ze/nixos-consume)
* [nixos-infect](https://github.com/elitak/nixos-infect) — the project this borrows heavily from
* [NixOS Manual: Installing from another Linux distro](https://nixos.org/manual/nixos/stable/#sec-installing-from-other-distro)

## Prerequisites

### Disk space
Plan on **at least a 20GB disk**. The ephemeral kexec installer has to be built into `/nix/store` on
the *original* filesystem, alongside the foreign Linux, before the `kexec` jump ever happens. In
testing, a 10GB disk ran out of space mid-build with `nix build` failing outright. If you hit `error:
... note: build failure may have been caused by lack of free disk space`, this is why.

### RAM
Plan **at least 2GB of RAM** — even that is tight. The kexec installer uses `netboot-minimal.nix`
(network install, no offline channel copy) instead of the larger `netboot-base.nix`, which failed
outright on a 2GB test VM (`EINVAL` / kernel page fault). Target VPS systems typically have limited
RAM due to cost, which is why the following is enabled by default:
```nix
# Default NixOS recommended value, set as the highest swap priority
zramSwap = { enable = true; memoryPercent = 50; priority = 100; algorithm = "zstd"; };

# Cheap fallback insurance to use at a lower priority only after zramSwap is full
swapDevices = [{ device = "/var/swapfile"; size = 2048; priority = 5; }];
```

### SSH keys
Ensure the freshly provisioned host has the root account configured to allow SSH via
`/root/.ssh/authorized_keys`. The resulting `NixOS` system has no accounts other than root and no
password — SSH'ing in with your key is the only way back into the system:
```bash
$ scp ~/.ssh/authorized_keys <user>@<host>:/tmp
$ ssh <user>@<host>
$ sudo install -m 600 -o root -g root /tmp/authorized_keys /root/.ssh/authorized_keys
```

## Usage

### Convert your VPS
Ensure your host meets the [Disk space](#disk-space) and [RAM](#ram) prerequisites and has
[SSH keys](#ssh-keys) seeded, then:

1. Provision your host using Ubuntu Server 24.04
2. Run the script straight from GitHub:
   ```bash
   $ curl https://raw.githubusercontent.com/phR0ze/nixos-consume/master/consume | NIXPKGS=nixos-25.11 bash
   ```
3. Your SSH session to the original OS drops the moment `kexec` runs (it kills the whole process
   tree). Reconnect with the same key after a few seconds — you'll land in the ephemeral installer.
   Watch progress with:
   ```bash
   $ journalctl -u consume-install -f
   ```
   You'll lose the connection again once that completes and it reboots into the final system.

If the unattended install fails, the `consume-install` unit shows `failed` in `systemctl status` and
the ephemeral installer's `sshd` stays up (it does **not** auto-reboot on failure) so you can debug
and re-run `nixos-install --root /mnt ...` by hand.

### Environment variables

| Variable                | Default       | Description                                        |
| ------------------------ | ------------- | -------------------------------------------------- |
| `UNATTENDED=y`          | unset         | Skip confirmation prompt                           |
| `NIXPKGS`               | `25.11`       | Nixpkgs flake ref; also sets stateVersion          |
| `NIXOS_FLAKE=<url>`     | unset         | Custom flake.nix, fetched instead of generated     |
| `NIXOS_CONFIG=<url>`    | unset         | Custom configuration.nix (fetched, not generated)  |
| `STATIC_IP=y`           | auto          | Force static network config (else auto-detected)   |
| `FALLBACK_SWAP=n`       | `y`           | Skip the target's fallback disk swapfile           |
| `MEM_TUNING=n`          | `y`           | Skip sysctl/oomd memory tuning on target           |
| `KEXEC=n`               | `y`           | Stop before kexec; leaves configs for inspection   |
| `NIX_INSTALL_URL=<url>` | nixos.org URL | Override Nix installer URL                         |
| `SERIAL_CONSOLE=y`      | unset         | Add serial console kernel params                   |

### Tested Linux distros and VPS providers

| Distro            | Flavor          | Version   | Hosting             | Firmware |
| ----------------- | --------------- | --------- | ------------------- | -------- |
| Ubuntu            | Server          | 24.04     | KVM/QEMU            | EFI      |
| Ubuntu            | Server          | 24.04     | KVM/QEMU            | BIOS     |

## Development

### Running unpublished changes
If you're testing against a not-yet-pushed copy of the script, copy it over and run it directly
instead of pulling from GitHub:
```bash
$ scp consume <user>@<host>:/tmp/consume
$ ssh <user>@<host>
$ sudo NIXPKGS=nixos-unstable bash -x /tmp/consume
```

### Testing with a local quickemu VM
Given the destructive nature of this script, don't iterate against a real host — test against a
disposable local VM instead (see [quickemu](../../../virtualization/virtual_machines/quickemu/README.md)
if you need one set up). The workflow used to develop and test the script:

1. Provision a fresh Ubuntu Server 24.04 VM (at least 20G disk, 2G RAM — see
   [Prerequisites](#prerequisites)) and get it to a normal logged-in state (a non-root user with a
   password, `openssh-server` installed). For a BIOS-firmware VM, Ubuntu's
   [autoinstall](https://ubuntu.com/server/docs/install/autoinstall) can build this unattended by
   booting the ISO's `casper/vmlinuz`+`casper/initrd` directly with `-append "autoinstall
   ds=nocloud; ..."` and a small NoCloud seed ISO, bypassing the interactive installer entirely.
2. Take a `qemu-img snapshot -c <tag>` of that baseline *before* ever running `consume` against it,
   so you can revert and retest repeatedly without re-provisioning.
3. Before each test run: revert to the baseline snapshot (`qemu-img snapshot -a <tag> disk.qcow2`),
   boot the VM, and seed `/root/.ssh/authorized_keys` (a manual prerequisite `consume` itself checks
   for — see [SSH keys](#ssh-keys)):
   ```bash
   $ ssh <user>@<host> "sudo mkdir -p /root/.ssh && sudo tee -a /root/.ssh/authorized_keys" < ~/.ssh/id_ed25519.pub
   ```
4. Copy over and run the script per [Running unpublished changes](#running-unpublished-changes)
   above, adding `UNATTENDED=y` to skip the confirmation prompt for a hands-off run.
5. Watch for the SSH session dropping (the `kexec` jump), reconnect, and watch `journalctl -u
   consume-install -f` in the ephemeral installer until it reboots.
6. Verify the result: `systemctl --failed` (expect none), `systemctl is-system-running` (expect
   `running`), and that a full `reboot` survives cleanly.
7. Revert to the step-2 snapshot again before the next test run — `consume` is destructive and there
   is no in-place undo once `kexec` has run.

This same procedure works for exercising the BIOS/GRUB code path specifically (a separate baseline VM
built with `boot="legacy"` instead of the default EFI firmware) and the `STATIC_IP=y` path (force it
with the env var even on a DHCP VM, and check the rendered `networking.nix` looks sane — this path
doesn't currently have a dedicated test VM).

## Gotchas
* This is a destructive, irreversible conversion of the running system — take a provider-level
  snapshot or backup first if one is available
* Networking config (especially static IPs on providers without DHCP) is inferred from the running
  system; verify the rendered configuration after conversion before relying on it for future rebuilds
* A root spanning multiple physical disks (e.g. LVM/mdadm striped across disks) is refused rather than
  guessed at — `consume` only supports the single-disk KVM/QEMU targets it's actually been tested
  against
* Check the [environment variables](#environment-variables) table for the current list of supported
  overrides since these change between releases
