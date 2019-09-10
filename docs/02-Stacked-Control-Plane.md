# The Stacked Control Plane nodes for HA

Kubernetes official reference: [Stacked etcd topology](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/ha-topology/#stacked-etcd-topology)

![Image Stacked etcd topology](https://d33wubrfki0l68.cloudfront.net/d1411cded83856552f37911eb4522d9887ca4e83/b94b2/images/kubeadm/kubeadm-ha-topology-stacked-etcd.svg)

From the image above we conclude the following:
1. We need at least 3 Servers acting as Controller Managers ( *Master Nodes* ).
2. The etcd database is stacked on the same controller manager, we have 3 etcd instances running.
3. We will need a Load Balancer, in order the worker nodes can communicate with Masters.
4. For this lab, we will creating and configuring 3 worker nodes
5. In total, we will need 7 LXC containers running:
- 1 Load Balancer running HAPROXY
- 3 Controller Manager nodes (Master Nodes)
- 3 Worker nodes

We will be using the stacked control plane, due the benefit that requires less infrastructure than a external etcd cluster configuration:
> With stacked control plane nodes. This approach requires less infrastructure. The etcd members and control plane nodes are co-located.


# 
Return to: [main menu](https://github.com/jimenezcorzo/Kubernetes-The-Hard-Way-15.3-LXC/blob/master/Readme.md)
