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
* [egui](#egui)
* [Emerging / Watch List](#emerging--watch-list)
  * [Xilem](#xilem)
  * [GPUI](#gpui)
* [Dioxus vs Flutter](#dioxus-vs-flutter)
  * [Real-world adoption](#real-world-adoption)
* [Recommendation](#recommendation)
* [Media & Gaming Angle](#media--gaming-angle)
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

## egui
[egui](https://github.com/emilk/egui) (via `eframe`) is one of the oldest and most widely used pure-Rust
immediate-mode GUI toolkits — mature and stable on Linux desktop and Web/WASM, which are `eframe`'s
original and strongest target platforms.

Android is the weak point, and a weaker one than Dioxus's: native Android support in `eframe` landed
via a comparatively recent PR using `winit`'s `AndroidApp`
([emilk/egui#5318](https://github.com/emilk/egui/pull/5318)), and is explicitly still maturing.
Multiple users report the Android dev-environment setup itself is rough unless you're already a
full-time Android developer with a pre-configured build host
([egui#5848](https://github.com/emilk/egui/issues/5848),
[egui#2066](https://github.com/emilk/egui/issues/2066)). Unlike Dioxus's `dx serve --platform android`,
egui has no first-party CLI for mobile packaging — expect something closer to the hand-rolled
`cargo-quad-apk` experience already documented on the [Macroquad](../gaming/macroquad/README.md) page.

On power consumption: the common assumption that immediate-mode GUIs are inherently a battery drain
(a claim this repo already makes in the gaming/Macroquad context) doesn't fully hold for egui — it
supports a reactive/on-demand repaint mode that only redraws on input events or explicit invalidation
rather than every frame. Only its continuous/always-redraw mode costs meaningfully more CPU; that's a
mode choice, not an inherent property of the toolkit.

Media processing is fine via the Rust `image` crate plus `egui_extras`'s image-loading helpers — no
worse than Dioxus or Freya here.

**Takeaway**: egui is a legitimate, currently more battle-tested than Dioxus contender for **Linux +
Web**, but its Android story is rougher and less turnkey than Dioxus 0.7's (which at least ships
official first-party mobile tooling, bugs and all). For the full three-platform requirement it sits at
the same fallback tier as Tauri/Freya rather than displacing Dioxus as the primary recommendation.

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

## Dioxus vs Flutter
Flutter isn't a Rust framework, but it's the other realistic answer to "one codebase, Linux + Android
+ Web" and it's worth a direct comparison against Dioxus rather than dismissing it on language grounds
alone. See [Flutter](../../../ui/flutter/README.md) for the detailed personal reference.

| Dimension | Dioxus 0.7+ (native renderer) | Flutter |
|---|---|---|
| Architecture/Rendering | Pure Rust; Blitz (HTML/CSS layout) + Vello (WGPU) | Dart UI logic; Skia/Impeller rendering engine in C/C++ |
| Linux desktop | Native, GPU-accelerated, self-contained <6MB builds | Mature overall, but no supported way to get the Wayland `xdg_toplevel` handle, blocking native seamless window-drag ([flutter/flutter#187837](https://github.com/flutter/flutter/issues/187837)); otherwise efficient (~50MB disk/~25MB memory for a simple app) |
| Android | Experimental: app crash on portrait/landscape rotation ([#3470](https://github.com/DioxusLabs/dioxus/issues/3470)), `dx build --android --release` emits stale Java 8 Gradle config incompatible with modern AGP/Gradle ([#5251](https://github.com/DioxusLabs/dioxus/issues/5251)), out-of-date docs, assets outside `.rs` sources reportedly not read, no native widgets/animations (CSS-based only) | Mature, primary target platform since 2018, huge tooling investment |
| Web/WASM | Small hello-world (~100KB-2.36MB) but real multi-component apps can balloon toward 100MB+ without careful tree-shaking (whole app ships as one WASM blob today) | CanvasKit/skwasm renderer, heavier per-load but 2–3x faster than JS builds, mature size/perf tooling |
| Media processing | Rust `image`/`wgpu` ecosystem directly, no FFI | Native Dart/platform channels, or Rust via `flutter_rust_bridge` |
| Ecosystem & backing | Small, venture-backed (Dioxus Labs, YC); rendering stack still stabilizing through 2026 | Google-backed since 2015/GA 2018; huge `pub.dev` package ecosystem, large job market |
| Dev experience | Hot-patching across all platforms; single language (Rust) throughout | Hot reload; requires Dart, plus Rust+FFI if a hybrid core is used |

### Real-world adoption
The user specifically asked what popular apps prove one way or the other — the picture is more nuanced
than a single example suggests:

* **RustDesk** is the concrete case worth knowing precisely: it did **not** migrate from a Rust GUI
  toolkit to Flutter. It migrated from **Sciter** (a lightweight, closed-source webview-like rendering
  engine) to Flutter, keeping its **Rust backend unchanged**
  ([migration wiki](https://github.com/rustdesk/rustdesk/wiki/RustDesk-Desktop-Flutter-Migration),
  [migration writeup](https://dev.to/rustdesk/rustdesk-is-migrated-from-sciter-to-flutter-279m)).
  Flutter now covers iOS, Android, Windows, Linux, Mac and Web for RustDesk in production at real
  scale — with one carve-out: Flutter doesn't support 32-bit Windows, so Sciter is kept alive just for
  that target. This is strong, large-scale, real-world validation of the **Rust core + Flutter UI**
  hybrid pattern specifically, not evidence against pure-Rust UI toolkits (RustDesk never shipped on
  one to begin with).
* **Counter-signal on the hybrid pattern**: at least one solo developer went the *opposite* direction —
  from Flutter+Rust (via `flutter_rust_bridge`) back to pure Rust+egui — citing the FFI-bridge
  generated code as unreadable/brittle ("I don't understand this generated code, it's not readable")
  and the two-language maintenance burden as not worth it for a smaller project
  ([writeup](https://jdiaz97.github.io/greenblog/posts/flutter_to_egui/),
  [HN discussion](https://news.ycombinator.com/item?id=44361288)). Worth weighing before assuming the
  hybrid path is free.
* **Dioxus production users**: Dioxus Labs' own marketing states Huawei is shipping production apps
  with Dioxus, and Airbus/the European Space Agency are building a collision-avoidance system with it.
  These are vendor-reported, not independently verified case studies — no public write-ups were found
  corroborating scale or which platform targets are actually in use. Treat as a signal of some
  enterprise interest, not proof of Android/Web production maturity.
* **Broader market trend**: searches for "companies switching from Flutter" mostly surface moves
  toward React Native, Kotlin Multiplatform, or fully native — not toward Rust GUI toolkits. There's no
  evidence of a wave of companies replacing Flutter with Dioxus/Tauri/etc.; where movement exists, it's
  individual developers on smaller projects (like the egui case above), not enterprises.

## Recommendation
Given the requirements above (pure Rust, low power, media processing, Linux + Android + WASM):

* **If Android is a near-term must-ship target** (or the Wayland window-chrome gap above isn't a
  blocker): **Flutter, optionally with `flutter_rust_bridge`** for Rust-side logic/media processing,
  is the lower-risk choice today. It's the only option here with a proven, large-scale, real-world
  cross-platform deployment (RustDesk). "As much pure Rust as possible" is still achievable at the
  logic/core layer via the bridge — but weigh the bridge-maintenance complaint from the egui
  counter-example above if this is a smaller/solo project.
* **If architectural purity (Rust through the UI) is the priority** and rough Android edges are
  tolerable — e.g. desktop+web ship first, Android later, or the plan includes upstreaming fixes —
  **Dioxus 0.7+ native renderer** remains the better long-term fit and the smaller footprint. A few
  enterprises are reportedly already betting on it, though unconfirmed at RustDesk's proven scale.
* **Fallback if neither native-renderer path fits a given app:**
  * Tauri 2.x — accept a webview/JS frontend in exchange for maturity and mobile-plugin breadth.
  * Freya — accept giving up the WASM/web target in exchange for a fully native, pure-Rust,
    Skia-rendered UI on desktop + Android.
  * egui — if Android is a later/lower-priority target, egui's Linux+Web maturity and simplicity make
    it a lower-risk pure-Rust option than waiting on Dioxus's native renderer to stabilize; just don't
    expect turnkey Android packaging.
* **Not yet viable:** Xilem, GPUI-mobile — keep watching, don't build production apps on them yet.

## Media & Gaming Angle
The `## Recommendation` above is tuned for general-purpose app UI (forms, navigation, CRUD-style
layouts). If the priority shifts to **displaying media/images well, with gaming as an optional
stretch goal**, the ranking changes — this is a re-weighting, not a reversal.

Dioxus, Flutter, Freya and Tauri are all fundamentally widget-tree/DOM UI frameworks (HTML/CSS-like
layout, or a Skia-retained widget tree). Basic image display (an `<img>`-equivalent) is fine on all of
them, but none has sprites, a render-texture pipeline, or a natural game loop — "optional gaming" on
top of them means bolting on custom `wgpu`/canvas code against the grain of the framework, not using a
supported path.

* **Primary pick for this angle: Macroquad.** Already cross-platform Linux/Android/WASM, purpose-built
  for 2D image/sprite manipulation and render textures, lightweight, with a large catalogue of working
  examples already recorded on the [Macroquad](../gaming/macroquad/README.md) page. It's the closest
  single framework to all four original requirements (pure Rust, low power, cross-platform, media
  processing) plus a natural fit for optional gaming.
* **Strong alternative: Flutter + [Flame](https://flame-engine.org/).** Inherits Flutter's mature
  image-handling ecosystem (`extended_image`, `photo_view`, `flutter_cache_manager` — already
  documented as personal favorites in [Flutter](../../../ui/flutter/README.md)) and adds a real,
  actively developed 2D game layer (game loop, sprites, collision detection) across
  Android/iOS/Web/desktop from one codebase. The pragmatic choice if "as much pure Rust as possible"
  can flex for a media/gaming-heavy app.
* **If gaming becomes central, not just optional**: see the "Gaming Ecosystem" survey (Bevy, Fyrox,
  GGEZ, Nannou) already on the [Macroquad](../gaming/macroquad/README.md) page rather than duplicating
  it here. Bevy specifically has mature WASM support but Android is "possible but not easy" and a low
  project priority — the same shape as Dioxus's and egui's Android stories.
* **egui** is viable for custom canvas drawing and simple 2D games via its `Painter`/texture APIs (see
  [egui](#egui) above), but lacks Macroquad's sprite/asset/render-texture conveniences — reasonable
  only as an add-on if the project is already committed to egui for its UI, not a from-scratch pick for
  this axis.

**Verdict**: for "media/image display first, gaming optional," Macroquad is the closest match to the
stated requirements; Flutter+Flame is the lower-risk, more mature alternative if pure Rust can flex.

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
