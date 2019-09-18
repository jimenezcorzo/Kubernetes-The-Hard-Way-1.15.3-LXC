# Bootstrapping the Kubernetes Worker Nodes
## Provisioning a Kubernetes Worker Node

**Note:** The Following tasks need to be executed on the three *Worker nodes*. You can use tmux or manually.

Install the OS dependencies:
```
  apt-get update
  apt-get -y install socat conntrack ipset
```
##Disabling Swap
As per we are doing this labs using LXC Containers technology, we cannot to disable the swap on the running containers, because that depents of the host machine. We are configuring the services to ignore the swap on error. 

# 
Return to: [main menu](https://github.com/jimenezcorzo/Kubernetes-The-Hard-Way-15.3-LXC/blob/master/Readme.md)
