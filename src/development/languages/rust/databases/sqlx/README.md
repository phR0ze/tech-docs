# SQLx <img style="margin: 6px 13px 0px 0px" align="left" src="../../../../../data/images/logo_36x36.png" />

SQLx is a Rust SQL toolkit. It is pure Rust, async and features compile-time checked queries without 
a DSL. It supports PostgreSQL, MySQL and SQLite. SQLx is the rising star in this space and if your 
more an ORM fan then use SeaORM which is built on top of it.

### Quick links
* [.. up dir](..)
* [Overview](#overview)

## Overview

### Runtime features
SQLx is compatible with multiple async runtimes and multiple tls implementations. You need to specify 
via features which combination you intend to use.

Tokio with Rustls and Web PKI
```
sqlx = { version = "0.8", features = [ "runtime-tokio", "tls-rustls-ring-webpki" ] }
```

### Glibc dependency
The issue here seems to be that the rustc and glibc versions can get out of sync based on the way 
they get installed.
```
target/debug/deps/libsqlx_macros-9f83608d0670ef09.so: /nix/store/8mc30d49ghc8m5z96yz39srlhg5s9sjj-glibc-2.38-44/lib/libc.so.6: version `GLIBC_2.39' not found (required by target/debug/deps/libsqlx_macros-9f83608d0670ef09.so)
  --> .cargo/registry/src/index.crates.io-6f17d22bba15001f/sqlx-0.8.3/src/lib.rs:64:1
```

The solution is to build out a dev environment for your project calling out `rustc` and `glibc`
```flake.nix
{
  inputs = {
    nixpkgs.url = "github:nixos/nixpkgs/3566ab7246670a43abd2ffa913cc62dad9cdf7d5";
    flake-utils.url = "github:numtide/flake-utils";
  };
  outputs = { self, nixpkgs, flake-utils, ... }: flake-utils.lib.eachDefaultSystem (system: let
    pkgs = import nixpkgs { inherit system; };
  in
  {
    devShells.default = pkgs.mkShell {

      # Note: by calling out both rustc and glibc here I was able to work around a GLIBC versioning 
      # issue I was seeing although potentially related to using rustup to install my system version.
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
        echo "Launching rust API componet in vscode... `code api`"
      '';
    };
  });
}
```
