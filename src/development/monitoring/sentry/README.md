# Sentry <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

Sentry is an enterprise grade monitoring tool for:
* Error Monitoring
  * Alerts and notifications via email or other intergrations
* Session Replay
* Tracing
* Code Coverage

### Quick links
- [.. up dir](../README.md)
- [Overview](#overview)
  - [Pricing](#pricing)
- [Getting started](#getting-started)
  - [Create a project](#create-a-project)
  - [Integrate with Rust](#integrate-with-rust)
  - [Integrate with Tokio](#integrate-with-tokio)
  - [Sentry Options](#sentry-options)

## Overview
Sentry's primary competition is the venerable Bugsnag, which given modern standards, is getting a 
little long in the tooth. Sentry, scrapy and hungry for adoption, is boasting a lower price point, 
SDKs for virtually every major programming language and a more modern intuitive feel in the web UI. 
Additionally Sentry is breaking into other tangential observability areas as well with tracing, 
session replay and code coverage.

**References**
* [Platform support - Sentry docs](https://docs.sentry.io/platforms/)
* [Workflow - Sentry docs](https://sentry.io/resources/the-sentry-workflow/)

### Pricing
Free for one user for unlimited projects with a limit of 5k errors, 10M spans, 50 replays and only 1 
uptime monitor and 1 cron monitor.

## Getting started

### Create a project
Sentry project's scope events to a distinct application in your environments and assign 
responsibility to specific users and teams. Projects can be created for a particular application or 
component of your application. Creating separate projects for your API server and your frontend 
client might allow for better ownership assignment and more granular alerts.

**References**
* [Create a project - Sentry docs](https://docs.sentry.io/product/sentry-basics/integrate-frontend/create-new-project/)
* [Best practices - Sentry docs](https://docs.sentry.io/organization/getting-started/#4-create-projects)

1. Create a new project for your application. This can be a single project for your app or a project 
   per component in your application. Most likely you'll want at least one per Git repo. You'll also 
   want to ensure that different languages have their own projects.
   1. Log in to your Sentry organization
   2. Select the `Projects` menu option on the left and click `Create Project` top right
   3. Choose the corresponding language or platform for your application or component e.g. `Rust`
   4. Choose `I'll create my own alerts later`
   5. Give the project a name e.g. `my-project`
   6. Click `Create team` e.g. `my-team`
   6. Click `Create Project`
2. Extract the DSN for your project
   1. Navigate to `USER > Organizational Settings >Projects >my-project`
   2. In the second tier menu choose `SDK SETUP >Client Keys (DSN)`

### Integrate with Rust app
***Note:*** you will need your `DSN (Data Source Name)` from the Sentry UI created in the previous step to 
tell the Sentry client where to send events.

**References**
* [Sentry for Rust - Sentry docs](https://docs.sentry.io/platforms/rust/)

1. Initialize sentry in your code, keep a handle to the guard and remember your DSN. Note that Sentry 
   must be initialized before any async runtime is run. This means that the typical 
   `#[actix_web::main]` attribute can't be used to decorate your main function and instead tokio must 
   be run manually.
   ```rust
   fn main() {
     let _guard = sentry::init(("https://examplePublicKey@o0.ingest.sentry.io/0", sentry::ClientOptions {
       release: sentry::release_name!(),
       // Capture user IPs and potentially sensitive headers when using HTTP server integrations
       // see https://docs.sentry.io/platforms/rust/data-management/data-collected for more info
       ..Default::default()
     }));
     tokio::runtime::Builder::new_multi_thread().enable_all().build().unwrap()
       .block_on(async {
           // implementation of main
       });
     }
   }
   ```
3. The quickest way to verify Sentry is communicating back to the mother ship is with a panic
   ```rust
   fn main() {
     let _guard = sentry::init(("https://examplePublicKey@o0.ingest.sentry.io/0", sentry::ClientOptions {
       release: Some("my-project@2.3.12+123".into()),
       environment: Some("production".into()),
       ..Default::default()
     }));
 
     // Sentry will capture this
     panic!("Everything is on fire!");
   }
   ```

### Integrate with Tokio
Sentry has support for the Tokio Tracing crate in threee ways:
* Events can be captured as breadcrumbs for later
  * By default events above `Info` are recorded as breadcrumbs
* Error events can be captured as events to Sentry
  * By default events above `Error` are recorded as errors
* Spans can be recorded as structured transaction events
  * By default spans above `Info` are recorded as transactions
  * By default `traces_sample_rate` is set to `0.0` which means don't send traces

**Configure Sentry as a Tracing subscriber**
1. First call the `sentry::init` from the [Integrate with Rust](#integrate-with-rust) section

2. Add in the Subscriber crate `sentry-tracing` and configuration 
   ```rust
   use sentry_tracing::EventFilter;
   use tracing_subscriber::prelude::*;
   
   let sentry_layer = sentry_tracing::layer().event_filter(|md| match md.level() {
       &tracing::Level::ERROR => EventFilter::Event,
       _ => EventFilter::Ignore,
   });
  sentry_tracing::layer::<Registry>().event_filter(|x| match x.level() {
                        &Level::ERROR => sentry_tracing::EventFilter::Event,
                        _ => sentry_tracing::EventFilter::Ignore,
                    }),
   
   tracing_subscriber::registry()
       .with(tracing_subscriber::fmt::layer())
       .with(sentry_layer)
       .init();
   ```

### Sentry Options

**References**
* [Sentry options - Sentry docs](https://docs.sentry.io/platforms/rust/configuration/options/)

The following options can be passed in at init time
* ***dsn*** - tells the SDK where to send the events, if not set reads ***SENTRY_DSN***
* ***release*** - free form string indicating the project name, falls back on ***SENTRY_RELEASE***
* ***environment*** - free form string for development hierarchy, falls back on ***SENTRY_ENVIRONMENT***
* ***max_breadcrumbs*** - default 100
