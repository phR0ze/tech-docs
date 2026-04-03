# OAuth <img style="margin: 6px 13px 0px 0px" align="left" src="../data/images/logo_36x36.png" />

Fundamentally OAuth is an open standard for allowing applications to communicate with one another
without the need for sharing the user's credentials. Users can delegate access to one system over to
another system. Thus you are authorizing one system to act on behalf of you on that other system.

OAuth assumes that everything occurs on secure channels i.e. over TLS.

### Quick links
* [.. up dir](..)
* [Overview](#overview)
  * [Terms](#terms)
  * [Four OAuth Roles](#four-oauth-roles)
  * [Client Registration](#client-registration)
  * [Why auth code](#why-auth-code)
  * [OAuth Scopes](#oauth-scopes)
* [OAuth Core](#oauth-core)
  * [Authorization Code](#authorization-code)
  * [Client Credentials](#client-credentials)
  * [Refresh Token](#refresh-token)
  * [Deprecated grants](#deprecated-grants)
  * [Core endpoints](#core-endpoints)
* [OAuth Extensions](#oauth-extensions)
  * [Discovery/Metadata](#discoverymetadata)
  * [RFC 7515 - JWS](#rfc-7515---jws)
  * [RFC 7636 - Authorization Code + PKCE](#rfc-7636---authorization-code-+-pkce)
  * [RFC 8628 - Device Authorization](#rfc-8628---device-authorization)
  * [RFC 7662 - Token Introspecdtion](#rfc-7662-token---introspection)
  * [RFC 7009 - Token Revocation](#rfc-7009---token-revocation)
  * [RFC 7523 - JWT Bearer](#rfc-7523---jwt-bearer)
  * [RFC 7522 - SAML 2.0 Bearer](#rfc-7522---saml-20-bearer)
  * [RFC 8693 - Token Exchange](#rfc-8693---token-exchange)
* [OIDC](#oidc)
  * [JSON Web Token](#json-web-token)
  * [OIDC Login Example](#oidc-login-example)

## Overview
OAuth solves the problem of allowing 3rd party applications access to your data in other
applications. There are a few different flows a.k.a. grant types but the only ones that should really
be used at this point are:
* ***Authorization Code w/PKCE*** for human interaction
* ***Client Credentials*** for machine to machine with no human interaction

**References**
* [OAuth & OIDC explanation](https://www.youtube.com/watch?v=996OiexHze0)
  * [OAuth debugger](https://oauthdebugger.com/)
  * [OIDC debugger](https://oidcdebugger.com/)
    * [Writeup](https://recaffeinate.co/post/introducing-openid-connect-debugger/)
* [JWT Introspection](https://jwt.io)
* [OAuth and OIDC examples](https://github.com/caseysoftware/oauth-and-openid-connect)
* [Okta API Access Manager](https://developer.okta.com)
* [OAuth Playground](https://developers.google.com/oauthplayground)

### Terms
* `Access Token` - how the application gets that access
* `Authorization server` - who the application is asking for access, handles interactive consent
* `Authorization endpoint` - alias for authorization server
* `Back channel` - HTTPS from my server to another server with credentials i.e. secure
* `Claims` - attributes packaged inside a token with credibility from signature
* `Digital Signature` - proves message integrity and ownership using public-key crypto
* `Discovery endpoint` - `/.well-known/openid-configuration` advertises the server's capabilities in JSON
* `Front channel` - Mobile/browser based apps that can't be shipped with shared secrets
* `Grants` - step-by-step procedures a client uses to obtain tokens
* `Resource owner` - that's you
* `Resource server` - what you're granting access to
* `Scope` - permissions that the app is asking for
* `Token endpoint` - machine-to-machine, issues tokens

### Four OAuth Roles

| Role                 | Description               | Example              |
| -------------------- | ------------------------- | -------------------- |
| Resource Owner       | The user                  | Gmail account holder |
| Resource Server      | The API guardian          | Gmail API            |
| Client               | The app requesting access | LinkedIn             |
| Authorization Server | Issues tokens             | Gmail's auth server  |

### Authorization request params
* `response_type=id_token` — I want an ID token
* `response_mode=form_post` — deliver it via HTML form POST (not URL fragment — avoids browser history leaks; avoids size limits)
* `redirect_uri` — exact address to receive the token (must be strict match — loose matching enables token hijacking)
* `scope=openid` profile email — what to include in the token
* `nonce` — a random value saved in a cookie on the client side; the server includes it in the token,
  proving the token is for this request and not injected from a hijacker

### Client Registration
Before an application can use OAuth 2, it must be registered with the Authorization Server as a one
time configuration to get the app's `Client ID` and `Client Secret` which are used to prove the
client is who it claims to be. This is done through the Authorization Server's Web Portal or API
management interface. During this process the app owner needs to provide:
* `Application Name`: a human-readable name that users will see during authorization
* `Application Website`: your application's homepage or documentation URL
* `Redirect URI (Callback URL)`: the exact URL where the authorization server will send users after
they authorize or deny your application.

### Why auth code
The OAuth 2 Authorization code flow returns a code from the Authorization server instead of the
access token directly and requires another step to then get the access token. The reason for this is
that we need to securely deal with front channel (i.e. mobile and web apps). All of the OAuth flow
steps could happen on the front channel and are potentially just page redirects with information
passed along as query parameters of your request. Thus if someone was looking over your shoulder or
you had a malicious toolbar or something they could see the auth code as it is also trasmitted back
to the app via the front channel. However the final exchange mechanism of the auth code for the
access token happens on the back channel over HTTPS and includes a secret key that only the client
and server have via pre-registration.

**Auth code request**
```
https://accounts.google.com/o/oauth2/v2/auth?
  client_id=absc123&
  redirect_uri=https://yelp.com/callback&
  scope=profile&
  response_type=code&
  state=foobar
```

**Auth code response**
```
https://yelp.com/callback?
  code=sdlfkjsdfliujsdlfkjsdl&
  state=foobar
```

**Exchange access token request**
```
POST www.googleapis.com/oauth2/v4/token
Content-Type: application/x-www-form-urlencoded

code=sdlfkjsdfliujsdlfkjsdl&
client_id=absc123&
client_secret=secret123&
grant_type=authorization_code
```

**Exchange access token response**
```json
{
  "access_token": "secret-access-token",
  "expires_in": 3920,
  "token_type": "Bearer"
}
```

**Access request**
```
GET api.google.com/some/endpoint
Authorization: Bearer secret-access-token
```

### OAuth Scopes
As simple or complex as you want.  Better to stay simple and increase it later. There is no unified 
approach and the scope strings are free form and can be any format. Best to unify your own.

* github example: `repo`, `public_repo`, `repo:invite`, `write:repo_hook`
* google example: `Analytics API: https://www.googleapois.com/auth/analytics`
* okta example: `okta.apps.manage`, `okta.apps.read`
* Example: create, read, update, delete

## OAuth Core
[IETF RFC 6749](https://datatracker.ietf.org/doc/html/rfc6749) defines the OAuth 2.0 core grants.
OAuth is based on the `Grant Type` primitive. A grant is the step-by-step procedures a client uses to
obtain tokens for a given scenario.

### Authorization Code
You can authorize the 3rd party app to see for example your contacts in Gmail. This is the most
common OAuth flow and involves the front and back channels both:

1. The app is already registered with and accepted by your IdP
2. The app presents you with a button that says e.g. `Connect with Google`
3. When you click on the button:
   1. The app will redirect you to Google's authorization server
   2. You'll be prompted to login to Google if not all ready
   3. You'll be presented with a dialog to consent to the 3rd party app's request for access
   4. Once you click yes you'll be redirected back to the app's callback/redirect URI
4. Your browser will deliver the code in memory back to the app
5. The app will then present the code + client credentials to the Google token endpoint
6. The Google Authorization server will then issue an access token 
7. The app will then be able to present the access token to Google APIs for access
8. You can revoke the app's access at any point

### Client Credentials
The Client Credentials grant is a back channel only flow that use the client credentials to exchange
for an access token directly with no need for user consent or the auth code step. This is useful for
Machine to Machine (M2M) or service communication. 

### Refresh Token
Refresh tokens are defined in RFC 6749 (the core OAuth 2.0 spec) and are not a separate grant type
but does use a grant type value `grant_type=refresh_token`

1. Client sends the `refresh_token` to the token endpoint
2. AS issues a new access token (and optionall a new refresh token)
3. Old refresh token is invalidated (rotation is recommended, required in OAuth 2.1)

### Deprecated grants
* `Implicit` - originally for SPAs this is now deprecated
* `Resource owner password credentials (ROPC)` - provides user/pass directly to client and is deprecatd

### Core endpoints
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

## OAuth Extensions
OAuth is a loose operating agreement, not a contract. It leaves a lot of things undefined on purpose. 
Extensions were added to define additional functionality on top of the existing OAuth standard so
that this new functionality would be built in a similar way across the industry and stop the
fragmentation.

Some of the extensions define new structures and processes and some provide new Grant types.

***Modern Use***
* `Authorization Code + PKCE` for web apps, spas, mobile
* `Client Credentials` for M2M or services
* `Device Authorization`  for devices
* Avoid the rest as they are deprecated in OAuth 2.1

### Discovery/Metadata
These endpoints let clients auto-discover entpoints, supported grant types, scopes, etc... without
hardcoding URLs. They are the foundation for OpenID Connect Discovery.

***WARNING*** the root location e.g. `/.well-known...` is required to adhere to the specification.
Clients will take the issuer URL and append the well-known path e.g.
`https://iam.example.com/.well-known/oauth-authorization-server`. RFC 8615 defines `/.well-known` as
the prefix for site-wide metadata across many protocols (not just OAuth2).

#### Authorization Server Metadata - RFC 8414
Authorization Server Metadata lets clients auto-discover entpoints, supported grant types, scopes,
etc... without hardcoding URLs.

```bash
curl -s localhost:8080/.well-known/oauth-authorization-server
```

#### JSON Web Key Set - RFC 7517
The JSON Web Key Set publishes the server's public signing keys so resource servers can verify JWTs
without calling back to the auth server.

```bash
curl -s localhost:8080/.well-known/jwks.json
```

### RFC 7515 - JWS
JSON Web Signature (JWS) is a standard for respresenting content secured with a digital signature or
MAC (Message Authentication Code) encoded as a compact, URL-safe string. A JWS has three
`Base64url-encoded` parts separated by dots e.g. `header.payload.signature`
* `Header` is the algorithm used e.g. `RS256`, `HS256`, `ES256` and is included in the signing
* `Payload` is the data being signed (arbitrary bytes)
* `Signature` is the cryptographic signature over `header.payload`

JWS is the mechanism that makes JWTs trustworthy. JWT is essentially a JWS with special payload data
i.e. the claims or in other words a JWT is JSON claims encoded as a JWS.

* Access tokens are often JWTs
* ID tokens in OIDC are always JWTs

### RFC 7636 - Authorization Code + PKCE
PKCE, or Proof Key for Code Exchange, is essential for public clients as it protects against
authorization code interception attacks. It is used by SPAs and mobile apps use this `grant` to
replace the deprecated `Implicit` grant.

### RFC 8628 - Device Authorization
For devices with no browser like (smart TVs, CLIs)

### RFC 7662 - Token Introspection
* Examines a token to describe its contents
* Useful for opaque tokens
* Describes if the token is active or not
* Mandatory if you have Token Revocation

### RFC 7009 - Token Revocation
* Revokes tokens (optional)

### RFC 7523 - JWT Bearer
Client asserts identity via signed JWT instead of redirect

### RFC 7522 - SAML 2.0 Bearer
Federation with SAML identity providers

### RFC 8693 - Token Exchange
Impersonation / delegation between services

## OIDC
OIDC fundamentally just provides an ID token (i.e. a JWT that encodes information about the user).
The OAuth flow is identical to standard OAuth but now includes the `openid` scope request and you
also get back in addition to the access token the ID token. Because OIDC is just an extension on top
of OAuth 2 you will see many homegrown OIDC like implementations that do something similar to OIDC
using OAuth but might not actually be fully OIDC compatible. This gives rise to the confusion that
OAuth can do authentication because everyone hacked their own solution together for this or have
converted to the standard OIDC to solve this.

It is likely the most important OAuth extension that:
* Provides a standard way to get user info via the `/userinfo` endpoint
  * get additional meta data e.g. profile picture
* Is the basis for `Sign in with` on hundreds of sites
* Depends on JSON Web Tokens or JWTs to encode user information
* Doesn't support client credential flow i.e. only works with users
  * i.e. no machine to machine support

### JSON Web Token
JWTs are an extension to OAuth popularized by the OIDC extension. JWTs provide a mechanism for
encoding information typically about the user. This information called claims is used together with
a signature that can be validated to ensure nothing has been tampered with.

* OAuth doesn't require JWTs but they're common
* JWTs are encoded, not encrypted
* Composed of `header.claims.signature`
* Header:
  * `alg` signing algorithm
  * `kid` key identifier (used to find the right public key)
* Payload claims are loosly defined by often include:
  * `iss` issuer - who signed the token
  * `aud` audience (which app this token is for, i.e. must match your client ID) e.g. `sdlfkjsdf`,
  * `sub` subject (user identifier) e.g. `you@gmail.com`
  * `iat` issued at e.g. `1311281970`
  * `exp` expiration e.g. `1311281970`
  * `auth_time` auth_time e.g. `1311281970`
  * Identity claims
    * `name` name e.g. `Nate Barbettini`
    * `email` email e.g. `you@gmail.com`
    * `picture` picture

#### Recommended JWT checks
1. Verify the signature
2. Check `iss` matches expected user
3. Check `aud` matches your client id
4. Check `exp` token is not expired
5. Check `nonce` matches what you sent

#### Alternative JWT check
Send it to the authorization server's introspection endpoint and ask if it is valid. The server would
then respond with validity + claims, but this introduces:
* an extra network hop
* availability dependency
* potential throttling

The recommendation is to prefer local JWT validation

### OIDC Login example
1. Browser hits protected route
2. Middleware -> 302 redirects to authorization endpoint
3. Browser hits authorization endpoint
4. Auth server authenticates the user, sets its own session cookie
5. Auth server returns HTML pages with `<form` auto-submitted via JS
6. Form POSTS ID token to the app's `redirect_uri`
7. App validates token, sets session cookie, 302 redirects to original route
8. Browser accesses protected route with cookie -> 200 OK
