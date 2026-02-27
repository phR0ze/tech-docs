# Zed <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Built by the authors of Atom and Electron, Zed claims to be a next generation editor built in Rust 
for performance and modern use cases. They aim to focus on speed, `AI` and collaboration.

### Quick links
- [.. up dir](../README.md)
- [Overview](#overview)
  - [Features](#features)
  - [Basic operations](#basic-operations)
- [Installation](#installation)
  - [Local Zed on NixOS](#local-zed-on-nixos)
  - [Remote Zed](#remote-zed)
- [Commands](#commands)
  - [Cheat Sheet](#cheat-sheet)
- [Configuration](#configuration)
  - [File locations](#file-locations)
  - [Out of the box](#out-of-the-box)
  - [Color theme](#color-theme)
  - [Icon theme](#icon-theme)
  - [Extensions](#extensions)
  - [Settings](#settings)
  - [Key bindings](#key-bindings)
- [Claude Code integration](#claude-code-integration)
  - [Close Agent panel](#close-agent-panel)
- [Debugging](#debugging)

- [Troubleshooting](#troubleshooting)

## Overview

### Features
* Multibuffer editing
* Extensible via extension ecosystem
* First-class modal editing via Vim bindings

### Basic operations
* Open configuration page `Ctrl+,`
* Control buffer font size `Ctrl+=` or `Ctrl+-`

## Installation

**References**
* [Zed - NixOS Wiki](https://wiki.nixos.org/wiki/Zed)

### Local Zed on NixOS
This is the installation on you local machine, i.e. normally, vs being installed on a remote machine, 
which is a feature Zed provides. 

**References**
* [Zed Docs - Linux](https://zed.dev/docs/linux)

1. You need to use latest unstable, for me that means adding an overlay exception
   ```
   zed-editor = pkgs-unstable.zed-editor;
   rust-analyzer = pkgs-unstable.rust-analyzer;
   ```

2. Add the install configuration
   ```
   environment.systemPackages = [
     pkgs.zed-editor
     pkgs.rust-analyzer
   ];
   ```

### Remote Zed
Zed has the ability to work with a remote system from your local system. In order to do so though the 
Zed remote server needs installed on the remote system. As such the `zed-editor` package also 
produces a `remote_server` binary that matches the client version that is accessible in Nix as 
`${pkgs.zed-editor.remote_server}/bin/remote_server`

## Commands

### Cheat Sheet
| Short cut       |  Description
| --------------- | ---------------------------------------
| `ctrl+shift+p`  | Open the command palette
| `ctrl+,`        | Open `~/.config/zed/settings.json`
| `ctrl+;`        | Toggle showing or hiding the line numbers
| `shift+;`       | When in command mode simply hit `:`

## Configuration
All configuration is done through the `settings.json` file which is required to be writable by Zed to 
function properly. To see all availble settings launch the command palette and search for `zed: open 
default settings`

### File locations
* `~/.config/zed`
* `~/.config/zed/settings.json`
* `~/.local/share/zed`

### Out of the box
First run will splash up some simple configuration options

1. Enable VIM mode
2. Enable and login to Copilot
   * Zed supports many AI ecosystems
3. Keep default vscode keybindings

```
"lsp": {
   "rust-analyzer": {
     "binary": {
       "path": "/run/current-system/sw/bin/rust-analyzer",
     },
   }
 }
```

### Color theme
You can select the theme you'd like or install a new one with:

1. Navigate to `MENU >Select Theme...`
2. Click the `Install Themes` button
3. Filter for `Tokyo Night Themes`
4. Click the `Install` button to the right
5. Press `Ctrl+k, Ctrl+t` then select `Tokyo Night Storm`

### Icon theme
You can select the icon theme you'd like or install a new one with:

1. Navigate to `MENU >Select Icon Theme...`
2. Click the `Install Icon Themes` button
3. Filter for `VSCode Great Icons Theme`
4. Click the `Install` button to the right
5. Navigate to `MENU >Select Icon Theme...`
6. Choose `VSCode Great Icons Theme`

### Extensions
Press `Ctrl+Shift+x` to open up extensions

1. Select the `Languages` tab
   1. Install `TOML`
   2. Install `Dockerfile`

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
  "ui_font_size": 18,
  "buffer_font_size": 15,
  "theme": {
    "mode": "system",
    "light": "Solarized Dark",
    "dark": "One Dark"
  }
}
```

### Keybindings
* Close buffer `Ctrl+w`

## Claude Code integration
Claude Code will just work essentially if you've already installed it in the terminal. Zed will pick
up that you've already logged in there and just use it, but if not then run `/login` and go throught
the regular terminal login process that will pop open a browser and ask you to choose how you want to
login then ask you to authorize.

Zed will handle installing the Claude Code adapter automatically on the first use, no manual
extension is required.

* Claude will run in Zed's agent panel rather than a separate terminal.
* Multi-file edits show up as diffs you can accept/reject individually
* Task progress is visible in the sidebar

### Close Agent panel
By default `Ctrl+Alt+b` is used to open or close the agent panel which is aweful.

## Debugging


## Troubleshooting

### Save All
Pressing `Ctrl+Shift+s` triggers and error with message `The name org.freedesktop.portal.Desktop` was 
not provided by any .service files
