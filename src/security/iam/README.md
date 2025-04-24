# IAM <img style="margin: 6px 13px 0px 0px" align="left" src="../data/images/logo_36x36.png" />

Identity and Access Management

### Quick links
* [.. up dir](..)

## Overview
The three core concepts of Identity and Access Managment are: `authentication`, `authorization` and 
`accounting`.

* `Authentication (Authn)` - establish who you are
  * Credentials, JWT, API Key
  * Something you know and something you have (token)
* `Authorization (Authz)` - establish what you can do
  * Permissions for resources
* `Accounting` - measuring the time and resources they use
  * Auditing

* Components
  * ***User*** - user account
  * ***Agent*** - entity operating on your behalf
  * ***Group*** - more than one user, agent, device or account with same rights
  * ***Roles*** - set of permissions that can be applied to a user or group
  * ***Policy*** - policy based permissions
  * ***Device*** - device
  * ***Account*** - automated internal operation
  * ***Access Control***
    - MAC (Mandatory Access Control)
    - DAC (Discretionary Access Control)
    - RBAC (Role Based Access Control)
  * ***RBAC*** - give permissions based on role
  * ***Federation*** - a collection of domains that have established trust
  * ***Least priviledge*** - minimum necessary permission to do job
  * ***Provisioning*** - is the act of creating, updating, and deleting information for user
  * ***Conditional Access*** - provided when needed
  * ***Just-in-Time Access*** - provide access for a window of time

### Identity Provider
Identity providers execute authentication. IdP can create store and management identities. A user 
will send credentials to the IdP and the IdP will send back a token.

1. Service Provider sends token request to Identity Provider
2. Identity Provider will then send back a token to be passed through

### Identity Governance
A process for manageing access to resources via a centralized solution. The Identity Governance 
Process confirms authentication process has taken place by checking the user's token that was 
provided by the IdP.

### Single Sign-On SSO
The ability to sign on a single time for multiple services.

### Protocols
Federated Identity Management (FIM) is accomplished through 3 primary protocols:
* ***SAML*** - Security Assertion Markup Language
  * XML based, old
* ***OAuth2.0*** - requests authentication without sharing passwords
  * JSON based token sharing, modern
  * Shares just the token
* ***OIDC*** - OpenID Connect
  * Protocol that extends OAuth2.0 with additional functions
  * Adds the ability to share account data like user profile

### Cloud Identity Providers
* Facebook
* Google
* Twitter
* Auth0
* AWS
* Okta
