# GNOME Boxes <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

***GNOME Boxes*** is a simplified GUI front-end for libvirt/QEMU/KVM, aimed at "click and go" VM
creation rather than the full control `virt-manager` exposes.

### Quick links
- [.. up dir](../README.md)
- [Overview](#overview)
  - [Flatpak Attempt](#flatpak-attempt)
    - [Glycin Sandbox Crash](#glycin-sandbox-crash)
  - [Nix devShell Approach](#nix-devshell-approach)
    - [Session libvirtd](#session-libvirtd)
    - [Data Locations](#data-locations)
    - [SSH Access](#ssh-access)
    - [Passwordless Root Login](#passwordless-root-login)
- [Shared Folders](#shared-folders)
  - [Guest Requirements](#guest-requirements)
  - [Headless Guest Troubleshooting](#headless-guest-troubleshooting)
- [Getting Started](#getting-started)

## Overview
Boxes talks to the same `libvirtd` → `QEMU` → `KVM` stack as [VirtManager](../virt_manager/README.md),
just through a much simpler UI — no XML editing, no advanced device configuration. Upstream is
mid-rewrite: the GTK4/libadwaita port (v51 beta) is landing as a **Flatpak-only** release, which is
why it's worth trying via Flatpak first before falling back to the native `nixpkgs` package.

**References**
- [GNOME Boxes - GitLab](https://gitlab.gnome.org/GNOME/gnome-boxes)
- [NixOS Wiki: Virt-manager](https://wiki.nixos.org/wiki/Virt-manager)
- [NixOS Wiki: Libvirt](https://wiki.nixos.org/wiki/Libvirt)

### Flatpak Attempt
Since upstream's new GTK4 rewrite ships Flatpak-only, that's the natural way to try the current beta
without waiting on `nixpkgs` to catch up:

```bash
$ flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
$ flatpak install flathub org.gnome.Boxes
$ flatpak run org.gnome.Boxes
```

#### Glycin Sandbox Crash
On NixOS this crashed immediately on launch:

```
Gtk:ERROR:../gtk/gtkiconhelper.c:495:ensure_surface_for_gicon: assertion failed (error == NULL):
Failed to load /run/host/share/icons/Paper/16x16/status/image-missing.png:
Loader process exited early with status '1'
```

GTK4's image loader (`glycin`) spawns a *second*, more restrictive nested sandbox
(`flatpak-spawn --sandbox`) for every image/icon load. That nested sandbox failed on any icon theme
tried — first the host's icon theme (which resolves through `/run/host/...` into `/nix/store`
symlinks the nested sandbox can't see), then even the runtime's own bundled `Adwaita` icons after
overriding the icon theme lookup. Manually invoking the failing loader binary directly confirmed the
binary itself works fine — the failure is somewhere inside the `flatpak-spawn --sandbox` nested
confinement layer itself, not the icon files.

Attempted fixes, none of which resolved it:
- `flatpak override --user --env=XDG_DATA_DIRS=...` to stop it reaching for the host icon theme
- `flatpak override --user --env=GLYCIN_DISABLE_SANDBOX=i-know-the-risks` to skip the nested sandbox
- Restarting `flatpak-session-helper` in case a stale process was ignoring the new override
- Passing the env var directly via `flatpak run --env=...` instead of a persisted override

This appears to be a genuine bug in this glycin/runtime version's Flatpak nested-sandboxing path,
also reported by users on non-NixOS distros — not something specific to this system's config.

### Nix devShell Approach
Rather than keep chasing the Flatpak sandboxing bug, the native `nixpkgs` package sidesteps it
entirely — it doesn't go through Flatpak's extra `flatpak-spawn --sandbox` layer at all. A
`nix develop` shell lets you try it without touching system config (`virtualisation.libvirtd`,
group membership, etc.):

```nix
# flake.nix
{
  description = "Temporary devShell for trying QEMU/KVM GUI front-ends without touching system config";

  inputs.nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";

  outputs = { self, nixpkgs }:
    let
      system = "x86_64-linux";
      pkgs = import nixpkgs { inherit system; };
    in {
      devShells.${system}.default = pkgs.mkShell {
        packages = with pkgs; [
          gnome-boxes
          virt-manager
          libvirt
          qemu_kvm
          spice-gtk
          virtiofsd
          passt
          OVMFFull.fd
        ];

        shellHook = ''
          export LIBVIRT_DEFAULT_URI=qemu:///session

          state="$PWD/.libvirt-session"
          mkdir -p "$state/config" "$state/run" "$XDG_RUNTIME_DIR/libvirt"

          sock="$XDG_RUNTIME_DIR/libvirt/libvirt-sock"
          pidfile="$state/run/libvirtd.pid"

          if [ ! -S "$sock" ]; then
            echo "Starting session-mode libvirtd (qemu:///session, no root needed)..."
            libvirtd --session \
              --config "$state/config/libvirtd.conf" \
              --pid-file "$pidfile" \
              > "$state/libvirtd.log" 2>&1 &
            echo $! > "$state/libvirtd.shellpid"
            sleep 1
          fi

          cleanup() {
            if [ -f "$state/libvirtd.shellpid" ]; then
              kill "$(cat "$state/libvirtd.shellpid")" 2>/dev/null
              rm -f "$state/libvirtd.shellpid"
            fi
          }
          trap cleanup EXIT

          echo "Ready. Try: gnome-boxes   or   virt-manager"
          echo "Logs: $state/libvirtd.log"
        '';
      };
    };
}
```

```bash
$ cd ~/vm-gui-shell
$ nix develop
$ gnome-boxes
```

#### Session libvirtd
`qemu:///session` runs `libvirtd` as your own user with no root and no systemd unit — a good fit for
a throwaway shell. It requires your user to already be in the `kvm` group for `/dev/kvm` access; add
that in your NixOS config (`users.users.<username>.extraGroups = [ "kvm" ];`) if it's not already set
for the host.

The `trap cleanup EXIT` kills the session `libvirtd` process when the shell exits — nothing lingers
system-wide.

#### Data Locations
The `shellHook` above only redirects the libvirt session socket into the project directory — it does
**not** redirect `XDG_CACHE_HOME`/`XDG_DATA_HOME`. That means anything Boxes creates while testing
lands in your real home directory and survives after the devShell exits:

| Data | Native package (`nix develop`) | Flatpak |
| --- | --- | --- |
| Downloaded ISOs / install media | `~/.cache/gnome-boxes/` | `~/.var/app/org.gnome.Boxes/cache/gnome-boxes/` |
| VM disk images | `~/.local/share/gnome-boxes/images/` | `~/.var/app/org.gnome.Boxes/data/gnome-boxes/images/` |
| Config | `~/.config/libvirt/`, `~/.config/gnome-boxes/` | `~/.var/app/org.gnome.Boxes/config/` |

To make a test fully throwaway — ISOs and disk images included — also override
`XDG_CACHE_HOME`/`XDG_DATA_HOME`/`XDG_CONFIG_HOME` in the `shellHook` to point at a temp directory
under the project and clean it up alongside the session `libvirtd`.

#### SSH Access
`qemu:///session` VMs commonly get plain QEMU usermode networking (no bridge at all — `virsh
domifaddr` returns nothing useful, and there's no `virbr0`-style device to attach to). To reach the
guest at all, use libvirt's **passt** backend, which supports declared port-forward rules directly in
the domain XML — the older built-in SLIRP backend does not.

1. Confirm `passt` is on `PATH` inside the devShell (added to the `packages` list above):
   ```bash
   $ which passt
   ```

2. Shut the VM down — network interface changes require it stopped:
   ```bash
   $ virsh --connect qemu:///session shutdown <vm-name>
   ```

3. Edit the domain XML:
   ```bash
   $ virsh --connect qemu:///session edit <vm-name>
   ```
   Replace the `<interface type='user'>...</interface>` block with:
   ```xml
   <interface type="user">
     <mac address="52:54:00:xx:xx:xx"/>
     <backend type="passt"/>
     <portForward proto="tcp">
       <range start="2222" to="22"/>
     </portForward>
     <model type="virtio"/>
   </interface>
   ```
   (keep the existing `<mac>` value from the original block, and requires libvirt ≥ 9.0)

4. Start it back up and connect:
   ```bash
   $ virsh --connect qemu:///session start <vm-name>
   $ ssh -p 2222 <guest-user>@127.0.0.1
   ```
   (assumes `openssh-server` is installed and running in the guest)

**Gotcha**: if you add a package like `passt` to the flake *after* the session daemons
(`virtqemud`/`virtstoraged`/`virtlogd`) are already running, they won't see it — they were launched
with the old `PATH` and libvirt reports `Cannot find 'passt' in path` on start. Kill them so they
respawn inside the current shell's `PATH`:
```bash
$ pkill -f 'virtqemud|virtstoraged|virtlogd'
```
Doing this while Boxes' GUI is open will also break its live connection silently — it keeps running
but stops reflecting real VM state (a VM can even show as available in the sidebar while actually
disconnected). Quit and relaunch `gnome-boxes` from the same shell afterward to reconnect it cleanly.

#### Passwordless Root Login
With the port-forward from [SSH Access](#ssh-access) in place, `scp` your public key over the same
forwarded port and wire it up for `root`:

1. Copy your public key to the guest:
   ```bash
   $ scp -P 2222 ~/.ssh/id_ed25519.pub <guest-user>@127.0.0.1:/tmp/key.pub
   ```

2. Install it for `root`:
   ```bash
   $ ssh -p 2222 <guest-user>@127.0.0.1
   $ sudo mkdir -p /root/.ssh
   $ sudo install -m 600 -o root -g root /tmp/key.pub /root/.ssh/authorized_keys
   $ rm /tmp/key.pub
   ```

3. Ubuntu's default `sshd_config` already ships `PermitRootLogin prohibit-password`, which allows
   key-based root login while still blocking root password auth — usually nothing to change. If
   root login is fully disabled (`PermitRootLogin no`), enable key-only access instead:
   ```bash
   $ sudo sed -i 's/^#\?PermitRootLogin .*/PermitRootLogin prohibit-password/' /etc/ssh/sshd_config
   $ sudo systemctl restart ssh
   ```

4. Log in keylessly as `root` from the host:
   ```bash
   $ ssh -p 2222 root@127.0.0.1
   ```

`prohibit-password` (not `yes`) is the safer default to keep — it still blocks a brute-forced root
password if the port ever ends up reachable beyond `127.0.0.1`, while leaving key-based access open.

## Shared Folders
Boxes' "Shared Folders" (Properties → Sharing) doesn't create a real filesystem mount by default —
it exposes the host directory over **WebDAV**, tunneled through a virtio-serial channel to
`spice-webdavd` running in the guest, which the guest then has to mount itself via GVFS:

```
/run/user/<UID>/gvfs/dav:host=127.0.0.1,port=<port>,ssl=false/
```

On a full GNOME desktop guest this all happens automatically — Nautilus mounts it on session
startup and it just shows up. On a **headless/server guest**, none of that automation exists, and
each missing piece fails silently or with a misleading error rather than a clear "X is not
installed."

### Guest Requirements
Three separate packages/processes are involved, and a minimal server image is missing all three by
default:

| Piece | Role | Package (Ubuntu) |
| --- | --- | --- |
| `spice-webdavd` | Bridges the local WebDAV endpoint over the virtio-serial channel to the host | `spice-webdavd` |
| `gio`/`gvfsd` dav backend | Lets `gio mount` understand `dav://` URIs at all | `gvfs-backends` |
| `gvfsd-fuse` | Projects the registered GVFS mount into a real path under `/run/user/<UID>/gvfs/` | `gvfs-fuse` |

```bash
$ sudo apt install spice-webdavd gvfs-backends gvfs-fuse
```

### Headless Guest Troubleshooting
Working through this on an Ubuntu Server guest surfaced three distinct, silent failure points —
each one looks like success (no error, or an unrelated warning) until the next check reveals it
isn't:

1. **`spice-webdavd` not running.** Confirm and start it:
   ```bash
   $ systemctl status spice-webdavd
   $ sudo systemctl enable --now spice-webdavd
   ```
   A benign `Referenced but unset environment variable... SPICE_WEBDAVD_EXTRA_*` warning in the logs
   is not fatal — the important line is `Active: active (running)`.

2. **Confirm the virtio-serial channel exists** (this is created by Boxes on the host side when a
   shared folder is configured for the VM — it requires a full VM restart, not just a guest reboot,
   since virtio-serial devices attach at VM startup):
   ```bash
   $ ls -l /dev/virtio-ports/
   # expect: org.spice-space.webdav.0 -> ../vportNpM
   ```

3. **`gio mount` fails with `volume doesn't implement mount`** — this means GVFS has no `dav`
   backend at all, not that the mount itself failed:
   ```bash
   $ sudo apt install gvfs-backends
   $ gio mount dav://127.0.0.1:<port>/
   $ gio mount -l   # should now list a GDaemonMount
   ```

4. **`gio mount -l` shows the mount, but `/run/user/<UID>/gvfs/` stays empty.** `gio mount` only
   registers the mount with `gvfsd` — a separate `gvfsd-fuse` process is what actually projects it
   into the filesystem. Check whether it's running, and start it manually if the guest has no
   session that would normally launch it:
   ```bash
   $ ps aux | grep gvfsd-fuse
   $ sudo apt install gvfs-fuse   # if the binary is missing entirely
   $ /usr/libexec/gvfsd-fuse /run/user/<UID>/gvfs -f &
   ```

5. Confirm the share is now populated, and optionally symlink it somewhere convenient:
   ```bash
   $ ls -la /run/user/<UID>/gvfs/
   $ ln -s /run/user/<UID>/gvfs/dav:host=127.0.0.1,port=<port>,ssl=false ~/shared
   ```

Compare this to [VirtManager](../virt_manager/README.md)'s virtiofs-based shares, which are a real
POSIX mount (`mount -t virtiofs <tag> /mnt/point`) with no GUI-session dependency at all — one of
the real trade-offs of Boxes' "simple" approach once you leave a desktop guest.

## Getting Started
1. Create a `flake.nix` like the one above in a scratch directory
2. Enter the shell
   ```bash
   $ nix develop
   ```
3. Launch Boxes
   ```bash
   $ gnome-boxes
   ```
4. Exit the shell (`Ctrl+D` or `exit`) when done — the session `libvirtd` is killed automatically

### Create a new vm
Note: Racknerd systems are usually `BIOS`

1. Load up your target ISO file
2. Name your vm
3. Choose the firmware type `BIOS` or `UEFI`
4. Set your resources as desired
