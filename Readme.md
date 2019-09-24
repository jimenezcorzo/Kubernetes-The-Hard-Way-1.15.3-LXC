# Kubernetes The Hard Way v15.3 running on LXC containers

This tutorial will guide you through the creation of a Kubernetes v15.3 cluster with High Availability scheme called ***stacked control plane nodes***, with three Master nodes and three worker nodes, following the long route, that's because the name, "The Hard way". 

I did this lab as part of my studies to achieve the **Kubernetes Certified Administrator KCA** https://www.cncf.io/certification/cka/ in order to obtain a deep understanding of how Kubernetes works internally, and the TLS certificates requirements to communicate all kubernetes components. 

> You should not develop a similar environment for production purposes, *happy learning!* 

**Note:** It is important to mention that I did this tutotrial, taking as reference two main previous works:
1. [kelseyhightower kubernetes-the-hard-way](https://github.com/kelseyhightower/kubernetes-the-hard-way) Oriented to **Google Cloud Platform.**
2. [Justmeandopensource video part 1](https://youtu.be/NvQY5tuxALY?list=PL34sAs7_26wNBRWM6BDhnonoA5FMERax0) Oriented to LXC Bare Metal - *Explanation Video* 
3. [Justmeandopensource video part 2](https://youtu.be/2bVK-e-GuYI?list=PL34sAs7_26wNBRWM6BDhnonoA5FMERax0) *Hands On Video* 

## My contribution 
I am doing this tutorial with latest version of Kubernetes available at the creation of this lab **v1.15.3**, using **openssl tool** instead the cfssl and cfssljson tools, to generate the TLS certificates, and also I am doing the lab using **LXC containers** instead of Virtual Machines, or any commercial Cloud Platform. 

## Kubernetes Cluster details

**Baremetal Host**
- OS: Ubuntu 19.04
- LXC: 3.17
- Containers running: ubuntu/18.04

**Kubernetes Cluster**
- Kubernetes components: v15.3, [kube-apiserver](https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kube-apiserver), [kube-controller-manager](https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kube-controller-manager), [kube-scheduler](https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kube-scheduler), [kubectl](https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kubectl), [kube-proxy](https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kube-proxy), [kubelet](https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kubelet)
- containerd: [v1.2.9](https://github.com/containerd/containerd/releases/download/v1.2.9/containerd-1.2.9.linux-amd64.tar.gz)
- Container Runtime: [v1.0.0-rc8](https://github.com/opencontainers/runc/releases/download/v1.0.0-rc8/runc.amd64)
- gVisor: [50c283b9f56bb7200938d9e207355f05f79f0d17](https://storage.googleapis.com/kubernetes-the-hard-way/runsc-50c283b9f56bb7200938d9e207355f05f79f0d17)
- CNI Container Networking:  [v0.6.0](https://github.com/containernetworking/plugins/releases/download/v0.6.0/cni-plugins-amd64-v0.6.0.tgz)
- etcd: [v3.4.0](https://github.com/etcd-io/etcd/releases/download/v3.4.0/etcd-v3.4.0-linux-amd64.tar.gz)
- CoreDNS: [v1.2.2](https://storage.googleapis.com/kubernetes-the-hard-way/coredns.yaml)

## Main Tasks Lists

- [Prerequisites](https://github.com/jimenezcorzo/Kubernetes-The-Hard-Way-15.3-LXC/blob/master/docs/01-Requisites.md)
- [The Stacked Control Plane nodes for HA ](https://github.com/jimenezcorzo/Kubernetes-The-Hard-Way-15.3-LXC/blob/master/docs/02-Stacked-Control-Plane.md)
- [Provisioning the LXC Containers](https://github.com/jimenezcorzo/Kubernetes-The-Hard-Way-15.3-LXC/blob/master/docs/03-Provisioning-LXC-Containers.md)
- [Installing and configuring the HAPROXY LoadBalancer](https://github.com/jimenezcorzo/Kubernetes-The-Hard-Way-15.3-LXC/blob/master/docs/04-Installing-configuring-HAPROXY-LoadBalancer.md)
- [Provisioning the CA and Generating TLS Certificates](https://github.com/jimenezcorzo/Kubernetes-The-Hard-Way-15.3-LXC/blob/master/docs/05-Provisioning-CA-Generating-TLS-Certificates.md)
- [Generating Kubernetes Configuration Files for Authentication](https://github.com/jimenezcorzo/Kubernetes-The-Hard-Way-15.3-LXC/blob/master/docs/06-Generating-Kubernetes-Configuration-Files.md)
- [Generating the Data Encryption Config and Key](https://github.com/jimenezcorzo/Kubernetes-The-Hard-Way-15.3-LXC/blob/master/docs/07-Generating-Data-Encryption-Config-Key.md)
- [Bootstrapping the etcd Cluster](https://github.com/jimenezcorzo/Kubernetes-The-Hard-Way-15.3-LXC/blob/master/docs/08-Bootstrapping-etcd-Cluster.md)
- [Bootstrapping the Kubernetes Control Plane](https://github.com/jimenezcorzo/Kubernetes-The-Hard-Way-15.3-LXC/blob/master/docs/09-Bootstrapping-Kubernetes-Control-Plane.md)
- [Bootstrapping the Kubernetes Worker Nodes](https://github.com/jimenezcorzo/Kubernetes-The-Hard-Way-15.3-LXC/blob/master/docs/10-Bootstrapping-Kubernetes-Worker-Nodes.md)
- [Configuring kubectl for Remote Access](https://github.com/jimenezcorzo/Kubernetes-The-Hard-Way-15.3-LXC/blob/master/docs/11-Configuring-kubectl-Remote-Access.md)
- [Provisioning Pod Network Routes](https://github.com/jimenezcorzo/Kubernetes-The-Hard-Way-15.3-LXC/blob/master/docs/12-Provisioning-Pod-Network-Routes.md)
- [Kubernetes testing tasks](https://github.com/jimenezcorzo/Kubernetes-The-Hard-Way-15.3-LXC/blob/master/docs/13-Kubernetes-testing-tasks.md)
- [Some helpful troubleshooting commands](https://github.com/jimenezcorzo/Kubernetes-The-Hard-Way-15.3-LXC/blob/master/docs/14-Troubleshooting-commands.md)
- Cleaning Up
