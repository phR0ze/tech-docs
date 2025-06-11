# JWT <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

JSON Web Tokens (JWT)s are a popular choice for authentication because they are stateless, meaning 
the server doesn't need to store any session information. This makes JWTs a great choice for modern 
Kubernetes or Docker based applications.

### Quick links
* [.. up dir](..)
* [Overview](#overview)
  * [Traditional approach](#traditional-approach)
  * [JWT approach](#jwt-approach)
  * [JWT Benefits](#jwt-benefits)
  * [JWT Composition](#jwt-composition)
* [Client Auth](#client-auth)

## Overview
JSON Web Tokens (JWT) pronounced `jot` are used for Authorization. This means you ware verifying that 
that the user that is sending requests to your server is the same one that logged in during the 
authentication process. JWTs encode claims to be transmitted as a JSON object. The JWT is digitally 
signed and includes a Message Authentication Code (MAC) for integritity and ownership.

**References**
* [JWT RFC](https://www.rfc-editor.org/rfc/rfc7519#section-1)

### Traditional approach
The way this is traditionally done is via sessions. You have a session ID that is stored as a cookie 
and every time the client makes a request they send the session ID to the server. The server then 
finds the user with that session ID and checks to see if the user is authorized for the request.

* Login
  * POST `/user/login { email, password}`
  * Session ID generated on server
  * Session ID sent back to client
  * Client stores session ID as cookie
* Client Requests
  * Pass along the session ID
  * Server checks session ID for authorization

### JWT approach
Instead of the cookie a JSON Web Token is used to do the authorization which provides additional 
user information and is tamper proof.

* Login
  * POST `/user/login { email, password}`
  * JWT generated on server
    * Encodes and signs with its own Private Key
    * Allows for server to know if its been tampered with
    * Nothing is stored on server
  * Client stores JWT as cookie
* Client Requests
  * Pass along the JWT
  * Server checks JWT for authorization

### JWT Benefits
* Solves distribution of authorization tokens such that you don't have to worry about different 
  server instances not having the session ID as JWTs are self-contained.
* Solves integrity issues of session IDs such that the server knows if the token has been tampered 
  with
* Easier to work with as JWTs are self-contained and no DB lookups are required to get user info. 
  that happened during the initial login.

### JWT Composition
JWTs represent a set of claims as a JSON object a.k.a the claims set. The claim set is zero or more 
name/value (claim name/claim value) pairs or members, where the names are strings and the values are 
arbitrary JSON values. There are registered claim names that can be used as a starting point but are 
optional and claims can be defined as needed for your use case. Claim names can be public or private. 
Public names are registered with IANA while private names are simply a well defined spec that server 
and client agree upon. Names are abbreviated to keep with the original goal of JWTs to be compact.

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE3NDk1Mjg4MjksInN1YiI6MX0.dHobiFls_hvkJlFDKOTVCAhBkAuA4ermaoXzf_xYdGo
```

* The first segment up to the first period is the header or algorithm and token type
  * `eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9`
* The second segment from the first period to the second is the payload or claims
  * `eyJleHAiOjE3NDk1Mjg4MjksInN1YiI6MX0`
* The third segment from the last period to the end is the signature to verify
  * `dHobiFls_hvkJlFDKOTVCAhBkAuA4ermaoXzf_xYdGo`

## Client Auth

### Creating JWT tokens
We can use the [jsonwebtoken](https://crates.io/crates/jsonwebtoken) crate to generate JWT tokens.

```rust
/// Generate a JWT token for the given user
/// 
/// - Default expiration is 1 hr
/// - ***secret*** is the JWT private key
/// - ***user_id*** is the ID of the user to include in the token
/// - ***exp*** is the expiration time in seconds from now
pub fn generate_jwt_token(secret: &str, user_id: i64) -> errors::Result<String> {
  let claims = serde_json::json!({
    "sub": user_id,
    "exp": (chrono::Utc::now() + chrono::Duration::seconds(JWT_EXP as i64)).timestamp() as usize,
  });

  let header = jsonwebtoken::Header::default();
  let encoding_key = jsonwebtoken::EncodingKey::from_secret(secret.as_bytes());

  jsonwebtoken::encode(&header, &claims, &encoding_key).map_err(|_| {
    errors::Error::http(axum::http::StatusCode::INTERNAL_SERVER_ERROR, "Failed to generate JWT token")
  })
}

#[cfg(test)]
mod tests {
  use super::*;

  #[test]
  fn test_generate_jwt_token() {
    let jwt = generate_jwt_token("secret", 1).unwrap();
    println!("jwt: {jwt}");
  }
} 
```

The resulting test output can decoded at [jwt.io](https://jwt.io/) to see the content.
```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE3NDk1Mjg4MjksInN1YiI6MX0.dHobiFls_hvkJlFDKOTVCAhBkAuA4ermaoXzf_xYdGo
```

**Header**
```
{
  "typ": "JWT",
  "alg": "HS256"
}
```

**Payload**
```
{
  "exp": 1749528829,
  "sub": 1
}
```

### Verify the signature

