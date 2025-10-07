# Sunshine <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Sunshine is part of the new era of remoting software taking advantage of faster networks and modern 
video/audio streaming protocols. Primarily built for gaming it also provides an excellent remote 
desktop experience with 1080p video and audio forwarded.

**Summary**
* Pros
  * Available in nixpkgs
  * Seems to work pretty well
* Cons
  * A little brittle, the moonlight client keeps crashing
* Result
  * Still experimenting with it

### Quick links
* [../](../README.md)
* [Overview](#overview)
* [Sunshine](#sunshine)
  * [Manual firewall settings](#manual-firewall-settings)
  * [Add Intel GPU support](#add-intel-gpu-support)
* [Moonlight](#moonlight)

## Overview

**References**
* [Sunshine - Github](https://github.com/LizardByte/Sunshine)
* [Apollo and Artemis](https://www.joeysretrohandhelds.com/guides/apollo-artemis-streaming-setup-guide/)
* [Artemis Android](https://github.com/ClassicOldSong/moonlight-android)
* [Apollo - Github](https://github.com/ClassicOldSong/Apollo)

## Sunshine
Sunshine is configured as a Systemd user unit, and will start automatically on login to a graphical 
session. This means the initial installation will likely require a logout/login or restart to trigger 
it or manually start it.

The web UI can be found at `https://localhost:47990/`

**References**
* [Sunshine - NixOS Wiki](https://wiki.nixos.org/wiki/Sunshine)

```nix
services.sunshine = {
  enable = true;
  autoStart = true;
  openFirewall = true;
  settings.port = 47989;
};
```

### Manual firewall settings
This is not necessary but is useful to know
```nix
networking.firewall = {
  enable = true;
  allowedTCPPorts = [ 47984 47989 47990 48010 ];
  allowedUDPPortRanges = [
    { from = 47998; to = 48000; }
    { from = 8000; to = 8010; }
  ];
};
```

### Add Intel GPU support
```nix
{
  hardware.opengl = {
    enable = true;
    extraPackages = with pkgs; [
      # your Open GL, Vulkan and VAAPI drivers
      vpl-gpu-rt # for newer GPUs on NixOS &gt;24.05 or unstable
      # onevpl-intel-gpu  # for newer GPUs on NixOS &lt;= 24.05
      # Below was required for intel Arc GPU's
      intel-media-driver
      # intel-media-sdk   # for older GPUs
    ];
  };
}
```

### Add Nvidia GPU support
```nix
services.xserver.videoDrivers = ["modesetting"];
boot.blacklistedKernelModules = ["nouveau"];
{
  hardware = {
    nvidia = {
      modesetting = {
        enable = true;
      };

      optimus_prime = {
        enable = true;
        # values are from lspci
        # try lspci | grep -P 'VGA|3D'
        intelBusId = "PCI:0:2:0";
        nvidiaBusId = "PCI:1:0:0";
      };
    };
  };

  services = {
    xserver = {
      videoDrivers = [
        "nvidiaBeta" # nvidia should work fine as well
      ];
    };
  };
}
```

## Moonlight
**References**
* [Moonlight setup](https://github.com/moonlight-stream/moonlight-docs/wiki/Setup-Guide)

```nix
environment.systemPackages = with pkgs; [
  moonlight-qt
];
```


