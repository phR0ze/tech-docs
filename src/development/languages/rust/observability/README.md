# Observability <img style="margin: 6px 13px 0px 0px" align="left" src="../../../../data/images/logo_36x36.png" />

Observability consists of instrumenting, generating, collecting and exporting telemetry (metrics, 
logs, traces).

### Quick links
- [.. up dir](../README.md)
* [Overview](#overview)
  * [Distributed Tracing](#distributed-tracing)
  * [Logging](#logging)

### Linked pages
* [Tracing](tracing/README.md)

## Overview
There are three components to observability:
* `application instrumentation` where a library is integrated into your application that provides the 
  ability to send or make available telemetry for the telemetry collection service to either receive in 
  a push model (think New Relic agent) or actively collect in a pull model (think Prometheus).

* `telemetry collection` - is the service that either collects or receives telemtry from the 
  applications and makes it available to an API or UI tooling for visualization. Typically the 
  collector will be receiving pushes from the client instrumentation.

* `telemetry visualization` - UI that presents the telemetry in a visually explorable way

**References**
* [Observability Primer](https://opentelemetry.io/docs/concepts/observability-primer/)

### OpenTelemetry
OpenTelemetry is a standard for instrumenting applications to generate telemetry data i.e. `metrics`, 
`logs` and `traces` that is then sent to a collector. In the application the OTel integration is 
configured to give a name for the application that it is then registered under on the collector and 
under which all the telemetry is associated and stored. The OTel app client then sends all telemetry 
data to the collector service.

**References**
* [Distributed Tracing with Jaeger](https://www.youtube.com/watch?v=SA8BainHeS0)
* [OpenTelemetry integration](https://www.youtube.com/watch?v=HrRrJ5wTtdk)

An OpenTelemetry exporter refers to the component in the OTel client code in the app that is 
responsible for sending the telemetry to the collector. It can be configured directly in code or can 
be configured through environment variables i.e. the enterprise way.

### Distributed Tracing
Distributed tracing lets you ***observe requests as they propagate*** through complex, distributed 
systems. Distributed tracing improves the visibility of your application or system's health and lets 
you debug behavior that is difficult to reproduce locally. It is essential for distributed systems, 
which commonly have nondeterministic problems or are to complicated to reproduce locally.

A ***log*** is a timestamped message emitted by services or other components. Unlike traces, they 
aren't necessarily associated with an particular user request or transaction. Logs 

A ***span*** represents a unit of work or operation. Spans track specific operations that a request 
makes, painting a picture of what happened during the time in which that operation was executed. A 
span contains a name, time-related data, structued log messages and other metadata.
