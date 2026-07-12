# BFF <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

***Problem Statement*** - Building a frontend application from a collection of microservices requires
the frontend to completely understand the backend APIs. Another issue is the frontend app will need
to be updated whenever an underlying API changes. Additionally the data coming back from the
microservices might not be formatted to fit how the UI needs to consume it. Thus the UI needs to have
additional logic to handle this directly making it more complicated.

***Backend for frontend (BFF)*** is an architectural pattern designed to solve these issues. It is a
custom API that fits between the frontend application and the backend application. That potentially
orchestrates calls, structures and formats the output for the UI to reduce complexity in the
frontend. It is focused on a single UI and only that UI so you can have custom calls and logic just
for that UI supported in the BFF that isn't required in the underlying public API.

### Quick links
- [.. up dir](../README.md)
- [Overview](#overview)

## Overview
Next.js consists of a client side as well as a server side. The server side can be used for server
side rendering, server actions or a BFF API. It gets deployed together as a Next.js app. These
functions can be used or not used as needed. For example you could have a Next.js app that only makes
use of the client components and the server side rendering and the server actions or client
components may call out to separate backend APIs directly or you might formalize this in the Next.js
server side to create a BFF that is used per the introduction of this doc to simplify the UI, format
backend data for the UI and have a single clean public API exposed. Benefits of the BFF architecture:

* Scale the Next app independent from the backend
* Scale the backend independent from the next app
* Clean unified public API so you don't expose the backend API
* Orchestration of backend api calls and surfacing a simpler interface to UI
* Custom data structures or formatting for the UI
* Ability to build in separate languages
* Reduces UI complexity

**References**
* [Backend for Frontend - Auth0](https://auth0.com/blog/the-backend-for-frontend-pattern-bff/)
* [Origins of BFF](https://philcalcado.com/2015/09/18/the_back_end_for_front_end_pattern_bff.html)

### MPA vs SPA vs SSR
* Multi-page apps based on static generated html pages
  - fastest load
  - experience controlled by browser and can be jarring
* Single page apps
  - user's device is responsible for loading artifacts
  - javascript on user's device renders the page
  - javascript on user's device loads data
  - see considerable jank as page continues to load piece meal
* Server Side Rendering
  - rendered on the server and pulls the data in advance so user sees finished results
  - javascript still needs to load for intactivity
  - slightly longer load time than SPA but solves the jank issues

**References**
* [Do you really need SSR - Youtube](https://www.youtube.com/watch?v=kUs-fH1k-aM)

### Server Side Rendering
Client Side Rendering (CSR) suffers from two issues. Search Engine Optimization (SEO) struggles as
the SPA pages require Javascript execution in the browser to render and thuse SEO fails. Additionally
load times can be high as the user browser's JavaScript engine, which might be on a small phone, is
responsible for rendering and might take a while. Modern React frameworks like Next.js provide a
server component for server side rendering (SSR). Then the fulling formed html pages are loaded in
the browser. This allows for SEO to easily index the server-rendered content and improves initial
load times for slow devices. Loading the SSR pages and Javascript bundles together is called
Hydration.

### React Server Components
React server components are the magic behind the new granular SSR and make all the difference. Most
apps should just default to SSR.
