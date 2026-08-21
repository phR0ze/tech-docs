# Using the Pangolin API <img style="margin: 6px 13px 0px 0px" align="left" src="../../../../data/images/logo_36x36.png" />

Some [Pangolin](../README.md) resource fields — like `tlsServerName`/`setHostHeader` on a Private
HTTP resource, per the [Vaultwarden Example](../vault_example/README.md) — aren't exposed in the
dashboard form yet, even though the backend and API fully support them. For anything the dashboard
doesn't cover, Pangolin's REST API is the way in — same underlying actions the dashboard itself
calls, just addressable directly.

### Quick links
- [.. up dir](..)
- [Two separate API servers — don't confuse them](#two-separate-api-servers--dont-confuse-them)
- [Turn it on already locked down — one edit pass, one restart](#turn-it-on-already-locked-down--one-edit-pass-one-restart)
- [Create a scoped org API key](#create-a-scoped-org-api-key)
- [Authenticate requests](#authenticate-requests)
- [Finding IDs you'll need](#finding-ids-youll-need)

## Two separate API servers — don't confuse them
Checked against the upstream source (`fosrl/pangolin`, `main` branch): Pangolin actually runs *two*
independent HTTP servers, and only one of them accepts an API key. Hitting the wrong one is why a
Bearer-token `curl` call against `https://pangolin.example.com/api/v1/...` comes back
`{"message":"Unauthorized","status":401}` even with a perfectly valid key — that generic message
(not `"Invalid API key"` or `"API key required"`) is the tell, since it comes from
[`verifySession`](https://github.com/fosrl/pangolin/blob/main/server/middlewares/verifySession.ts)'s
session-cookie check, not from the API-key middleware at all — the request never reached API-key
code to begin with.

| | **Dashboard API** (what you just hit) | **Integration API** (what you actually want) |
|---|---|---|
| Source | [`server/apiServer.ts`](https://github.com/fosrl/pangolin/blob/main/server/apiServer.ts) | [`server/integrationApiServer.ts`](https://github.com/fosrl/pangolin/blob/main/server/integrationApiServer.ts) |
| Auth | Session cookie only ([`verifySession`](https://github.com/fosrl/pangolin/blob/main/server/middlewares/verifySession.ts)/[`verifyUser`](https://github.com/fosrl/pangolin/blob/main/server/middlewares/verifyUser.ts)) — a Bearer header is silently ignored | API key, `Authorization: Bearer <keyId>.<keySecret>` ([`verifyApiKey`](https://github.com/fosrl/pangolin/blob/main/server/middlewares/integration/verifyApiKey.ts)) |
| Path prefix | `/api/v1/...` | `/v1/...` (**no** `/api`) |
| Port (`config/config.yml`) | `server.external_port`, default `3000` | `server.integration_port`, default `3003` |
| Reachable via | The public dashboard domain, through the `api-router` the main doc's own [`dynamic_config.yml`](../README.md#write-configtraefikdynamic_configyml) already wires up to `http://pangolin:3000` | **Nothing, by default** — not started at all unless enabled, and not routed by any Traefik rule this doc sets up |
| Enabled by | Always on | `flags.enable_integration_api: true` in `config/config.yml` — off unless you turn it on |

The Swagger UI, `/v1/openapi.json`/`.yaml`, and every `resource`/`org`/`site` management endpoint
covered by this doc all live on the Integration API side. There is no overlap where the Dashboard
API also accepts a Bearer token as a fallback — it's cookie-or-nothing.

## Turn it on already locked down — one edit pass, one restart
This is a resource/org/site *management* API — broader blast radius than any single resource, and
worth keeping off the public internet even though it's key-gated, the same reasoning the main doc
already applies to Traefik's own insecure dashboard API (see
[Traefik's Insecure API Isn't Actually Exposed](../README.md#traefiks-insecure-api-isnt-actually-exposed)).
Make both edits — enabling the API and binding it to localhost-only — *before* touching Docker at
all, so there's never a step where it's reachable without the restriction already in place:

Add the flag to `config/config.yml` (same file [written earlier](../README.md#write-configconfigyml)):
1. Change directory
```bash
$ cd /opt/pangolin
```
2. Edit `config/config.yml` with these changes
```yaml
flags:
  enable_integration_api: true
```
3. Add the port mapping, bound to localhost only, edit `docker-compose.yml`
```yaml
  pangolin:
    # ...existing config...
    ports:
      - "127.0.0.1:3003:3003"
```
Then apply both in a single step — `docker compose restart` won't pick up a `ports:` change at
all (it just stops/starts the container as it was originally created), so this needs `up -d`,
which recreates the container with both edits already baked in. Docker publishes port mappings
atomically at container creation, so there's no intermediate moment where the port exists without
the `127.0.0.1`-only restriction already applied:
```bash
$ cd /opt/pangolin
$ sudo docker compose up -d pangolin
$ sudo docker compose logs pangolin | grep -i "integration api"
```
Confirm the log line `Integration API server is running on http://localhost:3003` appears — if it
doesn't, the flag didn't take (check `config.yml` indentation, it's parsed by the same schema as
every other setting in that file). Confirm the bind separately, from the VPS itself:
```bash
$ sudo ss -tulpn | grep 3003
```
Should show `127.0.0.1:3003`, not `0.0.0.0:3003` or `[::]:3003` — if it's bound to all interfaces
instead, the `ports:` mapping didn't take (check the compose file was actually saved before the
`up -d`, and that `docker compose config` reflects it).

Then reach it from your own machine over an SSH tunnel through the already-hardened
[`2222/tcp`](../README.md#harden-sshd) port, rather than curling the VPS directly:
```bash
$ ssh -p 2222 -L 3003:127.0.0.1:3003 youruser@pangolin.example.com
```
Leave that session open and run the `curl` calls below against `localhost:3003` from a second
terminal on your own machine — nothing about the Integration API needs to be reachable when you're
not actively using it.

## Create a scoped org API key
1. Switch into the org that owns the resource you're changing (top-left org switcher, if you have
   more than one).
2. Navigate to `ORGANIZATION → API Keys`.
3. Click `+Create API Key`, give it a descriptive name (e.g. `vault-private-resource-sni-config`) so
   that future you will know why you created it.
4. In the permissions/actions selector, grant only what the task needs — e.g. `Update Resource`
   (`updateResource`) for the field-setting call above, plus `List Resources` (`listResources`) if
   you also need to look up a `resourceId` by name instead of reading it off the dashboard URL.
   Avoid granting org-wide/root actions for a narrow one-off task.
5. Click through to generate it, then **copy the key immediately** — Pangolin only displays it
   once; missing it means deleting and recreating the key, not retrieving it again. It's shown as
   `<keyId>.<keySecret>` — both halves together are the token, not just the part after the dot.

## Authenticate requests
Every call needs the full `<keyId>.<keySecret>` value as a bearer token, against the **Integration
API's** port and prefix — `/v1/...`, not `/api/v1/...`:
```bash
$ curl -H "Authorization: Bearer <keyId>.<keySecret>" "http://localhost:3003/v1/resource/<resourceId>"
```
Full request/response schemas for every endpoint are in the Swagger UI, reachable the same way once
the tunnel is up: `http://localhost:3003/v1/docs`.

## Finding IDs you'll need
Public and Private resources are entirely separate tables with separate ID spaces and separate
listing endpoints — pick the one matching what you're actually looking up, a numeric ID from one
table means nothing to the other's endpoint (expect a `404 Not Found` if you mix them up).

* **Public resources** — `resourceId`, the numeric ID in a resource's dashboard URL
  (`.../resource/<id>/...`), or from listing every Public resource in an org:
  ```bash
  $ curl -s "http://localhost:3003/v1/org/<org-id>/resources" \
    -H "Authorization: Bearer <keyId>.<keySecret>" | jq '.data.resources[] | {resourceId, name, fullDomain}'
  ```
* **Private resources** — `siteResourceId`, from listing every Private resource in an org instead:
  ```bash
  $ curl -s "http://localhost:3003/v1/org/<org-id>/private-resources" \
    -H "Authorization: Bearer <keyId>.<keySecret>" | jq '.data.siteResources[] | {siteResourceId, name, mode, destination, destinationPort}'
  ```
  A resource's `niceId` (the friendly `some-random-word`-style slug shown in the dashboard URL for
  some views) is **not** accepted as an ID in either endpoint above — `resourceId`/`siteResourceId`
  are both validated as a positive integer, so passing the `niceId` string fails parameter
  validation rather than looking the resource up by name.
* `<org-id>` — the org slug/ID visible in the dashboard URL once you're inside that org.

**After a one-off change**, either delete the key or narrow its actions back down — there's no
reason to keep a broadly-scoped key sitting in `API Keys` after the task that needed it is done.
A key that's still attached to an audit trail you can read later is more useful than one you have
to guess the purpose of six months on. Consider also turning `enable_integration_api` back off in
`config/config.yml` if you don't expect to need this again soon — one less running listener, even
one bound to localhost only, is one less thing to account for later.

***References***
* [Integration API - Pangolin Docs](https://docs.pangolin.net/manage/integration-api)
