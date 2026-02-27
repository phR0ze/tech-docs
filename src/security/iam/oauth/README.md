# OAuth <img style="margin: 6px 13px 0px 0px" align="left" src="../data/images/logo_36x36.png" />

Fundamentally OAuth is a technique that allows applications to communicate with one another without 
the need for sharing the user's credentials. Users can delagate access to one system over to another 
system. Thus you are authorizing one system to act on behalf of you on that other system.

### Quick links
* [.. up dir](..)
* [Overview](#overview)
  * [Terms](#terms)
  * [Client Registration](#client-registration)
  * [Client Credentials flow](#client-credentials-flow)
  * [Authorization Code flow](#authorization-code-flow)
  * [Why auth code](#why-auth-code)
  * [OAuth Scopes](#oauth-scopes)
* [OAuth Core](#oauth-core)
* [OAuth Extensions](#oauth-extensions)
  * [RFC 6749 - PKCE](#rfc-6749-pkce)
  * [RFC 7662 - Token Introspecdtion](#rfc-7662-token-introspection)
  * [RFC 7009 - Token Revocation](#rfc-7009-token-revocation)
  * [RFC 8693 - Token Exchange](#rfc-8693-token-exchange)
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
* `Resource owner` - that's you
* `Resource server` - what you're granting access to
* `Grant type` - how the application is asking for access
* `Scope` - permissions that the app is asking for
* `Authorization server` - who the application is asking for access
* `Access Token` - how the application gets that access
* `Claims` - details on the authorization granted
* `Back channel` - HTTPS from my server to another server with credentials i.e. secure
* `Front channel` - Mobile/browser based apps that can't be shipped with shared secrets

### Client Registration
Before your application can use OAuth 2, you must register with the Authorization Server as a one
time configuration to get their `Client ID` and `Client Secret` which are used to prove the client is
who they say they are. This is done through the Authorization Server's Web Portal or API management
interface. During this process you'll provide:
* `Application Name`: a human-readable name that users will see during authorization
* `Application Website`: your application's homepage or documentation URL
* `Redirect URI (Callback URL)`: the exact URL where the authorization server will send users after
they authorize or deny your application.

### Client Credentials flow
The Client Credentials flow is a back channel only flow that use the client credentials to exchange
for an access token directly with no need for user consent or the auth code step. This is useful for
Machine to Machine (M2M) or service communication. 

### Authorization Code flow
You can authorize the 3rd party app to see for example your contacts in Gmail. This is the most
common OAuth flow and involves the front and back channels both:

1. The app is already registered with and accepted by your IdP
2. You are presented with a button that says e.g. `Connect with Google`
3. When you click on the button you'll be prompted to login to Google if not all ready
4. You'll then be presented with a dialog to consent to the 3rd party app's request for access
5. Once you click yes you'll be redirected back to the app's callback/redirect URI
6. The app will then be allowed the requested access that you consented to
7. You can revoke the app's access at any point

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

### RFC 6749 - PKCE
PKCE, or Proof Key for Code Exchange, is essential for public clients as it protects against
authorization code interception attacks.

**References**
* [IETF RFC 6749](https://datatracker.ietf.org/doc/html/rfc6749)

### RFC 7662 - Token Introspection
* Examines a token to describe its contents
* Useful for opaque tokens
* Describes if the token is active or not
* Mandatory if you have Token Revocation

### RFC 7009 - Token Revocation
* Revokes tokens (optional)

### RFC 8693 - Token Exchange
* Trading or exchanging tokens

### RFC 8414 - Authorization Server Metadata
Optional discovery endpoint that can be queried to find out which extension our implementation 
supports.

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
* Claims are loosely defined but often include:
  * `iss` issuer
  * `sub` subject e.g. `you@gmail.com`
  * `name` name e.g. `Nate Barbettini`
  * `aud` audience e.g. `sdlfkjsdf`,
  * `iat` issued at e.g. `1311281970`
  * `exp` expiration e.g. `1311281970`
  * `auth_time` auth_time e.g. `1311281970`

### OIDC Login example
Client will login with the Authorization server to get an Access token and ID token. The
client will then likely generate a session cookie and store it in the browser to keep track of the
user. And on the back end the client will just hang on to the Access token and the ID token.
