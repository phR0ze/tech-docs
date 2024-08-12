Rust GUI
====================================================================================================
<img align="left" width="48" height="48" src="../../../art/logo_256x256.png">
Documenting my learning experience with Rust GUIs. Specifically I'm looking for cross-platform i.e. 
Android, WASM and Linux support using Arch Linux as my devlopment environment.
<br><br>

### Quick links
* [.. up dir](..)
* [Overview](#overview)
  * [Requirements](#requirements)
  * [General UI Design](#general-ui-design)
* [Install prerequisites](#install-prerequisites)
  * [Install Rust](#install-rust)
  * [Install Rust Android targets](#install-rust-android-targets)
* [Tauri](#tauri)
  * [Install Tauri Dependencies](#install-tauri-dependencies)
* [Dioxus](#dioxus)
* [Freya](#freya)
* [Tailwind CSS](#tailwind-css)
* [BULMA CSS](#bulma-css)
* [DaisyUI](#daisyui)

# Overview

## Requirements
* Low power consumption
* Media processing capabiliies
* As much pure Rust as possible
* Cross-platform support including Android, Linux and WASM

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
rather than baked in ones. 

### Install Tauri Dependencies
* [Tauri docs](https://next--tauri.netlify.app/next/guides/getting-started/prerequisites/linux)

1. Install Arch system libs
   ```
   sudo pacman -Syu
   sudo pacman -S --needed \
       webkit2gtk-4.1 \
       base-devel \
       curl \
       wget \
       openssl \
       appmenu-gtk-module \
       gtk3 \
       libappindicator-gtk3 \
       librsvg \
       libvips
   ```
2. Install Android rust targets
   ```bash
   $ rustup target add aarch64-linux-android
   $ rustup target add armv7-linux-androideabi
   $ rustup target add i686-linux-android
   $ rustup target add x86_64-linux-android
   ```
3. Install Android Studio from the [official website](https://developer.android.com/studio)
   1. Install deps: `libpulse, nss, libxcomposite, libxcursor`
   2. Build package: `yay -Ga android-studio; cd android-studio; makepkg -s`
   3. Install package: `sudo pacman -U sudo pacman -U android-studio-2022.1.1.21-1-x86_64.pkg.tar.zst`
   4. Set JDK path: `export JAVA_HOME=/opt/android-studio/jbr`
4. Install Android SDK
   1. Launch IDE: `android-studio`
   2. Follow prompts to install Android SDK to `$HOME/Android/Sdk"`
   3. Load and project and then navigate to `Tools >SDK Manager`
   4. Select the `SDK Tools` tab then `NDK (Side by side)` and `Android SDK Command-line Tools`
   5. Figure out your correct NDK path with `ll $HOME/Android/Sdk/ndk`
   6. Set the Android SDK path: `export ANDROID_HOME="$HOME/Android/Sdk"`
   7. Set the Android NDK path: `export NDK_HOME="$ANDROID_HOME/ndk/25.2.9519653"`
   
5. Install Android NDK

## Dioxus
Dioxus is a React inspired portable, performant and ergonomic framework for building cross-platform 
user interfaces in Rust. It is built on top of Tauri which provides the cross-platform aspects. Tauri 
could be used directly but then you loose Dioxus's convenient works out of the box paradigm. Dioxus 
layers on top making it just work and adding the React capablities.

see [Dioxus in the development/ui/cross\_platform section](../../../ui/cross_platform/dioxus/README.md)

## Freya

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

<!-- 
vim: ts=2:sw=2:sts=2
-->
