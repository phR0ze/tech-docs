<img align="left" width="48" height="48" src="../../images/logo_256x256.png">vscode
====================================================================================================
Develop with Visual Studio Code
<br><br>

### Quick links
* [.. up dir](..)
* [Install](#install)
* [Install Extensions](#install-extensions)
* [Keyboard Shortcuts](#keyboard-shortcuts)
  * [tasks.json](#tasks-json)
* [General Settings](#general-settings)
  * [Powerline glyphs in terminal](#powerline-glyphs-in-terminal)
* [Language Config](#language-config)
  * [Golang](#golang)
    * [Install Golang](#install-golang)
    * [Config Golang](#config-golang)
    * [Debug with dlv](#debug-with-dlv)
  * [Ruby](#ruby)
    * [Config Ruby](#config-ruby)
    * [Debug Ruby](#debug-ruby)
  * [Rust](#rust)
    * [Install Rust](#install-rust)
    * [Config Rust](#config-rust)
    * [Rust tasks](#rust-tasks)
* [Developing inside a Container](#developing-inside-a-container)
* [Troubleshooting](#troubleshooting)
  * [Remove All Extensions](#remove-all-extensions)

# Install
```bash
# Install from the cyberlinux repo
$ sudo pacman -S visual-studio-code-bin ripgrep

# Build and install from source
$ yay -Ga visual-studio-code-bin
$ cd visual-studio-code-bin; makepkg -s
$ sudo pacman -U visual-studio-code-bin-1.8.1-3x86_64.pkg.tar.xz
```

# Install Extensions
1. Launch `code`

2. Click the button on the left that looks like an extension icon

3. Install General extensions:
   | Name                 | Identifier                            |
   | -------------------- | ------------------------------------- |
   | Vim                  | `vscodevim.vim`                       |
   | VSCode Great Icons   | `emmanuelbeziat.vscode-great-icons`   |
   | Code Runner          | `formulahendry.code-runner`           |
   | Better TOML          | `bungcip.better-toml`                 |
   | Docker               | `ms-azuretools.vscode-docker`         |

4. Install Golang extensions
   | Name                 | Identifier                            |
   | -------------------- | ------------------------------------- |
   | Go                   | `golang.go`                           |

5. Install Rust extensions
   | Name                 | Identifier                            |
   | -------------------- | ------------------------------------- |
   | rust-analyzer        | `matklad.rust-analyzer`               |
   | CodeLLDB             | `vadimcn.vscode-lldb`                 |
   | crates               | `serayuzgur.crates`                   |

# Keyboard Shortcuts
Hit `Ctrl+Shift+p` and search for keyboard then choose `Preferences: Open Keyboard Shortcuts`

| Key sequence   | Description             |
| -------------- | ----------------------- |
| Ctrl+Shift+b   | Run Build Task          |
| Ctrl+Shift+n   | Open new instance       |
| Ctrl+Shift+p   | Open command search     |
| Ctrl+Shift+s   | File: Save All          |
| Ctrl+,         | Open settings           |    
| F12            | Go to Definition        |
| Ctrl+Shift+t   | Run Test Task           |

Keyboard shortcut overrides are found in: `~/.config/Code/User/keybindings.json`
```json
[
    {
        "key": "ctrl+shift+s",
        "command": "workbench.action.files.saveAll"
    },
    {
        "key": "ctrl+shift+t",
        "command": "workbench.action.tasks.test"
    },
    {
        "key": "ctrl+shift+r",
        "command": "workbench.action.tasks.runTask",
        "args": "Run"
    }
]
```

## tasks.json
Every project requires the creation of the `.vscode/tasks.json` file to map keybindings to your 
specific project.

* see [Rust tasks](#rust-tasks)

# General Settings
Configuration is saved at `~/.config/Code/User/settings.json`

Hit `Ctrl+Shift+p` and search for `json` and select `Preferences: Open Settings(JSON)`
```json
{
    // General configuration
    "explorer.confirmDelete": false,
    "explorer.confirmDragAndDrop": false,
    "telemetry.enableTelemetry": false,
    "telemetry.enableCrashReporter": false,
    "workbench.iconTheme": "vscode-great-icons",
    "terminal.integrated.fontFamily": "InconsolataGo Nerd Font Mono",
    "terminal.integrated.fontSize": 16,

    // Editor configuration
    "editor.tabSize": 4,
    "editor.insertSpaces": true,
    "editor.minimap.enabled": true,
    "editor.fontSize": 14,
    "editor.fontFamily": "Inconsolata-g",
    "editor.formatOnPaste": true,
    "editor.formatOnSave": true,

    "vim.handleKeys": {
        "<C-a>": false,
        "<C-b>": false,
        "<C-c>": false,
        "<C-e>": false,
        "<C-f>": false,
        "<C-h>": false,
        "<C-i>": false,
        "<C-j>": false,
        "<C-k>": false,
        "<C-n>": false,
        "<C-p>": false,
        "<C-s>": false,
        "<C-t>": false,
        "<C-u>": false,
        "<C-v>": false,
        "<C-o>": false,
        "<C-w>": false,
        "<C-x>": false,
        "<C-y>": false,
        "<C-z>": false
    },

    // Golang configuration
    "go.gopath": "~/Projects/go",           // Set the GOPATH to use
    "go.formatTool": "goimports",           // Use specific external format tool for go
    "go.useLanguageServer": true,           // Use the new gopls language server
    "[go]": {
        "editor.snippetSuggestions": "none",
        "editor.formatOnSave": true,
        "editor.codeActionsOnSave": {
            "source.organizeImports": true
        }
    },
    "gopls": {
        "usePlaceholders": false,           // add parameter placeholders when completing a function
        "completionDocumentation": true     // for documentation in completion items
    },
    "go.toolsManagement.autoUpdate": true,  // autoupdate gopls tools
    "files.eol": "\n",                      // gopls formatting only supports LF line endings

    // Rust configuration
    "rust-analyzer.hover.actions.enable": false,
    "rust-analyzer.inlayHints.typeHints.enable": false,
    "rust-analyzer.inlayHints.parameterHints.enable": false,
}
```

## Powerline glyphs in terminal
Powerline depends on fonts that support the particular glyphs that it uses. In order to get them to 
show up properly you need to install the right fonts then set VSCode to use the correct fonts for the 
terminal.

1. Install dependency fonts
   ```bash
   $ sudo pacman -S nerd-fonts-inconsolata-go
   ```
2. Determine the font name to use
   ```bash
   $ fc-list | grep -i inconsolata
   /usr/share/fonts/TTF/ttf-inconsolata-g.ttf: Inconsolata\-g:style=g
   ...
   /usr/share/fonts/TTF/InconsolataGo Nerd Font Complete Mono.ttf: InconsolataGo Nerd Font Mono:style=Regular
   ```
3. Copy out the portion after the first colon
   ```
   Inconsolata-g
   InconsolataGo Nerd Font Mono
   ```
3. Hit `Ctrl+Shift+p` and choose `Preferences: Open Settigns (JSON)` then add
   ```json
   "editor.fontFamily": "Inconsolata-g"
   "editor.fontSize": 14
   "terminal.integrated.fontFamily": "InconsolataGo Nerd Font Mono"
   "terminal.integrated.fontSize": 16
   ```

# Language Config

## Golang

### Install Golang
Install golang dependencies;

```bash
$ sudo pacman -S go go-tools go-bindata delve
```

### Config Golang
1. Set go path in `~/.bashrc`:    
   `export GOPATH=~/Projects/go`  

2. Install Extensions:  
   a. Click the button on the left that looks like an extension icon  
   b. Install ***Go ms-vscode.go***  

3. Add Golang Server Configs:  
   [Go Language](https://github.com/microsoft/vscode-go#go-language-server)  
   Note: you have to open a go file before the Go extension is officially activated  
   a. Hit `Ctrl+Shift+p` and enter `Preferences: Open Settings (JSON)` and paste  
   ```bash
   "go.gopath": "~/Projects/go", // Set the GOPATH to use
   "go.formatTool": "goimports", // Use specific external format tool for go
   "go.useLanguageServer": true, // Use the new gopls language server
   "go.languageServerExperimentalFeatures": {
       "documentLink": false, // Just reference local code via imports not external github pages with popup
       //"signatureHelp": false,
       // "format": true,
       // "autoComplete": true,
       // "diagnostics": true,
       // "goToDefinition": true,
       // "hover": true,
       // "goToTypeDefinition": true,
       // "findReferences": true,
       // "goToImplementation": true
   },
   "[go]": {
       "editor.snippetSuggestions": "none",
       "editor.formatOnSave": true,
       "editor.codeActionsOnSave": {
           "source.organizeImports": true
       }
   },
   "gopls": {
       "usePlaceholders": false, // add parameter placeholders when completing a function
       "completionDocumentation": true // for documentation in completion items
   },
   "files.eol": "\n", // Gopls formatting only supports LF line endings
   ```

### Debug with dlv
The golang extension that we installed ealier will fire the first time you open a golang project and 
ask if you want to download the missing helper tools, say yes to all and it will install `dlv-dap` 
into your go path and keep it up to date i.e. `~/Projects/go/bin` for cyberlinux.

`dlv-dep` should just work out of the box once you have a config setup below.

**Create a new debug config, example replace `PACKAGE` below with your package**
```
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Launch",
            "type": "go",
            "request": "launch",
            "mode": "auto",
            "program": "${workspaceFolder}/cmd/PACKAGE",
            "env": { },
            "args": [
            ]
        }
    ]
}
```

## Ruby

**References**
* [Visual Studio Code Ruby guide](https://code.visualstudio.com/docs/languages/ruby)

### Config Ruby
`Ruby LSP` is meant to replace Solargraph, rubocop, byebug and others

1. Launch `code`
2. Install Ruby version manager
   1. Download AUR package: `yay -Ga rbenv`
   2. Build: `makepkg -s`
   3. Install: `sudo pacman -U rbenv-1.2.0-1-any.pkg.tar.zst`
   4. Add `eval "$(rbenv init - bash)"` to your `~/.bashrc`
3. Install Ruby LSB extension 
   1. Navigate to extensions
   2. Install `Ruby LSP` from `Shopify`
   3. Press `Ctrl+,` to load Settings
   4. Search for Ruby and select `Ruby LSP` on the left
   5. Select `Ruby Version Manager` drop down and pick `rbenv`

### Debug Ruby
https://dev.to/dnamsons/ruby-debugging-in-vscode-3bkj

```json
{
  // Use IntelliSense to learn about possible attributes.
  // Hover to view descriptions of existing attributes.
  // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Listen for rdebug-ide",
      "type": "Ruby",
      "request": "attach",
      "cwd": "${file}",
      "remoteHost": "127.0.0.1",
      "remotePort": "1234",
      "remoteWorkspaceRoot": "${file}"
    }
  ]
}
```

## Rust

### Install Rust
The Rust `toolchain` is all the necessary build components for your local system while a `target` is 
the ability to cross compile to another platform.

1. Install rust via rustup and the rust debugger lldb
   ```bash
   $ sudo pacman -S rustup lldb
   $ rustup default stable
   ```
2. List all available targets and see which are installed
   ```bash
   $ rustup target list
   ```
3. Install musl target
   ```bash
   $ rustup target add x86_64-unknown-linux-musl
   ```
3. Install android targets for NDK dev
   ```bash
   $ rustup target add aarch64-linux-android armv7-linux-androideabi i686-linux-android x86_64-linux-android
   ```

### Config Rust
1. Install and configure language server:
   a. Install extension `rust-analyzer by matklad.rust-analyzer`  
   b. Click `Yes` bottom right to install helper tooling  
2. Install and configure debugger support:
   a. Install extension `CodeLLDB by Vadim Chugunov`  
   b. Now switch to debug mode and click drop down `Add Configuration...` then `LLDB`  
   c. Cargo should popup a dialog click `Yes` which generates   
   d. Add `"sourceLanguages": ["rust"]` which will add two debug configs
3. Install and configure crate support:
   a. Install extension `Better TOML by bungcip`  
   b. Install extension `crates by Seray Uzgur`  
4. Configure Rust Analyzer to not show automatic types

### Rust tasks
If you use a `workbench.action.tasks.runTask` in your keybindings it points to your local 
`.vscode/tasks.json` file which you can then associate to a specific task using the keybinding's 
`args` to your tasks.json's `label` mapping. `Build` and `Test` are predetermined labels that vscode 
already understands.

```bash
$ mkdir -p .vscode
tee .vscode/tasks.json <<EOL
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "Build",
      "type": "shell",
      "command": [
        "cargo build --all-features",
      ],
      "group": {
        "kind": "build",
        "isDefault": true
      }
    },
    {
      "label": "Test",
      "type": "shell",
      "command": "cargo test --all-features",
      "group": {
        "kind": "test",
        "isDefault": true
      }
    },
    {
      "label": "Run",
      "type": "shell",
      "command": "cargo run"
    }
  ]
}
EOL
```

# Developing inside a Container
Scripting languages like Ruby and Python frequently roll out changes to the language and library 
pieces breaking a system install. To keep the system up to date with the madness is a full time job. 
The only way to develop sanely is to do so from a stable development environment and that is what the 
[Developing inside a container](https://code.visualstudio.com/docs/devcontainers/containers) 
extension and paradigm brings. It is the ability to have a stable well defined environment that won't 
change unless you want it to.

Workspace files are mounted from the local file system. Extensions are installed and run inside the 
container, where they have full access to the tools, platform, and file system. This means that you 
can seamlessly switch your entire development environment just by connecting to a different 
container.

![Dev inside container image](images/dev-env-inside-container.png)

This lets VS Code provide a local-quality debugging experience including full code completions, code 
navigation, and debugging regardless of where your tools and code are located.

1. You can use a container as your full-time development environment
2. You can attach to a running container to inspect it

## Dev Containers Prerequisites
1. You need VS Code installed
2. You need Docker installed

The `devcontainer.json` file in your project tells VS Code how to access or create a development 
container.


# Troubleshooting

## Remove All Extensions
Extensions are stored in ***~/.vscode/extensions***

To clean up all extensions simply remove this directory
