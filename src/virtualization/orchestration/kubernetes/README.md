# Kubernetes

## Local dev
There are a few options for setting up a mini k8s environment for local development in VMs.

**References**
* [minikube vs k8d vs kind vs getdeck](https://www.blueshoe.io/blog/minikube-vs-k3d-vs-kind-vs-getdeck-beiboot/)

## Google resource hierarchy

* Organization
* Folders - policy application
* Projects - resource logical grouping
* Resources

## Kubernetes Enterprise
Kubernetes that includes Anthos to manage your networking

## Kubernetes Cluster
A k8s cluster is composed of worker nodes and control plane nodes. GKE always handles the control 
plane nodes and only the worker nodes are specified by the end user. GKE provides the cluster 
managment components like node auto scaling and node management.

## Workload Identity
Each workload can have its own identity that you can use to control individual workload access to 
Google resources.

## Control plane
* Kube API Server
* Etcd - state database
* Kube scheduler - schedules which nodes new pods end up on
* Kube controller manager
* Kube cloud manager - Cloud manager interacts with GCP for other resources

## GKE Autopilot vs Standard
GKE provides two flavors of Kubernetes that differ in how much of the cluster is managed by Google 
and how much is managed by the end user. The trade off for Google managing more of the cluster is 
that there is less flexibility for the end user to configure the cluster as well.

Certified courses on K8s
* Certified K8s Admin (CKA)
* Certified K8s Application Developer (CKAD)
* Certified K8s Security Specialist (CKSS)

## Determine memory cpu constraints
1. Remove all constraints
2. Apply load and watch in monitoring
3. Determine the usage at different levels
4. Derive constraints from data gathered during tests

## Pod
### Init containers

## Networking
* Node CIDR
* Pod CIDR
* Service CIDR

### Service
* Cluster IP (pod to pod i.e. inside pod)
  * Default if left out
  * Fixed name
  * port: this is the port to connect to on the service
  * targetPort: port the service will route to on the pods
  * no load balancing capability by itself
* Node Port (outside cluster)
  * assumes you have something outside the cluster that can manage this
  * nodePort: opens this port on every node in the cluster to the outside
  * port: the port that the external load balancer thinks its routing to via the node port (can be ignored)
  * targetPort: 
* Load Balancer 
  * Service that asks the environment to build a load balancer for us i.e. external GCP component
  * don't need to specify the node port but can if you want to
  * uses `port:` on the external load balancer

### DNS
Can choose on cluster DNS only useful for a single cluster or Google Cloud DNS which is useful across 
different clusters.

### Endpoint slices
Allow etcd to track all the IPs active the cluster

### Ingress
* don't use a load balancer service use a node port instead
* creates an external HTTP(S) global application load balancer
* Google Cloud Armor is a WAP for DDoS protection

### Container Native Load Balancing
* Google Cloud Platform feature only
* Load balancer knows where all the pods are and routes directly
* NEG Network Endpoint Group is what is used to track the container IPs

### Gateways
Next generation of Ingress technology i.e. anything you can do with an Ingress you can do with a 
Gateway and much more.

* uses routes
* similar concept as Istio's gateway
* original ingress was single cluster
* allows to choose different load balancer options
* gateway class is the type of load balancer

## Persistent Data and Storage

### Volumes
Ephemeral or persistent storage

* ephemeral storage
  * storage on the local node cleaned up when pod is destroyed or crashes
  * requests and limits for local empty dir constraints

* persistent storage
  * not hard drive but rather block storage in big storage array
  * persists across pod restarts
  * configmap
  * secrets
  * ReadWriteOnce - only one pod can read and write
  * ReadOnlyMany - multiple pods can read
  * ReadWriteMany - multiple pods can read and write

## Prometheus
* GCP now allows you to enable Prometheus scraping into Cloud Monitoring
* Grafana can pull from Monarch i.e. Google's Cloud Monitoring tool
* Monarch knows PromQL

<!-- 
vim: ts=2:sw=2:sts=2
-->
