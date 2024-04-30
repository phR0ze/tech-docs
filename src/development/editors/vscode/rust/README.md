# Rust

### Quick links
* [Install Rust](#install-rust)
* [Config Rust](#config-rust)
* [Rust tasks](#rust-tasks)

### Install Rust
The Rust `toolchain` is all the necessary build components for your local system while a `target` is 
the ability to cross compile to another platform.

1. In my NixOS based distro `rustup` and `lldb` are already installed so just install rust
   ```bash
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

<!-- 
vim: ts=2:sw=2:sts=2
-->
