# Rust

### Quick links
* [Install Rust](#install-rust)
* [Config Rust](#config-rust)
* [Rust tasks](#rust-tasks)

### Install Rust
see [Install Rust](../../../languages/rust/#install-rust)

### Install Rust Nightly
Rust nightly as installed by `rustup` supplies unpatched binaries out of the box that don't work with 
***NixOS*** background found at https://github.com/oxalica/rust-overlay.

1. Create the development shell `flake.nix` in your project
   ```nix
   {
     description = "Dev shell for rust nightly";
   
     inputs = {
       nixpkgs.url      = "github:NixOS/nixpkgs/nixos-unstable";
       rust-overlay.url = "github:oxalica/rust-overlay";
       flake-utils.url  = "github:numtide/flake-utils";
     };
   
     outputs = { self, nixpkgs, rust-overlay, flake-utils, ... }:
       flake-utils.lib.eachDefaultSystem (system:
         let
           overlays = [ (import rust-overlay) ];
           pkgs = import nixpkgs {
             inherit system overlays;
           };
         in
         with pkgs;
         {
           devShells.default = mkShell {
             buildInputs = [
               openssl
               pkg-config
               (
                 rust-bin.selectLatestNightlyWith (toolchain: toolchain.default.override {
                   extensions = [
                     "rust-src"
                     "rust-analyzer"
                   ];
                 })
               )
             ];
           };
         }
       );
   }
   ```
2. Add the file to git
   ```bash
   $ git add flake.nix
   ```

3. Execute dev shell
   ```bash
   $ nix develop
   ```

### Config Rust
My new NixOS based operating system has my extensions built in

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
