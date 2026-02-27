# Monitoring <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

### Quick links
- [.. up dir](../README.md)

### Linked pages
* [Sentry](sentry/README.md)

# New Relic
All the sections below are UI sidebar entries

* NRQL - New Relic query language
* PROMQL - Prometheus QL can be used in UI queries

## All Entities
* Gray dots mean no alerts conditions associated with the entry
* Red is a critical condition
* Summary view shows the golden metrics
* Navigator view gives you a traffic light high level summary

## Alerts & AI
* Policies (multiple conditions per/policy)
* General
  * Alerts can have an evaluation delay which delays before starting to analyze for app startup
  * Delays can be setup to wait for data to be sent to NR
  * Ability to add multiple thresholds to distinguish between warning and critical alerts
* Alert workflows
  * Can be created and attached to an alert
  * Configures the notifications to various outputs
    * Email, Slack
* GKE alert policy (via Add Data option)
  * Pre-configured alerts that can be dropped in
* APM
  * Can be created from views
  * Pre-built Ruby Alert Policy
* Synthetic Monitoring
  * Selenium scripts to actively ensure things are working (Scripted browser)
  * You can run them as crons from multiple regions to test from different parts of the world

### Alert Coverage Gaps
* Selecting an entry and clicking `Alert` will provide some suggested alerts
