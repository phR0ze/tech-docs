# Cursor <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Cursor laude's itself as the AI Code Editor. Essentially the took the open source Code editor and 
layered in AI to make development easier. This has the advantage of being more thoroughly integrated 
into the editor than Copilot which is only an extension.

### Quick links
- [.. up dir](..)
* [Overview](#overview)
* [Getting Started](#getting-started)
  * [Install Cursor](#install-cursor)
  * [Configure via First Run](#configure-via-first-run)
* [Configure personality](#configure-personality)
  * [Benefits of rules](#benefits-of-rules)

## Overview
Not only does Cursor have AI integrated into the actual editor which makes it more natural and easy 
to work with but the quality of responses coming from the chat bot are way more useful than what I've 
seen with Copilot even with apparently the same models being used. Cursor was predicting my needs 
without prompting while simply moving my mouse to the next location and learned from my changes when 
doing so. This seems light years ahead of Copilot. 

The most telling request I had was when I asked Copilot to build a Dart client based on my swagger 
specification and it simply told me it couldn't and when pushed told me to use swagger client 
generation tools and wouldn't change its mind. Cursor didn't even blink and built the entire thing 
for me in under 30sec.

Pricing though seems to be way problematic with Cursor. First of all it is double the cost of Copilot 
and then only gives you a limited 500 premium requests per month. In my simple experimentation for 
2.5hrs I was able to burn through 55 of those requests without even trying to.

## Getting Started

### Install Cursor
From a developer perspective creating a `flake.nix` for your project that then specifies the specific 
version of cursor that you want is a great way to get started.

```flake.nix
{
  inputs = {
    nixpkgs.url = "github:nixos/nixpkgs/2795c506fe8fb7b03c36ccb51f75b6df0ab2553f";
    flake-utils.url = "github:numtide/flake-utils";
  };
  outputs = { self, nixpkgs, flake-utils, ... }: flake-utils.lib.eachDefaultSystem (system: let
    pkgs = import nixpkgs {
      inherit system;
      config.allowUnfree = true;
    };
  in
  {
    devShells.default = pkgs.mkShell {

      # Supporting tooling
      packages = with pkgs; [
        mysql-workbench # Useful for designing relational table EER Diagrams
        sqlitebrowser   # Useful for examining the database
        code-cursor     # AI powered version of code
      ];

      # Build packages
      # * By calling out both rustc and glibc here I was able to work around a GLIBC versioning 
      #   issue I was seeing related to using rustup to install my rustc system version.
      nativeBuildInputs = with pkgs; [
        bashInteractive # Solve for normal shell operation
        pkg-config      # System dependency path resolution
        rustc           # Ensure we have Rust available
        cargo           # Rust build tooling
        glibc           # System dependency for SQLx macros
        sqlx-cli        # SQLx command line tool
      ];

      # Set the rust source path for rust-analyzer to be happy
      RUST_SRC_PATH = "${pkgs.rust.packages.stable.rustPlatform.rustLibSrc}";

      # Launch VSCode in the dev shell
      shellHook = ''
        echo "Launch vscode for flutter with 'code .' or for the api with 'code api'"
      '';
    };
  });
}
```

Simply run `nix develop` in the same directory you created `flake.nix` above

### Configure via First Run
1. Load up cursor
   ```bash
   $ nix develop
   $ cursor
   ```

2. `Log In` or click the tiny grayed out `Skip and continue` for the free option
   1. Click `Log In`, note separately I had already logged into Cursor's web portal via Chromium
   2. After clicking it immediately took me to Chromium and saw I was logged in
   3. Click the `Yes, Log In` button
   4. Back in the app click `Import from VS Code`
   5. Confirm by clicking `Import`
   6. Keep `Cursor Dark` and click `Continue`
   7. Click `Continue`
   8. Note that for a Business plan the team admin can turn data collection on for your device
   9. Check the acceptance box and click `Continue`
  10. Set your language and click `Continue` 

3. Change theme, default is way too dark
   1. Navigate to `File >Preferences >Theme >Color Theme`
   2. Choose `Dark (Visual Studio)`

4. Reinstall Extensions as import didn't work

   | Name                     | Identifier                            |
   | ------------------------ | ------------------------------------- |
   | Even Better TOML         | `tamasfe.even-better-toml`            |
   | Code Runner              | `formulahendry.code-runner`           |
   | Vim                      | `vscodevim.vim`                       |
   | VSCode Great Icons       | `emmanuelbeziat.vscode-great-icons`   |
   | Go                       | `golang.go`                           |
   | Flutter                  | `dart-code.flutter`                   |
   | rust-analyzer            | `rust-lang.rust-analyzer`             |
   | CodeLLDB                 | `vadimcn.vscode-lldb`                 |
   | Dependi                  | `fill-labs.dependi`                   |

5. Change UI font size
   1. Open preferences `Ctrl+Shift+P`

## Configure personality
Cursor can be given a set of rules to follow that will guide it on how to solve particular problems. 
The heart of it is the `.cursorrules` files which given customized AI behavior.

**References**
* [Awesome Cursor Rules](https://github.com/PatrickJS/awesome-cursorrules)
* [Cursor Rust Rules](https://github.com/tyrchen/cursor-rust-rules)

### Benefits of rules

1. ***Customized AI Behavior***: .cursorrules files help tailor the AI's responses to your project's 
   specific needs, ensuring more relevant and accurate code suggestions.

2. ***Consistency***: By defining coding standards and best practices in your .cursorrules file, you can 
   ensure that the AI generates code that aligns with your project's style guidelines.

3. ***Context Awareness***: You can provide the AI with important context about your project, such as 
   commonly used methods, architectural decisions, or specific libraries, leading to more informed 
   code generation.

4. ***Improved Productivity***: With well-defined rules, the AI can generate code that requires less manual 
   editing, speeding up your development process.

5. ***Team Alignment***: For team projects, a shared .cursorrules file ensures that all team members 
   receive consistent AI assistance, promoting cohesion in coding practices.

6. ***Project-Specific Knowledge***: You can include information about your project's structure, 
   dependencies, or unique requirements, helping the AI to provide more accurate and relevant 
   suggestions.

### How to add rules
Create the file `.cursorrules` in the root of your project directory

**Rust example**
```.cursorrules
You are an expert in Rust, async programming, and concurrent systems.

Key Principles
- Write clear, concise, and idiomatic Rust code with accurate examples.
- Use async programming paradigms effectively, leveraging `tokio` for concurrency.
- Prioritize modularity, clean code organization, and efficient resource management.
- Use expressive variable names that convey intent (e.g., `is_ready`, `has_data`).
- Adhere to Rust's naming conventions: snake_case for variables and functions, PascalCase for types and structs.
- Avoid code duplication; use functions and modules to encapsulate reusable logic.
- Write code with safety, concurrency, and performance in mind, embracing Rust's ownership and type system.

Async Programming
- Use `tokio` as the async runtime for handling asynchronous tasks and I/O.
- Implement async functions using `async fn` syntax.
- Leverage `tokio::spawn` for task spawning and concurrency.
- Use `tokio::select!` for managing multiple async tasks and cancellations.
- Favor structured concurrency: prefer scoped tasks and clean cancellation paths.
- Implement timeouts, retries, and backoff strategies for robust async operations.

Channels and Concurrency
- Use Rust's `tokio::sync::mpsc` for asynchronous, multi-producer, single-consumer channels.
- Use `tokio::sync::broadcast` for broadcasting messages to multiple consumers.
- Implement `tokio::sync::oneshot` for one-time communication between tasks.
- Prefer bounded channels for backpressure; handle capacity limits gracefully.
- Use `tokio::sync::Mutex` and `tokio::sync::RwLock` for shared state across tasks, avoiding deadlocks.

Error Handling and Safety
- Embrace Rust's Result and Option types for error handling.
- Use `?` operator to propagate errors in async functions.
- Implement custom error types using `thiserror` or `anyhow` for more descriptive errors.
- Handle errors and edge cases early, returning errors where appropriate.
- Use `.await` responsibly, ensuring safe points for context switching.

Testing
- Write unit tests with `tokio::test` for async tests.
- Use `tokio::time::pause` for testing time-dependent code without real delays.
- Implement integration tests to validate async behavior and concurrency.
- Use mocks and fakes for external dependencies in tests.

Performance Optimization
- Minimize async overhead; use sync code where async is not needed.
- Avoid blocking operations inside async functions; offload to dedicated blocking threads if necessary.
- Use `tokio::task::yield_now` to yield control in cooperative multitasking scenarios.
- Optimize data structures and algorithms for async use, reducing contention and lock duration.
- Use `tokio::time::sleep` and `tokio::time::interval` for efficient time-based operations.

Key Conventions
1. Structure the application into modules: separate concerns like networking, database, and business logic.
2. Use environment variables for configuration management (e.g., `dotenv` crate).
3. Ensure code is well-documented with inline comments and Rustdoc.

Async Ecosystem
- Use `tokio` for async runtime and task management.
- Leverage `hyper` or `reqwest` for async HTTP requests.
- Use `serde` for serialization/deserialization.
- Use `sqlx` or `tokio-postgres` for async database interactions.
- Utilize `tonic` for gRPC with async support.

Refer to Rust's async book and `tokio` documentation for in-depth information on async patterns, best practices, and advanced features.
```
