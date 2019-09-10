# Prerequisites

Let's talk about what we need...

**Baremetal Host**
- OS: Ubuntu 19.04
- LXC: 3.17

**Host Characteristics:**
- CPU: 2
- RAM: 8GB
- HD: 100GB

Checking with commands:

Ubuntu version:
```
# lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 19.04
Release:	19.04
Codename:	disco
```

Host settings:
```
# free -m
              total        used        free      shared  buff/cache   available
Mem:           7962        3959         437        1434        3566        2333
Swap:          2047          75        1972

# nproc
2

```
LXC Containers version
```
# lxc version
Client version: 3.17
Server version: 3.17
```

We won't worry on this lab about how to install LXC containers. The default configuration is enough:
```
# lxd init
...
LXD has been successfully configured.
```
Please select dir for the storage backend:
```
Name of the storage backend to use (dir or zfs) [default=dir]
```


# 
Return to: [main menu](https://github.com/jimenezcorzo/Kubernetes-The-Hard-Way-15.3-LXC/blob/master/Readme.md)
