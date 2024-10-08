# Build Source

### Quick links
* [Overview](#overview)
  * [System Dependencies](#system-dependencies)
* [Rust](#rust)

## Overview
Building a rust project that depends on system libraries takes some work.

### System Dependencies
Use a dev shell to include system dependencies e.g. `nix-shell -p pkg-config openssl gtk4 libadwaita`. 
The dev shell will read the nix development package paths and export them in a way that `pkg-config` 
will find them. This means that any build tools running in that shell will find the developement 
dependencies. Alternatively dumping out the env variables in the shell I found you can recreate the 
effect by grabbing the value of
```
PKG_CONFIG_PATH_FOR_TARGET=/nix/store/z5ghkxklksfayrri517mnp67vfj03im5-openssl-3.0.13-dev/lib/pkgconfig:/nix/store/0nb1qcnj6ha6vdzrkfkwbriw69n4nc3q-gtk4-4.12.5-dev/lib/pkgconfig:/nix/store/p78m3rmr5g1xqyizlsbg39cgbvjbjz74-cairo-1.18.0-dev/lib/pkgconfig:/nix/store/cs16dfmzrrdf88ghkh44vxwd6j17wjzn-fontconfig-2.15.0-dev/lib/pkgconfig:/nix/store/0bkz0x8vsb92kcr3inkgs0ba2dqlla5a-freetype-2.13.2-dev/lib/pkgconfig:/nix/store/hwlm6akqm83lapl79y8r9r2vw741sppd-zlib-1.3.1-dev/lib/pkgconfig:/nix/store/kh542xlg6mgb14xwz22mvqpm8810aidp-bzip2-1.0.8-dev/lib/pkgconfig:/nix/store/9kk0fm2s7c3269ph62av2l8hf95356ab-brotli-1.1.0-dev/lib/pkgconfig:/nix/store/831hplq82ppf633fv1qchlrsihhkryz4-libpng-apng-1.6.40-dev/lib/pkgconfig:/nix/store/mk3hpjywv0hkkn2bwplcb9lwvzsv2j0i-pixman-0.43.2/lib/pkgconfig:/nix/store/drz4r0b20cl5r4lsw600h9l59jsanvr6-libXext-1.3.6-dev/lib/pkgconfig:/nix/store/hkkwx6m0g0hz87ckjzmyvzxj2gr9fnzn-xorgproto-2023.2/share/pkgconfig:/nix/store/1jkdxgxri7licnqphwzgm3gr0gp8fhwb-libXau-1.0.11-dev/lib/pkgconfig:/nix/store/13m2njz5m4wwrmf4n74zmfpcczi6kk8s-libXrender-0.9.11-dev/lib/pkgconfig:/nix/store/pirdcrf04q6qm2yq8w6w4gam0ax1vlnd-libX11-1.8.7-dev/lib/pkgconfig:/nix/store/hdzim0zvasrzw9mzh1c4qsqa9d3r5afq-libxcb-1.16-dev/lib/pkgconfig:/nix/store/lz55qmq1sdayqvs96lzds4zim6g2m6kz-glib-2.78.4-dev/lib/pkgconfig:/nix/store/7sg7d748qf9lc8cq0ywhs1y2qbxain3n-libffi-3.4.4-dev/lib/pkgconfig:/nix/store/jvv5cyrd3mq66lcyhijl57gdhf8fg5c6-gdk-pixbuf-2.42.10-dev/lib/pkgconfig:/nix/store/b1a4g7a7cxh1sl0y4rzbqyc4nmxhi874-libtiff-4.6.0-dev/lib/pkgconfig:/nix/store/1pnrplzkfyp3317ydjm37lizi8lhscg6-libdeflate-1.19/lib/pkgconfig:/nix/store/r2z54nm9xkrz7z6cv09a0g3j3am5chaj-libjpeg-turbo-3.0.2-dev/lib/pkgconfig:/nix/store/cyxbqa1v3bphanvzp43jpq0b5nij0jxn-xz-5.4.6-dev/lib/pkgconfig:/nix/store/vzn5grj1zg9q4c6gm4gra6wh5bj0438v-graphene-1.10.8-dev/lib/pkgconfig:/nix/store/03rhvhl5a3g9vry2njzmva7h27vimvcx-pango-1.51.0-dev/lib/pkgconfig:/nix/store/wbnc9vygplxrysw19r0hw9i0d6g7ryac-harfbuzz-8.3.0-dev/lib/pkgconfig:/nix/store/yza9lrc5v8g914wfz8abggqbqzn7lcal-graphite2-1.3.14-dev/lib/pkgconfig:/nix/store/l52jky64pp4ghhdncc3f915dhyir0gxs-libXft-2.3.8-dev/lib/pkgconfig:/nix/store/13ay8131gj00mjz9vmg37ypqyb5pppms-wayland-1.22.0-dev/lib/pkgconfig:/nix/store/yy6nwcb3k3bx1rv77n4qjgj7gdmnlk5b-wayland-1.22.0-bin/lib/pkgconfig:/nix/store/8gdcfdqnj9rfbk2833vpbqbj6wc0vpvr-gsettings-desktop-schemas-45.0/share/pkgconfig
```
Run as an exported value `PKG_CONFIG_PATH=... cargo run --example menu` and it will also build.

## Rust
Once you've run `nix-shell` to create your dev environment you can run `code` from the shell and it 
will inherit the dev shell setup.

### Openssl Dependencies
Use dev shell: `nix-shell -p pkg-config openssl`

### GTK4 Dependencies
Use dev shell: `nix-shell -p pkg-config openssl gtk4 libadwaita`

