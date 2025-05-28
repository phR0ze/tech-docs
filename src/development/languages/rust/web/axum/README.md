# Axum <img style="margin: 6px 13px 0px 0px" align="left" src="../../../../../data/images/logo_36x36.png" />

Axum is a web framework built on top of `hyper` by the same team that made Tokio. Is compatible with 
the Tower middleware ecosystem and focuses on performance and simplicity. Axum brings Rust's 
`async/await` functionality to the forefront with Tokio.

### Quick links
* [.. up dir](..)
* [Overview](#overview)
  * [Getting started](#getting-started)
  * [Environment variables](#environment-variables)
  * [Database access](#database-access)
  * [Authn and Authz](#authn-and-authz)

## Overview

**Resources**
* [High performance REST APIs](https://www.twilio.com/en-us/blog/build-high-performance-rest-apis-rust-axum)
* [Axum Examples](https://github.com/tokio-rs/axum/tree/main/examples)
* [Service static files with Axum](https://benw.is/posts/serving-static-files-with-axum)
* [Axum Rust API](https://rust-api.dev/docs/part-1/tokio-hyper-axum/)

### Getting started

1. Create a new project
   ```bash
   $ cargo new my_rest_api
   $ cd my_rest_api
   ```
2. Add project dependencies
   ```bash
   $ cargo add axum
   $ cargo add tokio -F full
   $ cargo add tower-http -F cors
   $ cargo add serde_json
   $ cargo add serde -F derive
   $ cargo add chrono -F serde
   $ cargo add dotenv
   $ cargo add uuid -F "serde v4"
   $ cargo add sqlx --features "derive macros migrate runtime-tokio sqlite tls-rustls uuid"
   ```

### Environment variables
Rust has a number of options in the environment variable space:
* `envy` - deserializes environment variables into a struct using serde
* `dotenvy` - loads from a `.env` file
* `dotenv` - unmaintained older version

Typically devs will use both `envy` and `dotenvy`

### Database access
SQLx is the rising star in the SQL management space. SQLx isn't an ORM, is lighter and more versatile 
than the likes of SeaORM.

## Tower-http
Provides axum the ability to serve static files

Add the dependency to your project
```toml
[dependencies]
tower-http = { version = "0.4.0", features = ["fs", "trace"] }
```
