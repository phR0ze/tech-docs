# Kubernetes

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

<!-- 
vim: ts=2:sw=2:sts=2
-->
