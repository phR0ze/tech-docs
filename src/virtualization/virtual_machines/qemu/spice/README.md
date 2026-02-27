# SPICE <img style="margin: 6px 13px 0px 0px" align="left" src="../../../../data/images/logo_36x36.png" />

The [SPICE](https://www.spice-space.org/) project aims to provide a complete open source solution for 
remote access to virtual machines in a seamless way so you can play video, record audio, share usb 
devices and share folders without complications. SPICE is a single seat solution, which depending on 
your use case might be a benefit or a show stopper.

### Quick links
- [.. up dir](../README.md)
- [Overview](#overview)
- [Configuration](#configuration)
  - [Host configuration](#host-configuration)
  - [Guest configuration](#guest-configuration)
- [Sound](#sound)
- [Advanced video](#advanced-video)

## Overview
SPICE is a contender in the Virtual Desktop Infrastructure (VDI) space. The project aims to provide a 
complete open source solution for remote access to virtual machines in a seamless way so you can play 
videos, record audio, share usb devices and share folders without complications. SPICE is composed of 
4 components: `Protocol`, `Client`, `Server`, and `Guest`.

**References**
- [SPICE](https://www.spice-space.org/index.html)
- [SPICE and QEMU overview](https://linux-blog.anracom.com/2021/02/26/kvm-qemu-vms-with-multi-screen-spice-console-i-overview-over-local-and-remote-access-methods/)
- [remote-viewer - redhat](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/virtualization_deployment_and_administration_guide/sect-graphic_user_interface_tools_for_guest_virtual_machine_management-remote_viewer)

SPICE is a single seat solution, meaning that only a single user is able to view and work with the 
virtual desktop at a time and if you were to login elsewhere you would take over the current session. 
This is fantastic for managing a VM remotely and actually the behavior I prefer for this scenario.

* The SPICE server library is fully integrated with QEMU
* ***virt-viewer*** is the recommended client being fully integrated with `remote-viewer`
  * `remote-viewer` is usually installed along with virt-viewer and can be used as standalone as well
  * `remote-viewer spice://<host>:5900`
* ***Xspice*** is a stadnalone server that is both an X server and a Spice server
  * allows for running GUI applications in a container with Xspice
* `5900` is the default port for SPICE and also for VNC

## Configuration
Basic nix configuration for enabling SPICE with QEMU consists of two parts the host and the guest.

### Host configuration
One the host side you'll want to install and configure QEMU and libvirt

```nix
  virtualisation.libvirtd = {
    enable = true;
    qemu.swtpm.enable = true;                   # Configure windows swtpm
    qemu.ovmf.enable = true;                    # Configure UEFI support
    qemu.ovmf.packages = [ pkgs.OVMFFull.fd ];  # Configure UEFI support
    qemu.vhostUserPackages = [ pkgs.virtiofsd ];  # virtiofs support
  };

  environment.sessionVariables.LIBVIRT_DEFAULT_URI = [ "qemu:///system" ];

  # Enables the use of qemu-bridge-helper for `type = "bridge"` interface.
  environment.etc."qemu/bridge.conf".text = lib.mkDefault ''
    allow all
  '';

  # Allow nested virtualisation
  boot.extraModprobeConfig = "options kvm_intel nested=1";

  services.spice-webdavd.enable = true;             # File sharing support between Host and Guest
  virtualisation.spiceUSBRedirection.enable = true; # Support USB passthrough to VMs from host

  # Additional packages
  environment.systemPackages = with pkgs; [
    virt-viewer       # Provides SPICE client `remote-viewer spice:://<host>:5900`
    spice-gtk         # Provides GTK SPICE client `spicy`
    spice-protocol    # SPICE support
    win-virtio        # QEMU support for windows
    win-spice         # SPICE support for windows
    # quickemu        # Create Windows VM easily
  ];

  users.users.${machine.user.name}.extraGroups = [ "kvm" "libvirtd" "qemu-libvirtd" ];
})
```

### Guest configuration
For a nix guest OS you need to specify:
* use `qxl` as your video driver for higher performing video
* enable the `vdagentd`, `autorandr` to provide smooth video scaling
* define the spice port and disable ticketing

```nix
(lib.mkIf (machine.type.vm && guest.spice) {

  # Configure QEMU for SPICE
  virtualisation.qemu.options = [
    "-vga qxl"
    "-spice port=${toString guest.spicePort},disable-ticketing=on"
    "-device virtio-serial"
    "-chardev spicevmc,id=vdagent,debug=0,name=vdagent"
    "-device virtserialport,chardev=vdagent,name=com.redhat.spice.0"
  ];

  # Configure SPICE related services
  services.spice-vdagentd.enable = true;        # SPICE agent to be run on the guest OS
  services.spice-autorandr.enable = true;       # Automatically adjust resolution of guest to spice client size
  services.spice-webdavd.enable = true;         # Enable file sharing on guest to allow access from host

  # Configure higher performance graphics for SPICE
  services.xserver.videoDrivers = [ "qxl" ];
  environment.systemPackages = [ pkgs.xorg.xf86videoqxl ];

  # Open up the firewall for machine.vm.spicePort
  networking.firewall.allowedTCPPorts = [ guest.spicePort ];
})
```

## Sound
QEMU supports pipewire out of the box:
```
export PIPEWIRE_RUNTIME_DIR=/run/user/1000
qemu-system-x86_64 \
    -audiodev pipewire,id=snd0 \
    -device hda-micro,audiodev=snd0
```

```
-device virtio-sound-pci,audiodev=snd0 -audiodev alsa,id=snd0
```

```
-device intel-hda -device hda-duplex
```

## Advanced Video
By default the recommended SPICE video driver `qxl` only uses 16MB of memory for framebuffering, but 
needs to be set to 32MB or better yet 64MB for higher performance.

* [Linux blog kvmqemu](https://linux--blog-anracom-com.translate.goog/2017/07/06/kvmqemu-mit-qxl-hohe-aufloesungen-und-virtuelle-monitore-im-gastsystem-definieren-und-nutzen-i/?_x_tr_sl=auto&_x_tr_tl=en&_x_tr_hl=en&_x_tr_pto=wapp)

* Intel's GVTg allows you to share your host GPU with the guest
* `-spice <spice_options>`

### QXL options
* `ram_size`

* Default values: `ram='65536' vram='65536' vram64='0' vgamem='16384'`
* `vgamem` is the frame buffer size
