# Zed
Built by the authors of Atom and Electron, Zed claims to be a next generation editor built in Rust 
for performance and modern use cases. They aim to focus on speed, AI and collaboration.

**WARNING** ***No debugger support yet***

### Quick links
* [Overview](#overview)
  * [Features](#features)
* [Installation](#installation)
  * [NixOS](#nixos)
* [Configuration](#configuration)
  * [NixOS](#nixos)
* [Debugging](#debugging)

* [Troubleshooting](#troubleshooting)

## Overview

### Features
* Multibuffer editing
* Extensible via extension ecosystem
* First-class modal editing via Vim bindings

### Basic operations
* Open configuration page `Ctrl+,`
* Control buffer font size `Ctrl+=` or `Ctrl+-`

## Installation

### NixOS
**References**
* [Zed Docs - Linux](https://zed.dev/docs/linux)

**Install from nixpkgs**
1. You need to use latest unstable, for me that means adding an overlay exception
   ```
   zed-editor = pkgs-unstable.zed-editor;
   ```

2. Add the install configuration
   ```
   environment.systemPackages = [
     pkgs.zed-editor
   ];
   ```

## Configuration

### Out of the box
First run will splash up some simple configuration options

1. Enable VIM mode
2. Enable and login to Copilot
3. Keep default vscode keybindings

### Settings
Settings are stored at `~/.config/zed/settings.json`

```json
{
  "base_keymap": "VSCode",
  "telemetry": {
    "diagnostics": true,
    "metrics": true
  },
  "vim_mode": true,
  "ui_font_size": 16,
  "buffer_font_size": 16,
  "theme": {
    "mode": "system",
    "light": "Solarized Dark",
    "dark": "One Dark"
  }
}
```

### Keybindings
* Close buffer `Ctrl+w`

## Debugging


## Troubleshooting

### Save All
Pressing `Ctrl+Shift+s` triggers and error with message `The name org.freedesktop.portal.Desktop` was 
not provided by any .service files
