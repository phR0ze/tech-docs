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
```
target/debug/deps/libsqlx_macros-9f83608d0670ef09.so: /nix/store/8mc30d49ghc8m5z96yz39srlhg5s9sjj-glibc-2.38-44/lib/libc.so.6: version `GLIBC_2.39' not found (required by target/debug/deps/libsqlx_macros-9f83608d0670ef09.so)
  --> .cargo/registry/src/index.crates.io-6f17d22bba15001f/sqlx-0.8.3/src/lib.rs:64:1
