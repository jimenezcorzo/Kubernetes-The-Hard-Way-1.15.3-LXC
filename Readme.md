# Kubernetes The Hard Way v15.3 running on LXC containers

This tutorial will guide you through the creation of a Kubernetes v15.3 cluster with High Availability scheme called ***stacked control plane nodes***, with three Master nodes and three worker nodes, following the long route, that's because the name, "The Hard way". 

I did this lab as part of my studies to achieve the **Kubernetes Certified Administrator KCA** https://www.cncf.io/certification/cka/ in order to obtain a deep understanding of how Kubernetes works internally, and the TLS certificates requirements to communicate all kubernetes components. 

> You should not develop a similar environment for production purposes, *happy learning!* 

## Kubernetes Cluster details

**Baremetal Host**
OS: Ubuntu 19.04
LXC: 3.17

**Kubernetes Cluster**
Kubernetes components: v15.3
containerd: v1.2.9 
Container Runtime: v1.0.0-rc8
gVisor: 50c283b9f56bb7200938d9e207355f05f79f0d17
CNI Container Networking:  v0.6.0
etcd: v3.4.0
CoreDNS: v1.2.2

