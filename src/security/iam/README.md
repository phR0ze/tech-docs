# IAM <img style="margin: 6px 13px 0px 0px" align="left" src="../data/images/logo_36x36.png" />

An Identity and Access Management (IAM) provider is a system that ensures secure, controlled, easy 
access to resources.

### Quick links
* [.. up dir](..)
* [Overview](#overview)
  * [Auth0 case study](#auth0-case-study)
* [Self-hosted options](#self-hosted-options)
  * [Golang library support](#golang-library-support)
    * [Casdoor](#casdoor)
    * [Zitadel](#zitadel)
    * [Authelia](#authelia)
  * [Python](#python)
  * [Script](#script)
    * [logto](#logto)
  * [Java](#java)

### Linked pages
* [JWT](jwt/README.md)
* [MFA](mfa/README.md)
* [OAuth](oauth/README.md)

## Overview

**References**
* [IAM - medium](https://logto.medium.com/top-5-open-source-identity-and-access-management-iam-providers-2025-ef2428c01c6e)
* [Awesome IAM Knowledge docs](https://github.com/kdeldycke/awesome-iam)

IAM is composed of the following segments:
* `Governance`
  * `User Management`
    - provisioning
    - roles
    - audits
    - measuring the time and resources users use
  * `Organizational Management`
    - structuring teams
    - multitenancy

* `Authentication (Authn)`
  - establish who you are
  - Logins, passwords, biometrics, social login
  - Something you know and something you have (token)

* `Authorization (Authz)`
  - establish what you can do
  - Roles
  - permissions for resources

* `Audit`
  - Audit trails of user activity

### Auth0 case study
***Auth0***, founded in 2013, is the defacto standard for modern B2B2C SaaS IAM solutions that every 
startup is comparing themselves to. It is by far the most well known option in teh modern IAM space. 
So much so that Auth0 was acquired by Okta in 2022 and has since been subsumed into Okta's processes 
and lost its original developer focused flare.

Other notable paid clones include:
* ***Frontegg***, ***FusionAuth***, ***Descope***, ***WorkOS***, ***Stytch***, and ***SSOJet***

All the paid clones and many open source projects have tried to clone or are heavily inspired by 
Auth0, which makes it an interesting case study. What is it that Auth0 did that was so successful? 
Auth0 provided a B2B2C SaaS model that fits well with modern application design.

They built an intuitive API and Web portal to manage the IAM

### Interface design

You have a similar management interface with control options on the left 
such as `Applications`, `Authentication`, `Organizations`, `User Management`, `Branding`, `Security`, 
`Actions`, `Auth Pipeline`, `Monitoring`, `Marketplace`, `Extensions` and `Settings`

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
* Identity Provider
Identity providers execute authentication. IdP can create store and management identities. A user 
will send credentials to the IdP and the IdP will send back a token.

1. Service Provider sends token request to Identity Provider
2. Identity Provider will then send back a token to be passed through

**Common IdPs**
* Facebook
* Google
* Auth0
* AWS
* Okta

* Identity Governance
A process for manageing access to resources via a centralized solution. The Identity Governance 
Process confirms authentication process has taken place by checking the user's token that was 
provided by the IdP.

## Workfoce IdPs

### Single Sing-On SSO
The ability to sign on a single time for multiple services.

Federated Identity Management (FIM) is accomplished through 3 primary protocols:

**Protocols**

* ***SAML*** - Security Assertion Markup Language
  * Verable, widely used, xml based and old

* ***OAuth2.0*** - requests authentication without sharing passwords
  * JSON based token sharing, modern
  * Shares just the token

* ***OIDC*** - OpenID Connect
  * Protocol that extends OAuth2.0 with additional functions
  * Adds the ability to share account data like user profile

* ***SCIM*** - 

### Key considerations
* Authentication (Connectors)
  - Password
  - SSO (OAuth2, OIDC, SAM)
    - Okta, Azure, Google
  - Personal access tokens
  - OAuth consent flows
  - Email and SMS passwordless
  - Social logins: Google, Facebook, Microsoft, Apple, Github, X.com, LInkedIn,
  - Biometric
  - Machine to Machine (M2M)
* Authorization (Roles)
  - User/Organizational RBAC
  - ABAC
  - JWT/opaque token validation
  - API protection
* User Management
  - Users
    - Create
    - Reset
    - Member invitations
    - User migration
    - JIT user provisioning
    - Impersonation
    - Suspend
  - Organizations
    - User federation
    - Multi-tenancy
    - Structured teams
* Security and Management
  - MFA (TOTP, Email, SMS, backup codes)
    - Email: AWS, Mailgun, Postmark, SendGrid
  - Audit logs
    - measuring time and resources used
  - Webhooks
  - Compliance reporting
  - Encryption
  - Policy
  - Brute-force protection
  - Bot detection
  - Blocklists
  - SOC2/GDPR
* User experience
  - prebuilt auth flows (login, registration, password reset)
  - intuitive user flows
  - mobile friendly
  - customizable
* Developer experience
  * Customization and extensibility
    - ui themes
    - styling and colors
  - SDKs and Deployment flexibility
  - comprehensive docs
  - code samples
  - sandbox environments
  - low-code admin consoles
  - thriving dev community (discord, github)
  - enterprise support options
  - regular updates for zero-day vulnerabilities

## Self-hosted options
IAM solutions in this space sit in front of NextCloud, Grafana and other self hosted apps providing 
authentication and authorization. They add login, 2FA, access controls and other options but take 
different approaches.

| Feature         | Authelia          | Authentik           | 
| --------------- | ----------------- | ------------------- | 
| UI              | No                | Typescript          |
| MFA             | TOTP, WebAuthn    | TOTP, WebAuthn      |
| Resource Usage  | 30Mb RAM, 0.5 CPU | 2GB Ram, 2 CPU      |
| Setup           |                   |                     |
| Protocols       | LDAP, OIDC        | OAuth2/OIDC, SAML2, SCIM |

### Golang
`Golang` seems to dominate the open source IAM space with 3 of the top contenders being written in 
Golang. Typescript, Java and Python all have a single offering each as well.

| Library                                 | Purpose         | Projects using it
| --------------------------------------- | --------------- | -------------------------------
| `github.com/riverqueue/river`           | Background Jobs | Zitadel, 
| `github.com/dchest/captcha`             | Captcha         | Casdoor,
| `github.com/emirpasic/gods`             | Data Structures | Casdoor,
| `github.com/casdoor/gomail`             | Email           | Casdoor,
| `github.com/wneessen/go-mail`           | Email           | Authelia, 
| `github.com/ory/mail`                   | Email           | Kratos, 
| `github.com/sendgrid/sendgrid-go`       | Email/Sendgrid  | Casdoor,
| `github.com/go-ldap/ldap/v3`            | LDAP            | Casdoor, Zitadel, Authelia, 
| `github.com/golang-jwt/jwt/v5`          | JWT             | Casdoor, Zitadel, Authelia, Kratos
| `github.com/casdoor/notify`             | Notifications   | Casdoor,
| `github.com/nyaruka/phonenumbers`       | Phone Numbers   | Casdoor, Kratos, 
| `github.com/ttacon/libphonenumber`      | Phone Numbers   | Zitadel,
| `github.com/prometheus/client_golang`   | Prometheus      | Authelia, 
| `golang.org/x/oauth2`                   | OAuth2          | Casdoor, Zitadel, 
| `github.com/coreos/go-oidc/v3`          | OIDC            | Kratos
| `github.com/ory/go-oidc`                | OIDC            | Kratos
| `github.com/authelia/oauth2-provider`   | OAuth2/OIDC     | Authelia,
| `github.com/opentracing/opentracing-go` | Open Tracing    | Casdoor,
| `github.com/redis/go-redis/v9`          | Redis           | Authelia, 
| `github.com/casdoor/go-sms-sender`      | SMS             | Casdoor,
| `github.com/elimity-com/scim`           | SCIM            | Casdoor, 
| `github.com/zitadel/saml`               | SAML server     | Zitadel, 
| `github.com/crewjam/saml`               | SAML            | Zitadel, 
| `github.com/russellhaering/gosaml2`     | SAML2           | Casdoor,
| `github.com/slack-go/slack`             | Slack           | Casdoor, Kratos, 
| `github.com/twilio/twilio-go`           | Twilio          | Casdoor, Zitadel, 
| `github.com/authelia/otp`               | TOTP            | Authelia
| `github.com/pquerna/otp`                | TOTP            | Kratos, 
| `github.com/google/uuid`                | UUID            | Casdoor, Zitadel, Authelia, Kratos
| `github.com/go-webauthn/webauthn`       | WebAuthn        | Casdoor, Zitadel, Authelia, Kratos
| `github.com/russellhaering/goxmldsig`   | XML Signatures  | Casdoor, Zitadel, 

#### Casdoor
[Casdoor](https://casdoor.org/)
* Features
  * `Go 1.21` [Casdoor Github](https://github.com/casdoor/casdoor)
    * 11.4k stars, 1.3k forks, recent commits
  * Open Source UI-first IAM solution
    * Web UI feels dated
  * OAuth2, OIDC, SAML, SCIM, CAS, LDAP, WebAuthn, TOTP, MFA, RADIUS
  * SQL injection (CVE-2022–24124)

#### Zitadel
[Zitadel](https://zitadel.com/)
[Zitadel](https://github.com/zitadel/zitadel), 10.9 stars, 699 forks, recent commits
* [Zitadel docs](https://zitadel.com/docs)
* Features
  * Protocols: OAuth2/OIDC, SAML2, LDAP, Passkeys, SCIM 2
  * `Go 1.23.7` as a single binary, `120 MB RAM` idle, 
  * Console is a Dashboard UI
    * Can be used by users to modify profile, but recommend building your own UI
  * `Audit trail`
  * `Event API for changes`
  * Role based access control
  * Cloud native (modern, scalable), API first
  * Strong focus on `multi-tenancy`
    * Ideal for B2B where you manage multiple organziations
* Client SDKs
  * Go, Node.js, Next.js, NestJS
* Deployment
  * [Zitadel Chart](https://github.com/zitadel/zitadel-charts), 93 stars, 68 forks, recent commits
  * Docker compose

#### Authelia
[Authelia](https://github.com/authelia/authelia), 23.6k stars, 1.2k forks, recent commits

* Cons
  * Not a full fledged IAM solution
  * No UI
* Written in `Go 1.24.0` and `React` and is extremely light weight and fast
  * Tiny `20mb container siz`e and `tiny memory consumption at 30mb`
* Features
  * Login Regulation
  * Password reset
  * MFA via TOTP, WebAuthn
* Has a very clean minimal interface, acting more like a reverse proxy companion
* yaml configuration files
  * Gives a lot of granular control
  * Not for beginers
* Policies are define in code
* Works with Nginx, Traffik and Caddy
* 2FA, TOTP
* Perfect for full control and don't mind getting hands dirty
* Smart doorman for existing stack
* Deployment
  * [Authelia Chart](https://charts.authelia.com/)
  * Docker compose
  * [Authelia Istio integration](https://www.authelia.com/integration/kubernetes/istio/)

#### Ory Kratos
Kratos is the Ory Identity Server while Hydara is the Ory OIDC server.
* [Ory Kratos Github](https://github.com/ory/kratos)
* [Ory Hydra Github](https://github.com/ory/hydra)
* [Ory paid option](https://www.ory.sh/network/)
* [Ory Helm charts](https://k8s.ory.sh/helm/)
* `Go 1.24.1`
* Identity and credential management scaling to billions of users
* Registration, Login and Account management flows for:
  * Passkeys, Biometric, Social, SSO and MFA
* OAuth2, OIDC
* GDPR friendly

### Python

#### Authentik
[Authentik](https://github.com/goauthentik/authentik), 15.9k stars, 1.1k forks, recent commits.

* Full web UI for setup and management
  * Beautiful admin dashboard
  * Easily crate users, groups and flows
* Smaller community as its newer
* Protocols
* Supports SSO, OAuth2, SAML, IdP for other apps
  * LDAP and SCIM for enterprise
* Totally open source and self-hosted
  * `Python` and `Typescript` which can be resource heavy
* [Features comparison](https://goauthentik.io/#comparison) 
* Deployment
  * [Helm Chart](https://github.com/goauthentik/helm), 117 stars, 50 forks, committed to 8 hrs ago
  * Docker Compose

### Script

#### logto
Logto is an open-source alternative to Auth0 and an OIDC (OAuth 2.0) provider designed for modern 
applications and SaaS products, supporting both authentication and authorization. SAML is also 
supported.

**References**
* [Logto - youtube](https://www.youtube.com/@logto-io)
* [Logto - Elestio](https://www.youtube.com/watch?v=h-p2L8z6XEM)
* [Logto - Organizations](https://www.youtube.com/watch?v=XDRGFbhWvm8)
* [Logto - RBAC](https://www.youtube.com/watch?v=Lwnk-pE_uYQ)
* [Logto - Enterprise SSO](https://youtu.be/-mD8Sfab7sI?si=6u4dJhoLrzi6Eg9g)
* [Logto Github](https://github.com/logto-io/logto)

* Impressions
  * Features rich

* Features
  * Age: modern, 10k stars, 532 forks
  * Protocols: OIDC, OAuth2, SAML
  * SDKs: Android, Angular, Express, Flutter, Go, Next.js, Auth.js, Python, Ruby, etc …
  * Authn: Password, Email/SMS passwordless, social logins, enterprise SSO, MFA (TOTP, backup codes, personal access tokens, OAuth consent flows)
  * Authz
    - API protection
    - RBAC: provides the ability to create roles
    - JWT
  * Multi-tenancy
    - user organizations
      - specific groups and roles
    - member invites
    - per-organization MFA
    - JIT provisioning
    - tailored sign-in
  * User Mgmt
    - provisioning
    - impersonation
    - invitations
    - suspend
    - auditing: logs per user
    - migration
  * User Experience
    - beautiful out-of-the-box
    - fully customizable auth flows
    - enabling a unified multi-app omni-login via federated identity
  * Social providers: Google, Facebook, Microsoft, Apple, Github, X.com, LinkedIn, Slack, etc …
  * Enterprise providers: Azure, Google, Okta, etc…
  * Email delivery: AWS, Mailgun, Postmark, SenGrid etc…
  * SMS delivery
    - Twillio, SMS Aero, GatewayAPI etc…
  * Deployment:
    - Kubernetes chart or docker compose
    - Resources, 2 vCPU, 8 GB RAM
  * Customizable
    * With branding and colors
  * Auth flow
    * Redirects to your Logto instance
    * Once signed in redirects back to your app

#### NextAuth
A lightweight auth library tailored for Next.js devs providing social login, passwordless auth, and 
session management

### Java

#### Open Identity Platform
[Open Identity Platform](https://github.com/OpenIdentityPlatform)

#### CAS
[Apereo CAS](https://github.com/apereo/cas) providing SAML2, OIDC, MFA and more

#### Gluu
[Gluu](https://github.com/GluuFederation) is another Java offering that still receives a fair amount 
of attention.

#### Super Tokens
[Super tokens](https://github.com/supertokens/supertokens-core)

#### Keycloak
[Keycloak](https://github.com/keycloak/keycloak), 26.9k stars, 7.2k forks, recent commits
* `Java` code base, backed by Redhat
* Supports everything
  * All the protocols
  * B2B and B2C use cases
* Features
  * User federation
  * User management
  * User permissions

