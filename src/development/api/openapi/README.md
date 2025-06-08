# OpenAPI <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

The OpenAPI specification (OAS), formerly Swagger spec, is the de facto standard for defining RESTful 
APIs. Having a OAS allows for loading your contract into powerful tooling to analyze, test and 
interact with it. It allows both humans and machines the ability to discover and understand the 
capabilities of the service without the source code or reading documentation.

### Quick links
* [.. up dir](..)
* [Overview](#overview)
  * [Benefits of OAS](#benefits-of-oas)
  * [OAS Example](#oas-example)
* [OAS for Rust APIs](#oas-for-rust-apis)

## Overview
API documentation is the information that is required to successfully consume and integrate with an 
API. This can be in the form of technical writing, code samples and examples for better understanding 
how to consume an API. Concise and clear docs allows your API consumers to adopt it into their 
application quickly and is expected for any modern api.

**References**
* [Learn OpenAPI Specification](https://learn.openapis.org/)
* [Working with OpenAPI using Rust](https://www.shuttle.dev/blog/2024/04/04/using-openapi-rust)
* [Open API 3.0 Overview](https://swagger.io/docs/specification/v3_0/about/)
* [Documenting APIs with Swagger](https://swagger.io/resources/articles/documenting-apis-with-swagger/)

OAS defines an API's contract, allowing all the API's stakeholders to understand what the API does 
and interact with its various resources. The contract is language agnostic and human readable.

Swagger is a set of open-source tools built around the OpenAPI Specification
* `Swagger Editor` - browser based editor where you can write OpenAPI definitions
* `Swagger UI` - browser based rendering of OAS as interactive documentation
* `Swagger Codegen` - generate server stubs and client libs from OAS
* `Swagger Editor Next (beta)` - next generation editor
* `Swagger Core` - Java related libraries for creating, consuming and working with OpenAPI definitions
* `Swagger Parse` - standalone library for parsing OAS
* `Swagger APIDom` - single unifiying structure for describing APIs across languages

### Benefits of OAS
* You can improve cross-team collaboration by allowing teammates to quickly experiment with endpoints 
  by providing a frontend

* You can quickly get an understanding of endpoints that your teammates have made

* It’s widely used, so you can get an understanding of official APIs that utilise it much faster

* We can generate code from it as it’s machine parseable!

### OAS Example
```yaml
swagger: "3.0"
info:
  version: "1.0"
  title: "Hello World API"
paths:
  /hello/{user}:
    get:
      description: Returns a greeting to the user!
      parameters:
        - name: user
          in: path
          type: string
          required: true
          description: The name of the user to greet.
      responses:
        200:
          description: Returns the greeting.
          schema:
            type: string
        400:
          description: Invalid characters in "user" were provided.
```

## OAS for Rust APIs

### Utoipa
Utoipa is a Rust crate that can be used to decorate and then generate an OpenAPI Spec from your Axum 
based API implementation.

* OpenAPI 3.1 support
* Pluggable, easy setup and integration with Axum
* No bloat, enable what you need
* Support for generic types
* Automatic schema collection
* Various OpenAPI visualization tools

```rust
use std::net::SocketAddr;
use axum::{routing::get, Json};
use utoipa::OpenApi;

#[derive(OpenApi)]
#[openapi(paths(openapi))]
struct ApiDoc;

/// Return JSON version of an OpenAPI schema
#[utoipa::path(
    get,
    path = "/api-docs/openapi.json",
    responses(
        (status = 200, description = "JSON file", body = ())
    )
)]
async fn openapi() -> Json<utoipa::openapi::OpenApi> {
    Json(ApiDoc::openapi())
}

#[tokio::main]
async fn main() {
    let socket_address: SocketAddr = "127.0.0.1:8080".parse().unwrap();
    let listener = tokio::net::TcpListener::bind(socket_address).await.unwrap();

    let app = axum::Router::new().route("/api-docs/openapi.json", get(openapi));

    axum::serve(listener, app.into_make_service())
        .await
        .unwrap()
}
```

* The `ApiDoc` struct is decorated with the `OpenApi` derive macro and sets all the attributes 
  required to serve an OpenAPI Specification directly from your API.
* An attribute macro above the function handler can be used to document the given 
  information about a handler endpoint and show it in the OpenAPI spec when we open it.
* There is a list of responses in the macro with the body left as () as OAS has no specific type
