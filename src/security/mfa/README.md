# MFA <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

Multi-factor authentication

### Quick links
* [.. up dir](..)
* [Authenticator Apps](#authenticator-apps)
  * [Authenticator](#authenticator)
  * [Authme](#authme)
  * [Authy](#authy)
  * [Keysmith](#keysmith)
  * [Maurborgne](#maurborgne)
  * [OTPClient](#otpclient)

### Linked pages

# Authenticator Apps
Authenticator apps are used to generate random codes used for two-factor authentication (2FA). To use 
one you need to install the app, add an account and the authenticator will generate a time-based 
six-digit code to use for 2FA for the site or service in question.

## Authenticator
GTK based linux desktop app that supports over 200 providers. Any site or service that supports 
tokens should be able to work with it.

Fork of the original [Authenticator](https://launchpad.net/authenticator)

**References**
* [5 best password managers for linux](https://itsfoss.com/password-managers-linux)
* [Gitlab source for Authenticator NG](https://gitlab.com/dobey/authenticator-ng)

**Pros:**
* Supports both HOTP and TOTP
* QR code scanner via screenshots
* Huge database of more than 560 supported services
* Keep your PIN tokens secure by lockin the app with a password
* Automatically fetch an image for services using their favicon
* Dark and light themes
* No cloud sync your data stays on your computer

## Authme

**References**
* [Github project](https://github.com/Levminer/authme)
* [Authme website](https://authme.levminer.com/)

**Pros:**
* Rust and Svelte based
* Data is AES 256bit encrypted with your own password
* Import from any 2FA TOTP QR code
* Hidden from video capture and screenshots
* Easy export and backup of 2FA codes
* Completely offline
* Actively maintained with recent releases
* AUR package

**Cons:**
* No Android version
* No NixOS package

## Authy
Authy is one of the best authenticator apps out there and has a Linux version as well. Unfortunately 
the purpose of a desktop authenticator in my opinion is to avoid your phone being a single point of 
failure in your authentication and Authy ties your account to a phone number complicated this use 
case.

**Pros:**
* Authy keeps your data safe by encrypting your auth data and not storing passwords on its servers

**Cons:**
* SNAP only package management
* Ties your account to your phone number which will be a problem if you change phones

## Keysmith
KDE based desktop application to generate two-factor authentication (2FA) tokens when loggin into 
your online accounts.

Originally this code was based largely on the [authenticator-ng](https://gitlab.com/dobey/authenticator-ng)
application by Rodney Dawes and Michael Zanetti for Ubuntu Touch.

**References**
* [KDE's Keysmith](https://apps.kde.org/keysmith/)

**Pros:**
* Actively maintained with recent releases
* Supports HOTP and TOTP tokens
* Available in in Arch Linux extra repo

**Cons:**
* QR code scanning not yet supported
* Backup and restore accounts not yet supported

## Maurborgne
Maurborgne is a 2FA OTP generator that can generate HTOP and TOTP codes in order to access various 
services similar to other authenticator apps.

**References**
* [Maurborgne website](https://jhaygood86.github.io/mauborgne/)

**Pros**
* Secrets are stored locally only using libsecret
* App doesn't connect to the network for any reason
* App is intentionally sandboxed to minimize access to outside resources
* Import OTP pads using a built in screenshot capture of QR codes
* Export OTP pads by generating a QR code
* Import/Export using Aagis Encrypted JSON Vaults

**Cons**
* Interface seems a bit clunky
* Depends on libportal for taking screenshots
* Odd toolchain Vala, meson, ninja
* Not available in the AUR

## OTPClient

**References**
* [Blog review](https://opensourcemusings.com/3-multi-factor-authentication-apps-for-the-linux-desktop)

**Pros**
* Scan code using webcam

**Cons**
* A bit simplistic looking

<!-- 
vim: ts=2:sw=2:sts=2
-->
