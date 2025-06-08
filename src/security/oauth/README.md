# OAuth <img style="margin: 6px 13px 0px 0px" align="left" src="../data/images/logo_36x36.png" />

Fundamentally OAuth is a technique that allows applications to communicate with one another without 
the need for sharing the user's credentials. Users can delagate access to one system over to another 
system. Thus you are authorizing one system to act on behalf of you on that other system.

### Quick links
* [.. up dir](..)
* [Overview](#overview)
  * [Terms](#terms)
  * [JSON Web Token](#json-web-token)
  * [RFC 7662 Token Introspecdtion](#rfc-7662-token-introspection)
  * [RFC 7009 Token Revocation](#rfc-7009-token-revocation)
  * [RFC 8693 Token Exchange](#rfc-8693-token-exchange)
  * [RFC 8414 Authorization Server Metadata](#rfc-8414-authorization-server-metadata)
* [OAuth Core](#oauth-core)

## Overview
OAuth is a loose operating agreement, not a contract. It leaves a lot of things undefined on purpose. 
This is where extensions come in.

**References**
* [JWT Introspection](https://jwt.io)
* [OAuth and OIDC examples](https://github.com/caseysoftware/oauth-and-openid-connect)
* [Okta API Access Manager](https://developer.okta.com)
* [OAuth Playground](https://developers.google.com/oauthplayground)
* [OAuth and Ruby](https://medium.com/@hashimmazhar15/oauth-implementing-oauth-in-ruby-on-rails-using-doorkeeper-and-oauth2-gems-2490162b9cac)

### Terms
* Resource owner - that's you
* Resource server - what you're granting access to
* Grant type - how the application is asking for access
* Scope - permissions that the app is asking for
* Authorization server - who the application is asking for access
* Token - how the application gets that access
* Claims - details on the authorization granted

### JSON Web Token
JWTs are an extension to OAuth

* OAuth doesn't require JWTs but they're common
* JWTs are encoded, not encrypted
* JSON Web Encryption (JWE)
* Includes `iss` issuer, `iat` issued at, `sub` subject, `aud` audience, and `exp` expiration

### RFC 7662 Token Introspection
* Examines a token to describe its contents
* Useful for opaque tokens
* Describes if the token is active or not
* Mandatory if you have Token Revocation

### RFC 7009 Token Revocation
* Revokes tokens (optional)

### RFC 8693 Token Exchange
* Trading or exchanging tokens

### RFC 8414 Authorization Server Metadata
Optional discovery endpoint that can be queried to find out which extension our implementation 
supports.

### HTTP Redirects

### Secure browser storage

### OIDC
OpenID Connect is the most important OAuth extension that gives us a user's profile information.
* Provides a standard way to request and share profile data
* Gives us a `Sign in with` on hundreds of sites
* Depends on JSON Web Tokens or JWTs
* Doesn't support client credential flow i.e. only works with users
  * i.e. no machine to machine support

## OAuth Core
Only the `/authorize` and `/token` endpoints are required. All the other ones are optional. So do you 
support OAuth can mean different things.

***/authorize***
* the endpoint that the end user (resource owner) interacts with to grant permission for the 
  application to access the resource
* could return an authorization code or an access token

***/token***
* endpoint that the app uses to trade an authorization code or refresh token for an access token

***/userinfo***
* endpoint that apps use to retrieve profile info about the authenticated user supported by OIDC
* returns a spec defined set of fields, i.e. varies

***/.well-known/oauth-authorization-server***
* endpoint for apps to use to retrieve configuration info for the auth server
* gives all urls and all capabilities of the auth server

***/introspect***
* endpoint for apps to learn more about a token
  * revoked, expired etc...

***/revoke***
* endpoint for apps to deactivate or invalidate

### Grant Types or OAuth flows
There are a number of grant types that solve different use cases:
* Authorization Code - server only
* Implicit or Hybrid - don't use b/c weaknesses
* Resource Owner Password - legacy, transitional use case
* Client Credentials - back end systems without user interaction
* Device Code - dumb devices like terminals
* Authorization Code with PKCE

**Grant type to use**
* ***Human*** => ***Authorization Code w/PKCE***
* ***No human*** => ***Client Credentials***

### OAuth Scopes
As simple or complex as you want.  Better to stay simple and increase it later. There is no unified 
approach and the scope strings are free form and can be any format. Best to unify your own.

* github example: `repo`, `public_repo`, `repo:invite`, `write:repo_hook`
* google example: `Analytics API: https://www.googleapois.com/auth/analytics`
* okta example: `okta.apps.manage`, `okta.apps.read`
* Example: create, read, update, delete

## OAuth with client
Many apps today are actually just a front-end for a series of API calls. Weather its a Web app, 
desktop client or mobile client they are simply calling the backend API directly to store and 
interact with their data. The most common modern solution for security in this case is OAuth2.

Typicall the ***Authorization Code w/PKCE*** grant is used.

