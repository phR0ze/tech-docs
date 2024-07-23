# GTK
Note I'm focusing on GTK4

### Quick links
* [Overview](#overview)
* [Relm4](#relm4)

## Overview
`gtk-rs` project contains the base Rust bindings for GTK and its supporting libaries. Relm4 builds on 
top of `gtk-rs` to make the bindings more Rust idiomatic.

**Resources**
* [gtk4-rs book](https://gtk-rs.org/gtk4-rs/stable/latest/book/)

**Components**
* ***GTK*** - is the primary library used to construct user interfaces
* ***GSK*** - is an intermediate layer which provides a rendering API for Cairo, OpenGL or Vulkan
* ***GDK*** - is an intermediate layer which isolates GTK from the details of the windowing system.
* ***Pango*** - is the core text and font handling library used in GTK
* ***GdkPixbuf*** - is a library for image loading and manipulation
* ***Cairo*** - is a 2D graphics library with support for multiple output devices
* ***Glib*** - provides the core application building blocks for libraries and apps in C
* ***GObject*** - provides the object system used by GTK
* ***GIO*** - provides a portable, modern and easy-to-use file system abstration API for accessing local 
        and remote files; a set of abstractions over the DBus IPC, portable Networking and utilities.

### Install Dependencies in Nix
Use a dev shell to include system dependencies: `nix-shell -p pkg-config openssl gtk4 libadwaita`. 
The dev shell will read the nix development package paths and export them in a way that `pkg-config` 
will find them. This means that any build tools running in that shell will find the developement 
dependencies. Alternatively dumping out the env variables in the shell I found you can recreate the 
effect by grabbing the value of
```
PKG_CONFIG_PATH_FOR_TARGET=/nix/store/z5ghkxklksfayrri517mnp67vfj03im5-openssl-3.0.13-dev/lib/pkgconfig:/nix/store/0nb1qcnj6ha6vdzrkfkwbriw69n4nc3q-gtk4-4.12.5-dev/lib/pkgconfig:/nix/store/p78m3rmr5g1xqyizlsbg39cgbvjbjz74-cairo-1.18.0-dev/lib/pkgconfig:/nix/store/cs16dfmzrrdf88ghkh44vxwd6j17wjzn-fontconfig-2.15.0-dev/lib/pkgconfig:/nix/store/0bkz0x8vsb92kcr3inkgs0ba2dqlla5a-freetype-2.13.2-dev/lib/pkgconfig:/nix/store/hwlm6akqm83lapl79y8r9r2vw741sppd-zlib-1.3.1-dev/lib/pkgconfig:/nix/store/kh542xlg6mgb14xwz22mvqpm8810aidp-bzip2-1.0.8-dev/lib/pkgconfig:/nix/store/9kk0fm2s7c3269ph62av2l8hf95356ab-brotli-1.1.0-dev/lib/pkgconfig:/nix/store/831hplq82ppf633fv1qchlrsihhkryz4-libpng-apng-1.6.40-dev/lib/pkgconfig:/nix/store/mk3hpjywv0hkkn2bwplcb9lwvzsv2j0i-pixman-0.43.2/lib/pkgconfig:/nix/store/drz4r0b20cl5r4lsw600h9l59jsanvr6-libXext-1.3.6-dev/lib/pkgconfig:/nix/store/hkkwx6m0g0hz87ckjzmyvzxj2gr9fnzn-xorgproto-2023.2/share/pkgconfig:/nix/store/1jkdxgxri7licnqphwzgm3gr0gp8fhwb-libXau-1.0.11-dev/lib/pkgconfig:/nix/store/13m2njz5m4wwrmf4n74zmfpcczi6kk8s-libXrender-0.9.11-dev/lib/pkgconfig:/nix/store/pirdcrf04q6qm2yq8w6w4gam0ax1vlnd-libX11-1.8.7-dev/lib/pkgconfig:/nix/store/hdzim0zvasrzw9mzh1c4qsqa9d3r5afq-libxcb-1.16-dev/lib/pkgconfig:/nix/store/lz55qmq1sdayqvs96lzds4zim6g2m6kz-glib-2.78.4-dev/lib/pkgconfig:/nix/store/7sg7d748qf9lc8cq0ywhs1y2qbxain3n-libffi-3.4.4-dev/lib/pkgconfig:/nix/store/jvv5cyrd3mq66lcyhijl57gdhf8fg5c6-gdk-pixbuf-2.42.10-dev/lib/pkgconfig:/nix/store/b1a4g7a7cxh1sl0y4rzbqyc4nmxhi874-libtiff-4.6.0-dev/lib/pkgconfig:/nix/store/1pnrplzkfyp3317ydjm37lizi8lhscg6-libdeflate-1.19/lib/pkgconfig:/nix/store/r2z54nm9xkrz7z6cv09a0g3j3am5chaj-libjpeg-turbo-3.0.2-dev/lib/pkgconfig:/nix/store/cyxbqa1v3bphanvzp43jpq0b5nij0jxn-xz-5.4.6-dev/lib/pkgconfig:/nix/store/vzn5grj1zg9q4c6gm4gra6wh5bj0438v-graphene-1.10.8-dev/lib/pkgconfig:/nix/store/03rhvhl5a3g9vry2njzmva7h27vimvcx-pango-1.51.0-dev/lib/pkgconfig:/nix/store/wbnc9vygplxrysw19r0hw9i0d6g7ryac-harfbuzz-8.3.0-dev/lib/pkgconfig:/nix/store/yza9lrc5v8g914wfz8abggqbqzn7lcal-graphite2-1.3.14-dev/lib/pkgconfig:/nix/store/l52jky64pp4ghhdncc3f915dhyir0gxs-libXft-2.3.8-dev/lib/pkgconfig:/nix/store/13ay8131gj00mjz9vmg37ypqyb5pppms-wayland-1.22.0-dev/lib/pkgconfig:/nix/store/yy6nwcb3k3bx1rv77n4qjgj7gdmnlk5b-wayland-1.22.0-bin/lib/pkgconfig:/nix/store/8gdcfdqnj9rfbk2833vpbqbj6wc0vpvr-gsettings-desktop-schemas-45.0/share/pkgconfig
```
Run as an exported value `PKG_CONFIG_PATH=... cargo run --example menu` and it will also build.

You can run `code` from the shell and it will inherit the dev shell setup.

### Running examples
```
$ cd examples
$ cargo run --features="v4_6" --bin about_dialog
```

## Input and event Handling
**References**
* [GTK docs on input and event handling](https://docs.gtk.org/gtk4/input-handling.html)
* [GTK Event Controller](https://docs.gtk.org/gtk4/class.EventController.html)

GTK uses logical devices, i.e. pointer/keyboard pair also called a `seat`, for input from the user. 
When a user interactes with a logical device GTK receives events from the windowing system. GDK 
translates these raw events into `GdkEvents`. For key events the top level window is the first to 
trigger which passes the event to `Event Controllers` that are attached to the window. 

* GDK is the layer to use for all interactions with the windowing system i.e. keyboard events.
* GDK keys can be converted to and from names with `gdk_keyval_name()` and `gdk_keyval_from_name()`.

### Event Controllers
Event controllers are standalone objects that can perform specific actions upon received GdkEvents. 
The `EventControllerKey` is an event controller the provides access to key events.

### Key bindings
Key bindings are a form of shortcuts that allow you to 

## Font Attributes
GTK has a few ways to influence the look and feel of your fonts. Pango attributes is the first using 
a simple markup language with the `<span>` tag to allow configuring, font family, size, weight, style 
etc...

Example:
```rust
let size = "50";
let label = gtk::Label::default();
label.set_markup(format!("<span foreground=\"white\" font=\"{}\">test label</span>", size).as_str());
```

## Relm4
Relm4 claimes to make developing beautifaul cross-platform applications idiomatic, simple, fast and 
enables you to become productive in just a few hours. We'll see:

* [Relm4](https://relm4.org/)
* [Relm4 book](https://relm4.org/book/stable/)

### Running examples
```
$ cargo run --features="libadwaita gnome_43" --example toast
```

