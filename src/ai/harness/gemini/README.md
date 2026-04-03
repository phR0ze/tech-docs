# Gemini <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Gemini is Google's latest branding around AI. It encompasses:
* Gemini model - Latest Google LLMs
* Gemini embedded agent in various products e.g. Chrome, gmail etc...
* Gemini CLI - open source project that exposes the Gemini model

Google also provides a larger umbrella of all things AI including open source models and competing
vendor modles like anthropic through its Vertex API and associated model garden.

### Quick Links
- [.. up dir](..)
- [Overview](#overview)
  - [Review](#review)
  - [Telemetry](#telemetry)
  - [Install on NixOS](#install-on-nixos)
  - [First run](#first-run)
  - [Paid tier](#paid-tier)
- [Configuration](#configuration)
  - [settings.json](#settingsjson)
  - [GEMINI.md](#geminimd)

## Overview

### Review
This is by no means a complete review or even fair. It is simply my impressions that I'll be adding
to over time about my experience with the Gemini CLI. Although I'm sure there are ways ot configuring
it to behave better with my minimal configuration similar to what I've done in OpenCode or Claude
Code I'm seeing the following:

* Gemini CLI is super slow responding to requests comparatively

### Telemetry
Like Claude Code and likely all enterprise positioned solutions, Gemini will report back telemetry.
It does so in two ways. It will report anonymized usage statistics to Google so they can bill
effectively and ***help improve the tool***. You can opt out of the improve the tool aspects by
[disabling statistics gathering](#settingsjson). The second way telemetery is report is to enterprise
company administrators. This is disabled by default but can be enabled to monitor everything your
doing through the CLI, both prompt input and output.

You can determine if it is activated on your machine by checking:
* `/etc/gemini-cli/settings.json`

### Install on NixOS
I [rolled my own](https://github.com/phR0ze/nixos-config/tree/main/options/apps/dev/gemini)
to keep install to keep up with the latest.

### First run

1. Launch gemini with `gemini` in a terminal
2. Likely it will ask you to trust the folder your in
3. Next it will ask how you want to authenticate
   1. Choose `Login with Google`
   2. Work through the consent form in the popup browser
4. Restart the CLI

### Paid tier
Getting Gemini up and running is simple and fast via OIDC and your google account. However that won't
give you access to the paid tier by default. There are additional steps to setup a project and
configure billing on the project inside google's [AI Studio](https://aistudio.google.com/).
Alternatively for the enterprise space you can configure Google's ADC (Appliction Default
Credentials) path to expose the Vertex AI API to then give you access to the GCP project where you
have configured Gemini API keys that can be used.

**WARNING** the 3.1 models are only available in the `global` region when using the Vertex AI path as
of 3/4/2026.

1. Ensure you have gcloud installed
   ```nix
   {
     environment.systemPackages = [
       pkgs.google-cloud-sdk
     ];
   }
   ```
2. Configure ADC (Application Default Credentials)
   ```bash
   $ gcloud auth login --update-adc
   ```
3. Configure environment variables for project and location
   ```bash
   $ export GOOGLE_CLOUD_PROJECT="your-project-id"
   $ export GOOGLE_CLOUD_LOCATION="global"
   ```
4. I don't believe this is necessary, but you can default your gcloud config as well
   ```bash
   $ gcloud config set project $GOOGLE_CLOUD_PROJECT 
   $ gcloud auth application-default set-quota-project $GOOGLE_CLOUD_PROJECT
   ```
5. Ensure that your project has Vertex AI enabled
   ```bash
   $ gcloud services list --enabled --filter="name:aiplatform.googleapis.com"

   NAME                       TITLE
   aiplatform.googleapis.com  Vertex AI API
   ```
5. Start Gemini and go through the re-login flow
   1. Trigger auth login
      ```
      > /auth login
      ```
   2. Choose `Vertex AI` which will pick up the env vars and silently complete the login

## Configuration

### settings.json
Settings are configured in `~/.gemini/settings.json` globally or `.gemini/settings.json` for the
local repository only.

* Disable telemetry
  ```json
  {
    "usageStatisticsEnabled": false
  }
  ```
* Disable dynamic window title
  ```json
  {
    "ui": {
      "dynamicWindowTitle": false
    }
  }
  ```


### GEMINI.md
`GEMINI.md` is much like `CLAUDE.md` as all these CLIs are just copying each other.

