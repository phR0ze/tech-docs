# JWT <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

### Quick links
* [.. up dir](..)
* [Overview](#overview)

## Overview
JSON Web Tokens (JWT) pronounced `jot` are used for Authorization. This means you ware verifying that 
that the user that is sending requests to your server is the same one that logged in during the 
authentication process. JWTs encode claims to be transmitted as a JSON object. The JWT is digitally 
signed and includes a Message Authentication Code (MAC) for integritity and ownership.

**References**
* [JWT RFC](https://www.rfc-editor.org/rfc/rfc7519#section-1)

### Traditional approach
The way this is normally done is via sessions. You have a session ID that is stored as a cookie and 
every time the client makes a request they send the session ID to the server. The server then finds 
the user with that session ID and checks to see if the user is authorized for the request.

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
