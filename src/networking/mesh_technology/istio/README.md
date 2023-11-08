# Istio

## Ambient Mesh Data Plane
The Ambient Mesh data plane is a sidecarless approach to deploying Istio that allows you to opt into 
various capabilities gradually by splitting the Layer 4 and Layer 7 capabilities. Layer 4 is a 
tunneling technology that is much closer to the CNI. Essentially Istio is implementing a CNI plugin 
to provide a single ztunnel instance per node i.e. daemon set instead of a sidecar proxy. However to 
get the standard Envoy proxy capabilities they have setup a `waypoint` proxy per application as 
needed. So instead of always having a sidecar we can only use a Envoy if needed.

The real win is that you don't have to touch the application as much.

***References***
* [Istio Ambient Mesh Launch Demo - youtube](https://www.youtube.com/watch?v=nupRBh9Iypo)
