# Firefox <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Documenting how I like to configure Firefox

### Quick links
* [.. up dir](..)
* [Config](#config)
  * [Recent tab switching](#recent-tab-switching)
* [Extensions](#extensions)
  * [Markdown viewer](#markdown-viewer)
  * [Privacy Badger](#privacy-badger)
  * [uBlock Origin](#ublock-origin)
* [Troubleshooting](#troubleshooting)
  * [Clear your cookies and cache](#clear-your-cookies-and-cache)
  * [Refresh Firefox](#refresh-firefox)

## Config

### Recent tab switching
1. Navigate to `MENU >Settings`
2. Check `General >Ctrl+Tab cycles through tabs in recently used order`

## Extensions

### Markdown Viewer
1. Browse to `about:config`
2. Search for `helpers.private_mime_types_file` which show the mime path `~/.mime.types`
3. Create the file `~/.mime.types`
   ```
   text/plain       md txt
   ```
4. Download and install [Markdown viewer](https://addons.mozilla.org/en-US/firefox/addon/markdown-viewer-chrome/)

### Privacy Badger
Privacy Badger automatically sends the Global Privacy Control signal to opt you out of data sharing 
and selling, and the Do Not Track signal to tell companies not to track you. If trackers ignore these 
signals, Privacy Badger will learn to block them

1. Enable the `Run in Private Windows` option

### uBlock Origin
Blocking ads and annoying popups has become more and more imperative to keep you browser running 
smoothly, improve load times and safety online.

**Install**
1. Navigate to `MENU >Add-ons and themes`
2. Punch into the search `ublock origin`
3. Click through and then `Add to Firefox >Add`
4. Select `Allow this extension to run in Private Windows` and click `Okay`

**Configuration**
1. Enable `Filter lists`
   * `Annoyances >EasyList - Annoyances`
   * `Annoyances >AdGuard - Annoyances`
   * `Annoyances >uBlock filters - Annoyances`

## Troubleshooting
Firefox stores data in:
* `~/.mozilla` profile information
* `~/.cache/mozilla` caching

### Clear your cookies and cache
The first thing to try if Firefox is having issues it to clear your cookies and cache

1. Navigate to `Settings >History >Clear recent history...`
2. Select `Everything` from the `When` drop down
3. Click `Clear`

### Refresh Firefox
Firefox has a nice troubleshooting feature that lets you restart in a safe mode and refresh Firefox. 
The refresh will remove extensions, caching and customizations but won't remove bookmarks. It will 
then attempt to re-add your extensions afterwards in a clean state.

Note: this fixed my slow google docs experience

1. Navigate to `Settings >Help >Troubleshoot Mode...`
2. Hit the `Restart` button
3. Click the `Refresh Firefox` button
4. Click `Refresh Firefox`, it won't clear bookmarks
