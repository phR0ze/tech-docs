# Tracing <img style="margin: 6px 13px 0px 0px" align="left" src="../../../../../data/images/logo_36x36.png" />

A distributed trace, more commonly known as a trace, records the paths taken by requests (made by an 
application or end-user) as they propagate through multi-service architectures, like microservice 
and serverless applications.

### Quick links
- [.. up dir](../README.md)
* [Overview](#overview)
* [Fasttrace](#fastrace)
  * [Add fastrace dependencies](#add-fastrace-dependencies)
  * [Initialize fastrace with Axum](#initialize-fastrace-with-axum)
  * [Add fastrace to a function](#add-fastrace-to-a-function)

* [Jaeger](#jaeger)

## Overview
Distributed tracing lets you ***observe requests as they propagate*** through complex, distributed 
systems. Distributed tracing improves the visibility of your application or system's health and lets 
you debug behavior that is difficult to reproduce locally. It is essential for distributed systems, 
which commonly have nondeterministic problems or are to complicated to reproduce locally.

A ***log*** is a timestamped message emitted by services or other components. Unlike traces, they 
aren't necessarily associated with an particular user request or transaction. Logs 

A ***span*** represents a unit of work or operation. Spans track specific operations that a request 
makes, painting a picture of what happened during the time in which that operation was executed. A 
span contains a name, time-related data, structued log messages and other metadata.

* `opentelemetry` -  Rust OpenTelemetry implementation
* `tokio` - An asynchronous runtime for Rust
* `tracing` - Application-level tracing for Rust
* `tracing-opentelemetry` - OpenTelemetry integration for tracing
* `tracing-subscriber` - Utilities for implementing and composing `tracing` subscribers

## Tracing
Tracing is the defacto standard for logging and distributed traces in the Rust ecosystem.

**References**
* [Asynchronous Rust: Tracing and Metrics](https://www.youtube.com/watch?v=YHo_ab5S1bo)

## Fasttrace
Fastrace provides a production-ready solution with seamless ecosystem integration, out-of-box 
OpenTelemetry support, and a more straightforward API that works naturally with the existing logging 
infrastructure.

**References**
* [Fastrace a modern approach](https://fast.github.io/blog/fastrace-a-modern-approach-to-distributed-tracing-in-rust/)

***Fastrace***
* Pros
  * Out of box OpenTelemetry support
  * Straightforward API that works naturally with existing logging
  * Uses existing `log::info` facade for logging
    * Libraries can use fastrace without forcing users into a specific logging ecosystem
  * Zero cost disable
  * Minimal code required to include tracing
* Cons
  * Less pervasive

***Tracing***
* Pros
  * More pervasive
* Cons
  * Uses its own `tracing::info` instead of the standard `log::info`
  * Overhead can be substantial when instrumented
  * Doesn't have an easy way to enable/disable and requires developer feature flags
  * No context propagation out of the box

### Ecosystem
The Fastrace project has done a lot of work to ensure that it is compatible with the ecosystem.

* `fastrace` - core Fastrace support
* `fastrace-jaeger` - Jaeger reporter/exporter for fastrace telemetry
* `fastrace-opentelemetry` - OpenTelemetry reporter for fastrace telemetry
* `fastrace-reqwest` - Fastrace integration with reqwest for context propagation
* `fastrace-axum` - Fastrace integration enabling seamless trace context propagation across gRPC apps
* `fastrace-tonic` - support for Rust implementation of gRPC that puts mobile and HTTP/2 first
* `logforth`

### Add fastrace dependencies
```toml
[dependencies]
logforth = { version = "0.25.0", features = ["fastrace"] }
fastrace = "0.7.9"
fastrace-axum = "0.1"
fastrace-tracing = "0.1"
tracing = "0.1.41"
tracing-subscriber = "0.3"
```

### Initialize fastrace with Axum
```rust
use fastrace::collector::Config;
use fastrace::collector::ConsoleReporter;
use tracing_subscriber::layer::SubscriberExt;

#[tokio::main]
async fn main() {

    // Configure logging and tracing
    setup_observability("user-service");

    let app = axum::Router::new()
        .route("/ping", axum::routing::get(ping))

        // Add the FastraceLayer to routes which extracts the trace context from incoming requests.
        .layer(fastrace_axum::FastraceLayer);

    let listener = tokio::net::TcpListener::bind("0.0.0.0:8080").await.unwrap();
    axum::serve(listener, app).await.unwrap();

    // Ensure all collected tracing is reported before shutdown
    fastrace::flush();
}

fn setup_observability(service_name: &str) {

    // Because Axum is instrumented with tracing we need to setup a compatibility layer to allow
    // fastrace to receive the tracing spans from Axum
    tracing::subscriber::set_global_default(
        tracing_subscriber::Registry::default().with(fastrace_tracing::FastraceCompatLayer::new()),
    ).unwrap();

    // Setup logging with logforth
    logforth::stderr()
        .dispatch(|d| {
            d.filter(log::LevelFilter::Info)
                // Attaches trace id to logs
                .diagnostic(logforth::diagnostic::FastraceDiagnostic::default())

                // Attaches logs to spans
                .append(logforth::append::FastraceEvent::default())
        }).apply();

    // Configure fastrace reporter
    fastrace::set_reporter(ConsoleReporter, Config::default());

    // Optionally configure Jaeger reporter
    fastrace::set_reporter(
        fastrace_jaeger::JaegerReporter::new("127.0.0.1:6831".parse().unwrap(), service_name).unwrap(),
        fastrace::collector::Config::default()
    );
}
```

### Add fastrace to a function
```rust
#[fastrace::trace]  // Automatically creates and manages spans
async fn fetch_user_details(user_id: &str) -> UserDetails {
    let client = reqwest::Client::new();

    // Standard log calls are automatically associated with the current span
    log::info!("Fetching user {}", user_id);

    let response = client.get(&format!("https://user-details-service/users/{}", user_id))

        .headers(fastrace_reqwest::traceparent_headers())  // Automatic trace context propagation

        .send().await.expect("Request failed");

    response.json::<UserDetails>().await.expect("Failed to parse JSON")
}
```

## Jaeger
Jaeger is an open source, distributed tracing platform for monitoring and troubleshooting workflows 
of complex distributed systems. 

**References**
* [Jaeger tracing](https://www.dash0.com/knowledge/what-is-jaeger-tracing)
* [Jaeger architecture](https://www.retit.de/open-source-apm-tool-overview-jaeger/)

In practice what this means is Jaeger will:
* Receive traces from clients and store the traces for later retrieval
* Visually present distributed tracing in their UI
