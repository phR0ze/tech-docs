# OpenCloud <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Self-hosted file storage focused alternative to Nextcloud. Collabora built into the file management.

### Quick links
- [.. up dir](../README.md)
- [Overview](#overveiw)
  - [Compared to Google drive](#compared-to-google-drive)
- [Local Deployment](#local-deployment)

## Overview
Forked from ownCloud infinite scale (OCIS), OpenCloud is essentially a continuation of the OCIS
Go-based rewrite developed by the Heinlein Group in Berlin, launched in 2025. The team jumped ship to
focus more on open source. So fundamentally why use OpenCloud instead of Nextcloud:

**Pros**
* Its a lot faster. OpenCloud is Golang based while Nextcloud is PHP based.
* OpenCloud stores all metadata on the filessystem alongside file data. There is no MySQL instance to
  maintain, no schema migrations, no database backups. Just snapshot the whole files system for
  backtups.
* OpenCloud's desktop client uses delta sync and is way faster
* Doesn't include basic apps like mail, calendar, chat

**Cons**
* Doesn't include basic apps like mail, calendar, chat

### Compared to Google Drive
* Real-time document editing
  - simultaneous editing from multiple users works fine
* File Versioning
  - versions are automatically saved and can be restored
* Sharing and Permissions
  - view, edit and public permissions work well
  - no forced signups just clean download links for public view
* Spaces
  - team based collaboration model where all content belongs to the space rather than individual
    accounts.
* Anonymous File drop
  - A secrete file drop allows anonymous users to securely upload files without needing an account
* Federated Sharing
  - Open Cloud Mesh allows for secure file sharing and collaboration across multiple instances

**Where it falls short**
* No native document format.
  - It depends on Collabora which feels more like LibreOffice than Google Docs
* No Google Sheets or Slides alternative
* Mobile apps are still maturing
* No built in chat/meet

## Local Deployment

