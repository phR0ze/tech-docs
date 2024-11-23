# Privoxy <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

[Privoxy](https://www.privoxy.org) is a non-caching web proxy with advanced filtering capabilities 
for enhancing privacy, modifying web page data and HTTP headers, controlling access, and removing
ads and other obnoxious internet junk.

### Quick links
* [.. up dir](..)
* [Overview](#overview)
* [Getting started](#getting-started)
  * [Install](#install)
    * [Test](#test)
  * [Basic config](#basic-config)
    * [http\_proxy](#http-proxy)
    * [Server mode](#server-mode)
* [Configuration](#configuration)
  * [Block Ads](#block-ads)
  * [Block Sites](#block-sites)

## Overview
Written in C the source can be found at https://github.com/ssrlive/Privoxy

### Features
* Block ads
* Disables pop-ups
* Managing cookies
* Controlling access
* Remove unwanted web content
* User-Agent spoofing
* Stealth mode, removes the previous web page you visited
* Detect and disable click-tracking scripts

## Getting started

## Build
It's necessary to build privoxy to get the `https-inspection` feature and include the `--with-openssl` flag

1. Download the tarball from https://sourceforge.net/projects/ijbswa/files/latest/download

## Install
```bash
$ sudo pacman -S privoxy
$ sudo systemctl enable privoxy
$ sudo systemctl start privoxy
```

### Test
1. Launch chromium with the proxy set
   ```bash
   $ http_proxy="http://localhost:8118" chromium
   ```
2. Navigate to `http://p.p`

## Basic config

### http\_proxy
Firefox, Chromium and many other applications will respect the system wide proxy configured using
the environment variables:
* `http_proxy="http://localhost:8118"`

### Server mode
Privoxy can easily be configured to be available on your network as a shared proxy service

1. Edit `/etc/privoxy/config`
2. Set `listen-address` to your server's network IP e.g. `192.168.1.3:8118`

## Configuration

### Block Ads
Block ads with https://pgl.yoyo.org

### Block Sites
Whitelist

Edit your `/etc/privoxy/user.action` file and add
```
############################################################
# Block everything but exception list
############################################################

# Block everything
{ +block }
/

# Exception list
{ -block }
kids.example.com
toys.example.com
games.example.com
```

<!-- 
vim: ts=2:sw=2:sts=2
-->
