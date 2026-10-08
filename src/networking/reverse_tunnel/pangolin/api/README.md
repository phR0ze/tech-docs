# Using the Pangolin API <img style="margin: 6px 13px 0px 0px" align="left" src="../../../../data/images/logo_36x36.png" />

Some [Pangolin](../README.md) settings aren't in the dashboard yet even though the backend fully
supports them - e.g. `tlsServerName`/`setHostHeader` on a Private HTTP resource (see the
[Vaultwarden Example](../vault_example/README.md)), or creating a site with a Newt ID and secret you
choose instead of generated ones. For those, Pangolin's REST API is the way in - the same actions
the dashboard calls, just addressable directly.

This page assumes the NixOS deployment from the main doc (`services.oci.pangolin`), where Pangolin's
`config.yml` and compose file are rendered by the module rather than edited by hand.

### Quick links
- [.. up dir](..)
- [Two separate API servers - don't confuse them](#two-separate-api-servers---dont-confuse-them)
- [Enable it temporarily](#enable-it-temporarily)
- [Reach it without publishing anything](#reach-it-without-publishing-anything)
- [Create a scoped org API key](#create-a-scoped-org-api-key)
- [Authenticate requests](#authenticate-requests)
- [Finding IDs you'll need](#finding-ids-youll-need)
- [Example: create a site with your own Newt credentials](#example-create-a-site-with-your-own-newt-credentials)
- [Turn it back off](#turn-it-back-off)

## Two separate API servers - don't confuse them
Checked against the upstream source (`fosrl/pangolin`): Pangolin runs *two* independent HTTP
servers, and only one of them accepts an API key. Hitting the wrong one is why a Bearer-token `curl`
against `https://pangolin.example.com/api/v1/...` comes back `{"message":"Unauthorized","status":401}`
even with a valid key - that generic message (not `"Invalid API key"`) comes from
[`verifySession`](https://github.com/fosrl/pangolin/blob/main/server/middlewares/verifySession.ts)'s
session-cookie check; the request never reached API-key code at all.

| | **Dashboard API** | **Integration API** (the one you want) |
|---|---|---|
| Source | [`server/apiServer.ts`](https://github.com/fosrl/pangolin/blob/main/server/apiServer.ts) | [`server/integrationApiServer.ts`](https://github.com/fosrl/pangolin/blob/main/server/integrationApiServer.ts) |
| Auth | Session cookie only - a Bearer header is silently ignored | `Authorization: Bearer <keyId>.<keySecret>` ([`verifyApiKey`](https://github.com/fosrl/pangolin/blob/main/server/middlewares/integration/verifyApiKey.ts)) |
| Path prefix | `/api/v1/...` | `/v1/...` (**no** `/api`) |
| Port (in the container) | `3000` | `server.integration_port`, default `3003` |
| Reachable via | The public dashboard domain, through the module's `api-router` → `http://pangolin:3000` | **Nothing** - not started unless enabled, not published, not routed by Traefik |
| Enabled by | Always on | `flags.enable_integration_api: true` in `config.yml` |

The Swagger UI, `/v1/openapi.json`, and every `resource`/`org`/`site` management endpoint below live
on the Integration API. The Dashboard API never accepts a Bearer token as a fallback.

## Enable it temporarily
The module has no option for the Integration API, and `/var/lib/pangolin/config/config.yml` is
regenerated from its sops template on every activation (`./clu build`, every `nixos-rebuild switch`,
every boot). So a hand edit is ***temporary by construction*** - which suits a management API with
this much blast radius: use it, then let the next activation turn it back off.

1. Add the flag to the live `config.yml` on the VPS - append it under the existing `flags:` block:
   ```bash
   $ sudo nano /var/lib/pangolin/config/config.yml
   ```
   ```yaml
   flags:
       # ...existing flags...
       enable_integration_api: true
   ```
   Match the file's existing 4-space indentation - a mis-indented key is silently ignored.
2. Restart only the `pangolin` container - Gerbil, Traefik and CrowdSec don't need to move (Traefik
   lives in Gerbil's network namespace, not Pangolin's):
   ```bash
   $ sudo podman restart pangolin
   $ sudo podman logs --since 2m pangolin 2>&1 | grep -i 'integration api'
   ```
   Expect `Integration API server is running on http://localhost:3003`. Despite the wording, the
   server calls `listen(3003)` with no host, so it listens on every interface *inside the
   container* - which is what the next section relies on.

If you'll use the API regularly, the durable version is a module option that adds the flag to the
rendered `config.yml` (still without publishing the port) - kept off by default.

## Reach it without publishing anything
Don't add a port mapping. The host can already reach the container on the stack's podman bridge, so
the API stays off every public interface while still being usable:

1. Find the container's bridge address (assigned at start, so look it up each time):
   ```bash
   $ sudo podman inspect pangolin --format '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}'
   ```
2. From your own machine, tunnel to it over the hardened sshd port (`2222`, see
   [Ports and Firewall](../README.md#ports-and-firewall)):
   ```bash
   $ ssh -p 2222 -L 3003:<container-ip>:3003 admin@pangolin.example.com
   ```
   Leave that session open and run the `curl` calls below against `localhost:3003` from a second
   terminal. The Swagger UI is at `http://localhost:3003/v1/docs` the same way.

## Create a scoped org API key
1. Switch into the org that owns what you're changing (top-left org switcher).
2. Navigate to `ORGANIZATION → API Keys`.
3. Click `+Create API Key` and give it a descriptive name (e.g. `vault-private-resource-sni-config`)
   so future you knows why it exists.
4. Grant only what the task needs - e.g. `Update Resource` (`updateResource`), plus
   `List Resources` (`listResources`) if you need to look up IDs. Avoid org-wide/root actions for a
   one-off task.
5. **Copy the key immediately** - it's only displayed once, as `<keyId>.<keySecret>`; both halves
   together are the token.

## Authenticate requests
Every call needs the full `<keyId>.<keySecret>` as a bearer token, against the **Integration API's**
port and prefix - `/v1/...`, not `/api/v1/...`. Keep the key out of your shell history and the
process list by feeding it to curl as a config on stdin:
```bash
$ read -rs KEY        # paste <keyId>.<keySecret>, nothing echoes
$ api() { printf 'header = "Authorization: Bearer %s"\n' "$KEY" | curl -sS -K - "$@"; }
$ api "http://localhost:3003/v1/resource/<resourceId>" | jq
```
`printf` is a shell builtin, so the key never appears in an `argv` that `ps` could show. Since `api`
uses stdin for that config, pass any request body as a file (`--data-binary @file`), never `@-`.
Full request/response schemas for every endpoint are in the Swagger UI.

## Finding IDs you'll need
Public and Private resources are separate tables with separate ID spaces and separate listing
endpoints - a numeric ID from one means nothing to the other (expect `404 Not Found` if mixed up).

* **Public resources** - `resourceId`, the numeric ID in a resource's dashboard URL
  (`.../resource/<id>/...`), or:
  ```bash
  $ api "http://localhost:3003/v1/org/<org-id>/resources" | jq '.data.resources[] | {resourceId, name, fullDomain}'
  ```
* **Private resources** - `siteResourceId`:
  ```bash
  $ api "http://localhost:3003/v1/org/<org-id>/private-resources" \
    | jq '.data.siteResources[] | {siteResourceId, name, mode, destination, destinationPort}'
  ```
  A resource's `niceId` (the word-slug in some dashboard URLs) is **not** accepted by either -
  both IDs are validated as positive integers.
* `<org-id>` - the org slug/ID visible in the dashboard URL once you're inside that org.

## Example: create a site with your own Newt credentials
The dashboard always generates a site's Newt ID and secret, but `PUT /v1/org/{orgId}/site`
([`createSite.ts`](https://github.com/fosrl/pangolin/blob/main/server/routers/site/createSite.ts))
accepts optional `newtId` and `secret` fields and only generates them when omitted - useful to stand
up a replacement Pangolin that an existing Newt (credentials already in sops) connects to unchanged.
The key needs the `Create Site` action.
```bash
$ read -rs NEWT_SECRET
$ body=$(mktemp)      # created 0600
$ jq -n --arg id "<newt-id>" --arg s "$NEWT_SECRET" \
    '{name:"homelab", niceId:"homelab", type:"newt", newtId:$id, secret:$s}' > "$body"
$ api -X PUT -H 'Content-Type: application/json' --data-binary @"$body" \
    "http://localhost:3003/v1/org/<org-id>/site" | jq '.success, .message'
$ rm -f "$body"
```
* Leave `address`, `subnet` and `exitNodeId` out - for `newt` sites Pangolin picks them when Newt
  first connects.
* `newtId` must be unique on the instance; the secret is stored only as a hash.
* The response echoes the credentials back - the `jq` above prints only success/message.
* The endpoint isn't a site field: it's the dashboard URL Newt is already configured with
  (`services.oci.newt.pangolin.url`).

Pangolin's Blueprints can't do this - their `sites:` section only accepts `name` and
`docker-socket-enabled`, and only for sites that already exist.

## Turn it back off
1. Delete the API key (or narrow its actions) - there's no reason to keep a broad key around after
   the task that needed it.
2. Restore the rendered config and restart the container. Either run `./clu build` (which re-copies
   `config.yml` along with everything else) or, without rebuilding:
   ```bash
   $ sudo systemctl restart systemd-tmpfiles-resetup.service   # re-copies config.yml from the template
   $ sudo podman restart pangolin
   ```
   ***Restart the container either way*** - restoring the file alone leaves the running Pangolin
   with the API still enabled, and a rebuild only restarts the stack when the plaintext configs
   changed (the secret-bearing `config.yml` isn't part of that hash).
3. Confirm it's gone:
   ```bash
   $ sudo podman logs --since 2m pangolin 2>&1 | grep -ci 'integration api'   # 0
   ```

***References***
* [Integration API - Pangolin Docs](https://docs.pangolin.net/manage/integration-api)
