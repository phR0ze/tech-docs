# Dioxus <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Dioxus is a React inspired portable, performant and ergonomic framework for building cross-platform 
user interfaces in Rust. It is built on top of Tauri which provides the cross-platform aspects. Tauri 
could be used directly but then you loose Dioxus's convenient works out of the box paradigm. Dioxus 
layers on top making it just work and adding the React capablities. In `v0.6` Dioxus now supports 
Mobile out of the box finally making it a viable option as compared to Flutter.

### Quick links
- [.. up dir](../README.md)
* [Overview](#overview)
* [Getting started](#getting-started)
  * [Build the demo web app](#build-the-demo-web-app)
  * [Get examples running](#get-examples-running)
  * [Install CLI on NixOS](#install-on-nixos)
* [Dioxus CLI](#dioxus-cli)
  * [CLI config](#cli-config)
* [UI Styling](#ui-styling)
  * [Tailwind CSS](#tailwind-css)
  * [BULMA CSS](#bulma-css)
  * [DaisyUI](#daisyui)
* [Additional Research](#additional-research)
  * [WASM fundamentals](#wasm-fundamentals)
  * [Blitz](#blitz)

### Linked pages
* [Tailwind CSS](tailwind_css/README.md)

## Overview

**References**
* [Dioxus 0.6 guide](https://dioxuslabs.com/learn/0.6/guide/)
* [NixOS Wiki](https://wiki.nixos.org/wiki/Dioxus)
* [Awesome Dioxus](https://github.com/DioxusLabs/awesome-dioxus)
* [Fetch images example](https://github.com/DioxusLabs/dioxus/blob/master/examples/suspense.rs)
* [Desktop Window Builder](https://docs.rs/dioxus-desktop/latest/dioxus_desktop/struct.WindowBuilder.html#method.with_menu)
* [Example of WASM and Desktop](https://github.com/LyonSyonII/dioxus-tailwindcss)
* [Window Decorations](https://github.com/DioxusLabs/dioxus/blob/master/examples/overlay.rs)
* [Dioxus manual future restart](https://github.com/DioxusLabs/dioxus/issues/149)
* [Dioxus hooks helpers](https://github.com/oovm/dioxus-hooks)
* [React hook recipes](https://usehooks.com/)
* [Dioxus starter](https://github.com/mrxiaozhuox/dioxus-starter)
* [Real World example](https://github.com/dxps/fullstack-rust-axum-dioxus-rwa)
* [Material Icons browser](https://mui.com/material-ui/material-icons/)

## Getting Started

**References**
* [Dioxus 0.6 guide](https://dioxuslabs.com/learn/0.6/guide/new_app/)

### Build the demo web app
This will generate the Dioxus welcome app with some links and icons and the ability to build for 
`Desktop`, `Web` and `Mobile`. Note the `Web` option is nice to start from as you don't need any 
additional dependencies.

1. Open a shell and run: `dx new`
2. Which sub-template should be expanded? `Bare-Bones`
3. Do you want to use Dioxus Fullstack? `false`
4. Do you want to use Dioxus Router? `false`
5. Do you want to use Tailwind CSS? `false`
6. Which platform do you want DX to serve by default? `Web`

### Configure Vscode for Dioxus
1. Launch vscode and open your project
2. Install the `Dioxus` extension from `Dioxus Labs`
3. Install the `Android iOS Emulator` from `Diemas Michiels`

## Dioxus framework

### Components
Components are `functions` that take in some `Properties` and render an `Element`. All components 
take an object that outlines which parameters the component can accept. All `Props` structs in Dioxus 
need to derive the `Properties` trait which requires both `clone` and `PartialEq`.

## Get examples running
Dioxus applications require native dependencies be present for runtime. This takes a few extra steps 
for NixOS. Luckily Dioxus is NixOS friendly and provide a dev shell for us.

**References**
* [Dioxus Dev shell](https://github.com/DioxusLabs/dioxus/discussions/3710)
* `nix develop nixpkgs#dioxus-cli`

1. Check out the latest stable tag
   ```bash
   $ cd ~/Projects/temp/dioxus
   $ git checkout -b v0.6.3 v0.6.3
   ```
2. Launch the dev shell
   ```bash
   $ nix develop
   ```
3. Build and install dioxus-cli corresponding to the tag e.g. `v0.6.3`
   ```bash
   $ cargo install --path packages/cli
   ```
4. From inside the dev shell launch your example of choice
   ```bash
   $ cargo run --example todomvc
   $ cargo run --example dog_app --features="http"
   ```
5. Alternatively for the example projects
   ```bash
   $ cd example-projects/fullstack-hackernews
   $ cargo run --features=web
   ```
### -------------------------------------------
1. Create a dev shell for Dioxus
   ```flake.nix
   {
     pkgs ? import <nixpkgs> { },
   }:
   let
     overrides = (builtins.fromTOML (builtins.readFile ./rust-toolchain.toml));
   in
   pkgs.callPackage (
     {
       atkmm,
       cairo,
       gcc,
       gdk-pixbuf,
       glib,
       gtk3,
       mkShell,
       openssl,
       pango,
       pkg-config,
       rustup,
       rustPlatform,
       stdenv,
       webkitgtk_4_1,  # for javascriptcoregtk-rs-sys
       xdotool,        # for libxdo
     }:
     mkShell {
       strictDeps = true;
       nativeBuildInputs = [
         gcc
         openssl
         pkg-config
         rustup
         rustPlatform.bindgenHook
       ];
       # libraries here
       buildInputs = [
         atkmm
         cairo
         gdk-pixbuf
         glib
         gtk3
         pango
         webkitgtk_4_1
         xdotool
       ];
       PKG_CONFIG_PATH = "${pkgs.openssl.dev}/lib/pkgconfig";
       RUSTC_VERSION = overrides.toolchain.channel;
       shellHook = ''
         export PATH="''${CARGO_HOME:-~/.cargo}/bin":"$PATH"
         export PATH="''${RUSTUP_HOME:-~/.rustup}/toolchains/$RUSTC_VERSION-${stdenv.hostPlatform.rust.rustcTarget}/bin":"$PATH"
       '';
     }
   ) { }
   ```


### Install CLI on NixOS
To avoid version conflicts between Dioxus CLI and your application you'll need to match the versions 
of `wasm-bindgen` being used. This can be done by pinning your application dependency and by updating 
teh Dioxus CLI's lock file being used.

1. Match your `wasm-bindgen` versions
   1. Pin the `wasm-bindgen` version in your app
      ```Cargo.toml
      [dependencies]
      wasm-bindgen = "=0.2.97"
      ```
   2. Then update
      ```bash
      $ cargo update
      ```
   3. Alternatively you can rebuild Dioxus CLI with version override
      ```nix
      dioxus-cli = pkgs.dioxus-cli.overrideAttrs (_: {
        postPatch = ''
          rm Cargo.lock
          cp ${./Dioxus.lock} Cargo.lock
        '';
      
        cargoDeps = pkgs.rustPlatform.importCargoLock {
          lockFile = ./Dioxus.lock;
        };
      });
      ```
2. Install the Dioxus CLI with 
   ```nix
   ```

## Validate that Dioxus is working

### NixOS OpenGL problem
```
[linux] MESA-LOADER: failed to open dri: /run/opengl-driver/lib/gbm/dri_gbm.so: cannot open shared object file: No such file or directory (search paths /run/opengl-driver/lib/gbm, suffix _gbm)
```

**References**
* [libGL not working](https://github.com/NixOS/nixpkgs/issues/9415)
* [OpenGL problem](https://discourse.nixos.org/t/design-discussion-about-nixgl-opengl-cuda-opencl-wrapper-for-nix/2453)

### Clone and build Tailwind example

1. Clone the dioxus examples
   ```bash
   $ git clone https://github.com/DioxusLabs/dioxus.git
   ```
2. Build and run
   ```bash
   $ cd dioxus/examples/tailwind
   $ dx serve
   ```

## Modifications to default App

### Conditional platform code
Conditional platform code is used for CSS imports. WASM will pick up the CSS from the index.html 
configuration and local files while the desktop version will need to import it via the `launch_cfg` 
configuration.

```rust
#![allow(non_snake_case)]

use dioxus::prelude::*;

fn main() {
   #[cfg(target_family = "wasm")]
   dioxus_web::launch(App);

   #[cfg(any(windows, unix))]
   dioxus_desktop::launch_cfg(Root, dioxus_desktop::Config::new());
}

fn App(cx: Scope) -> Element {
   cx.render(rsx! {
       div {
           "Hello, world!"
       }
   })
}
```

### Set the window properties
You can set the window's name, size and other window properties using the 
`dioxus_desktop::WindowBuilder` builder object.

```rust
dioxus_desktop::launch_cfg(
  App,
  dioxus_desktop::Config::new()
    .with_window(
        dioxus_desktop::WindowBuilder::new()
          .with_title("diper")
          .with_decorations(false)
          .with_inner_size(dioxus_desktop::LogicalSize::new(300.0, 300.0)),
    )
);
```

## Build and run a Dioxus WASM app
1. Build and run
   ```
   dioxus serve --hot-reload
   ```
2. Browse to `http://localhost:8080/`

## Dioxus global state - fermi

### State rendering events
The `fermi` project provide a convenient way to access global state by object type. However in order 
to be efficient you need to be careful in how you construct and use this global state as Dioxus will 
be triggering the re-render of components based on their use of state that has also changed. This 
means that it is problematic to use a centralized state object for all state as any part of the 
system that uses the same state object will automatically trigger component updates even when they 
shouldn't be triggered. As a result the best practice for this is to use granular objects for state. 
Potentially a state object per type use-case. In this way we can avoid un-related component 
re-renders.

## Dioxus CLI
Provides support for

### CLI config
The `dioxus` cli binary we installed earlier in the process provides the ability to build our project 
and stage it to be served by a production server like `axum` to do this though we need to create the 
`Dioxus.toml` configuration file and specify where the static assets should end up. By default this 
is the `dist` directory inside your frontend project.

## UI Styling

* [Tailwind CSS](tailwind_css/README.md)

### Daisy UI
Tailwind.css provides the tools to build beautiful UIs with infinite customization. DaisyUI is a 
Tailwind plugin that provides a number of pre-created components along the lines of Bulma CSS that 
use Tailwind to allow you to get up and running faster and with fewer class names.

**References**
* [DaisyUI Components](https://daisyui.com/components/)

### BULMA CSS
Bulma is a free, open source framework that provides ready-to-use frontend components that you can 
easily combine to build responsive web interfaces.

* 100% responsive mobile first
* Just import what you need
* Modern, build with Flexbox

**References**
* [Bulma github](https://github.com/jgthms/bulma)
* [Bulma docs](https://bulma.io/documentation/)
* [Dioxus Bulma github](https://github.com/mrxiaozhuox/dioxus-bulma)

## Additional Research

**References**
* [Full featured mail client](https://github.com/jkelleyrtp/blazemail)
* [ER Mule Copier](https://github.com/pubnoconst/er_mule_copier)
* [CSS Primer impl](https://github.com/purton-tech/cloak/tree/main/crates/primer-rsx)
* [TwitVault](https://github.com/terhechte/twitvault)
* [Pomodoro timer](https://github.com/kualta/pomo)
* [Emoji Browser from Dioxus Class](https://github.com/edger-dev/dioxus-class)
* [Dioxus Bulma](https://github.com/mrxiaozhuox/dioxus-bulma)
* [Uplink - P2P chat](https://github.com/Satellite-im/Uplink)
* [Blazemail](https://github.com/jkelleyrtp/blazemail)
* [Rummy Nights](https://github.com/arqalite/rummy-nights)
* [SpideyClick](http://demo.spideyclick.net/)
* [Karaty](https://github.com/mrxiaozhuox/karaty)
* [Hangman online](https://github.com/lennartkloock/hangman-online)
* [Freya](https://freyaui.dev/)
* [Blitz Dioxus Native](https://www.youtube.com/watch?v=rMaF7cl6ooY)

### WASM fundamentals
* `wasm-bindgen` - create and cli for generating bindings to WASM from JavaScript. It allows you to 
  call to and from JavaScript. Meaning it is an interoperability layer for `Rust <==> JavaScript`. 
  Thus Rust can call into the Browser functions just like JavaScript does.
* `web-sys` - generates bindings for all the Web APIs and DOM APIs providing Rust typed access to all 
  functions.
* hydrate islands of interactivity not everything at once to get better performance. this is a 
  partial load and speeds up by 75%.

### Blitz
Blitz is a new WGPU based renderer to render natively.
* Similar to Electron (130mb), Tauri, Sciter (34mb), Ultralight (22mb), Litehtml. 
* Blitz is only 12mb, it is most similar to Flutter
* Drops out Javascript support for efficiency
* Built on Servo, but more modular
* Blitz DOM: built on Stylo, Taffy, Parley, AccessKit, Image, USVG
* Blitz Vello vs: WebRender or Skia
* Vello is a lib that uses WGPU

