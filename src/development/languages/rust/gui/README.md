Rust GUI
====================================================================================================
<img align="left" width="48" height="48" src="../../../art/logo_256x256.png">
Documenting my learning experience with Rust GUIs. Specifically I'm looking for cross-platform i.e. 
Android, WASM and Linux support using NixOS as my development environment.
<br><br>

### Quick links
* [.. up dir](..)
* [Overview](#overview)
  * [Requirements](#requirements)
  * [Landscape shift in 2026](#landscape-shift-in-2026)
  * [General UI Design](#general-ui-design)
* [Install prerequisites](#install-prerequisites)
  * [Install Rust](#install-rust)
  * [Install Rust Android targets](#install-rust-android-targets)
* [Tauri](#tauri)
* [Dioxus](#dioxus)
* [Freya](#freya)
* [Emerging / Watch List](#emerging--watch-list)
  * [Xilem](#xilem)
  * [GPUI](#gpui)
* [Recommendation](#recommendation)
* [Tailwind CSS](#tailwind-css)
* [Daisy UI](#daisy-ui)
* [BULMA CSS](#bulma-css)

# Overview

## Requirements
* Low power consumption
* Media processing capabiliies
* As much pure Rust as possible
* Cross-platform support including Android, Linux and WASM

## Landscape shift in 2026
Since this research started, the field moved from "cross-platform Rust GUI" almost always meaning
"Rust backend + system webview" (Tauri, and Dioxus built on top of Tauri) towards native, GPU-rendered
UI written entirely in Rust. The biggest single change is **Dioxus 0.7**, which replaced its
webview-based renderer with a native `Blitz` (HTML/CSS layout) + `Vello` (WGPU vector rendering) stack
— no JS engine, no system webview dependency, self-contained macOS builds under 6MB, and hot-reloading
that works across desktop, web and mobile. That's a much closer match to the "as much pure Rust as
possible" requirement above than the old Tauri-webview architecture this page used to describe.

## General UI Design

**References**
* [UI Glossary](https://www.uxdesigninstitute.com/blog/ui-glossary/)
* [React Icons for lookup](https://react-icons.github.io/react-icons/search?q=arrow)

# Install pre-requisites

## Install Rust
see [README.md/#install-rust](README.md/#install-rust)

## Install Rust Android targets
```bash
$ rustup target add aarch64-linux-android
$ rustup target add armv7-linux-androideabi
$ rustup target add i686-linux-android
$ rustup target add x86_64-linux-android
```

## Tauri
Tauri is a lot like Electron only faster and smaller. While Electron uses the Chromium engine with 
Node.js bundled together producing fat 150MB+ binaries, Tauri is a Rust engine that uses the 
operating system's WebView libraries, making it faster and smaller as its uding dynamic libraries 
rather than baked in ones. As of Tauri 2.x (stable, currently v2.10.x) mobile targets (iOS, Android)
are production-viable alongside desktop, with hot-reload extended to mobile emulators/devices and
native API plugins (notifications, biometrics, NFC, clipboard, deep links). The trade-off relative to
this page's "as much pure Rust as possible" requirement remains: the UI itself is still HTML/CSS/JS
rendered in a system webview — only the backend is Rust. Good fit if you want to use web frontend
tooling; not the pure-Rust option.

For Nix dev-shell setup (GTK/webview deps, Android SDK/NDK), see
[Tauri/Webview Dependencies](../../../../system/nixos/build_source/README.md#tauriwebview-dependencies)
and
[Android SDK/NDK Dependencies](../../../../system/nixos/build_source/README.md#android-sdkndk-dependencies)
rather than the old Arch/pacman steps this page used to list — those version numbers (Android Studio
`2022.1.1.21`, NDK `25.2.9519653`) are stale and shouldn't be treated as current recommendations.

**References**
* [Tauri docs](https://v2.tauri.app/start/prerequisites/)

## Dioxus
Dioxus is a React-inspired portable, performant and ergonomic framework for building cross-platform 
user interfaces in Rust. As of Dioxus 0.7 it is no longer just a layer over Tauri's webview — it ships
a native renderer (`Blitz` + `Vello`, GPU-accelerated via WGPU) as an alternative to the webview-based
desktop/mobile renderer, meaning a single codebase can target Web (WASM), Desktop and Mobile
(iOS/Android) with either backend. This makes Dioxus the closest match on this page to all four stated
requirements (pure Rust, low power, cross-platform, media-capable via `image`/`wgpu` ecosystem crates).

see [Dioxus in the development/ui section](../../../ui/dioxus/README.md) for setup details and current
status of the native renderer.

## Freya
[Freya](https://freyaui.dev/) is a cross-platform, non-web GUI library for Rust rendered with
[Skia](https://skia.org/) (via `skia-safe`) rather than HTML/CSS. It originally used Dioxus as its
reactive/component engine but has since been rewritten standalone for better type safety and to
tailor the engine to Freya's own needs. It renders directly to native windows with GPU acceleration
(OpenGL/Vulkan/Metal) on Windows, macOS and Linux, and supports Android via `freya-android` (OpenGL).
WASM/web-target maturity is unclear as of this writing — treat that as an open question to verify
before committing to Freya if a web target is required.

## Emerging / Watch List
Frameworks that are technically interesting for this requirement set but not yet production-ready —
worth tracking, not building on top of yet.

### Xilem
[Xilem](https://github.com/linebender/xilem) is Linebender's (the Druid successor) reactive Rust UI
framework, built as a layer on top of the `Masonry` widget toolkit, with `Vello` for rendering. It has
a web backend in addition to the native Masonry backend. Status is explicitly pre-alpha/alpha —
improving rapidly but with significant known issues. No clear Android story yet.

### GPUI
[GPUI](https://github.com/zed-industries/zed/tree/main/crates/gpui) is the GPU-accelerated, native
Rust UI framework built for and open-sourced alongside the [Zed](https://zed.dev/) editor. Officially
supported platforms are macOS, Linux and Windows. Mobile (iOS/Android) support exists only via an
early-stage third-party crate, [`gpui-mobile`](https://github.com/itsbalamurali/gpui-mobile), which
itself depends on upstream Zed crates (`gpui`, `gpui_wgpu`) not yet published to crates.io. Not viable
for this requirement set yet.

## Recommendation
Given the requirements above (pure Rust, low power, media processing, Linux + Android + WASM):

* **Primary path: Dioxus 0.7+ using the native renderer.** It's the only framework here with a
  simultaneous, production-track story for native GPU rendering, mobile, and WASM from one Rust
  codebase — the closest match to all four requirements today.
* **Fallback if the native renderer proves too immature for a given app:**
  * Tauri 2.x — accept a webview/JS frontend in exchange for maturity and mobile-plugin breadth.
  * Freya — accept giving up the WASM/web target in exchange for a fully native, pure-Rust,
    Skia-rendered UI on desktop + Android.
* **Not yet viable:** Xilem, GPUI-mobile — keep watching, don't build production apps on them yet.

## Tailwind CSS
A utility-first CSS framework packed with utility type classes that can be composed to build any 
design with infinite flexibility. Nothing is pre-styled; not even headings or links. You have to 
create everything from scratch, giving you the opportunity to create something unique. Typically a 
designer will create sets of semantic classes that group the appropriate utiltiy classes for 
component types e.g. `btn`, `btn-primary` etc... and use those everywhere and don't use the utility 
classes directly. `Tailwind UI` though is a higher level set of pre-styled components such as `hero`, 
`sections`, `CTA` etc... more like Bulma or Bootstrap but are based on the utility classes of 
Tailwind CSS. Notably though Tailwind UI is not free. However there are a host of others like 
`daisyUI` or `Flowbite` that are built on top of Tailwind CSS providing pre-styled components similar 
to Tailwind UI for free.

## Daisy UI
Tailwind.css provides the tools to build beautiful UIs with infinite customization. DaisyUI is a 
Tailwind plugin that provides a number of pre-created components along the lines of Bulma CSS that 
use Tailwind to allow you to get up and running faster and use fewer class names.

## BULMA CSS
Bulma is a free, open source framework that provides ready-to-use frontend components that you can 
easily combine to build responsive web interfaces.
