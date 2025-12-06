# Streaming <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

WARNING: only use these techniques only for educational, debugging, personal-offline-playback 
permitted by the service’s terms or for media that you originally uploaded and are simply backing up. 

### Quick links
* [.. up dir](../README.md)

### Linked pages
- [Download](download/README.md)

## Devtools exploration
Often its useful in learning to understand the browser interaction to get a better understanding of 
the streaming service that you

1. Open DevTools and reproduce the playback flow
  - Open the developer tools view and switch to the `Network` tab before loading the page.
  - Clear existing logs (the “clear” button) so only fresh requests show.
  - Refresh the page or start playing the video. Keep DevTools open and recording.

2. Filter network requests
  - Look for `Media`, `XHR`, `m3u8`, `manifest`
  - Look for file extensions: `.m3u8` (HLS), `.mpd` (DASH), `.ts`, `.mp4` requests
  - Look for endpoints named `/manifest`, `/play`, `/license`, `/token`, `/segment`, `/hls`, `/dash`.

3. Look for playlist/manifest responses
  - For `HLS` an `.m3u8` playlist is the usual entry. Master playlist lists variant streams; variant playlists list .ts segment URLs. A manifest may be tokenised (contains query string like ?token=...).
  - For `DASH` look for .mpd XML files referencing .m4s segments.
  - If you find a manifest URL, right‑click → “Open in new tab” to inspect its contents. If it shows readable URLs, those are the streaming links (often short‑lived).

4. Handle tokenised or time‑limited URLs
- Many streaming URLs include expiring tokens. Copying the URL may work only briefly.
- Some sites request manifests via an XHR that supplies a token; examine the request payload/response headers to understand how tokens are generated (often a prior /play or /auth call).

5. Watch for DRM/license requests
- If network shows requests to endpoints like /license or to Widevine/PlayReady license servers, the content is DRM protected. Manifests may exist, but decryption requires a license — you cannot get usable clear video by URL alone.
- In DevTools, license requests often show Content-Type application/octet-stream or specific license endpoints (license, wid, playready).

6. Inspect player JavaScript
- Search loaded JS for keywords: m3u8, Hls.js, shaka, dashjs, createPlayer, src, token, auth. The player logic often constructs manifest URLs or exchanges tokens. Copying the code can show how the manifest URL is built.
- Beware obfuscated/minified code — look for XHR/fetch calls that return JSON containing manifest or token.

7. Use a media tool when a manifest exists
- If you find a valid .m3u8/.mpd URL that is not DRM‑protected, tools like ffmpeg, VLC, or streamlink can play or download:
- ffmpeg -i "manifest.m3u8" -c copy out.mp4
- streamlink "manifest.m3u8" best -o out.ts
- If the manifest requires headers (Referer, Authorization, Cookie), replicate them with the tool (ffmpeg supports -headers, streamlink has options for cookies/headers).

8. Cookies, headers and CORS
- Some manifests require the same cookies/session or special headers. In DevTools, copy the request headers (Right-click request → Copy → Copy as cURL) and adapt them for ffmpeg/streamlink or curl to fetch manifests.
- For segment downloads, preserve Referer and Cookie headers; otherwise the server may reject requests.

9. When DRM is present: alternatives
- Accept that direct extraction won't produce playable clear video.
- Use the official app or website for playback.
- For legitimate offline access, check if the service offers downloads in its official apps.
- For analysis/learning, inspect encrypted manifests and license flows to understand implementation — but you cannot decrypt without the license key and a compliant client.

10. Practical checklist summary
- Start DevTools → Network before playback.
- Filter for m3u8/mpd/ts/mp4/manifest/XHR.
- Inspect XHR payloads for token/manifest URLs.
- Check for license/DRM endpoints (Widevine/PlayReady).
- If non‑DRM manifest found, copy URL and account for headers/cookies and short TTL.
- Use ffmpeg/streamlink/VLC with same headers/cookies to play/download.

Typical outcomes and examples
    Example A: A site exposes an .m3u8 with query token: you can open the .m3u8 and feed it to VLC for a short time until the token expires.
    Example B: Player is HLS + AES with no license endpoint: the segments might be encrypted with a static key from the manifest — decryptable if key is accessible, but most commercial sites avoid this.
    Example C: Widevine DRM: manifest present but segments encrypted and license server required — playback only via licensed clients.
