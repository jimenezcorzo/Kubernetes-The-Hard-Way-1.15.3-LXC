# Provisioning the LXC Containers

As we identified on the previous step, we will need to create 7 running containers:
- 1 LoadBalancer running HAPROXY
- 3 Controller Managers
- 3 Worker nodes

I suggest creating first a LXC profile, because our containers need to be priviledged and have some other settings in place to work properly. Let's do that:

```
# lxc profile list
+------------+---------+
|    NAME    | USED BY |
+------------+---------+
| default    | 0       |
+------------+---------+

```
By default, only a generic profile named default is created. Creating our own Kubernetes profile:

- Copy the default profile to a new one named **Kubernetes**
```
# lxc profile cp default Kubernetes
```
- Edit the profile Kubernetes
```
# lxc profile edit Kubernetes
```
Replace the content of the Kubernetes profile with:
```
config:
  limits.memory: 2GB
  limits.memory.swap: "false"
  raw.lxc: "lxc.apparmor.profile=unconfined\nlxc.cap.drop= \nlxc.cgroup.devices.allow=a\nlxc.mount.auto=proc:rw
    sys:rw"
  security.nesting: "true"
  security.privileged: "true"
description: LXD profile for Kubernetes
devices:
  eth0:
    name: eth0
    nictype: bridged
    parent: lxdbr0
    type: nic
  root:
    path: /
    pool: default
    type: disk
name: Kubernetes
used_by: []
```
Save changes and list the LXC profiles:
```
# lxc profile list
+----------------+---------+
|      NAME      | USED BY |
+----------------+---------+
| Kubernetes     | 0       |
+----------------+---------+
| default        | 0       |
+----------------+---------+
```
Now we are ready to launch the needed containers.

List the existing containers:
```
# lxc list
+--------------+---------+----------------------+-----------------------------------------------+------------+-----------+
|     NAME     |  STATE  |         IPV4         |                     IPV6                      |    TYPE    | SNAPSHOTS |
+--------------+---------+----------------------+-----------------------------------------------+------------+-----------+
```

## Creating the LXC containers

Creating the Controllers:
```
# lxc launch images:ubuntu/18.04 controller-0 --profile Kubernetes
Creating controller-0
Starting controller-0                       
# 
# lxc launch images:ubuntu/18.04 controller-1 --profile Kubernetes
Creating controller-1
Starting controller-1                       
# 
# lxc launch images:ubuntu/18.04 controller-2 --profile Kubernetes
Creating controller-2
Starting controller-2                      
# 
```
Creating the Worker nodes:
```
# lxc launch images:ubuntu/18.04 worker-0 --profile Kubernetes
Creating worker-0
Starting worker-0                       
# 
# lxc launch images:ubuntu/18.04 worker-1 --profile Kubernetes
Creating worker-1
Starting worker-1                       
# 
# lxc launch images:ubuntu/18.04 worker-2 --profile Kubernetes
Creating worker-2
Starting worker-2                      
# 
```
Finally, creating the Load Balancer container:
```
# lxc launch images:ubuntu/18.04 apiserverlb --profile Kubernetes
Creating apiserverlb
Starting apiserverlb                       
# 
```
Listing the created containers:
```
# lxc list
+--------------+---------+----------------------+-----------------------------------------------+------------+-----------+
|     NAME     |  STATE  |         IPV4         |                     IPV6                      |    TYPE    | SNAPSHOTS |
+--------------+---------+----------------------+-----------------------------------------------+------------+-----------+
| apiserverlb  | RUNNING | 10.12.91.252 (eth0)  | fd42:dad8:7f46:dd67:216:3eff:feca:5947 (eth0) | PERSISTENT | 0         |
+--------------+---------+----------------------+-----------------------------------------------+------------+-----------+
| controller-0 | RUNNING | 10.12.91.206 (eth0)  | fd42:dad8:7f46:dd67:216:3eff:fec9:7cbc (eth0) | PERSISTENT | 0         |
+--------------+---------+----------------------+-----------------------------------------------+------------+-----------+
| controller-1 | RUNNING | 10.12.91.148 (eth0)  | fd42:dad8:7f46:dd67:216:3eff:fe17:311a (eth0) | PERSISTENT | 0         |
+--------------+---------+----------------------+-----------------------------------------------+------------+-----------+
| controller-2 | RUNNING | 10.12.91.175 (eth0)  | fd42:dad8:7f46:dd67:216:3eff:fec3:50c4 (eth0) | PERSISTENT | 0         |
+--------------+---------+----------------------+-----------------------------------------------+------------+-----------+
| worker-0     | RUNNING | 10.12.91.178 (eth0)  | fd42:dad8:7f46:dd67:216:3eff:fe5b:d7c9 (eth0) | PERSISTENT | 0         |
+--------------+---------+----------------------+-----------------------------------------------+------------+-----------+
| worker-1     | RUNNING | 10.12.91.207 (eth0)  | fd42:dad8:7f46:dd67:216:3eff:fe65:da9a (eth0) | PERSISTENT | 0         |
+--------------+---------+----------------------+-----------------------------------------------+------------+-----------+
| worker-2     | RUNNING | 10.12.91.233 (eth0)  | fd42:dad8:7f46:dd67:216:3eff:fe58:9f65 (eth0) | PERSISTENT | 0         |
+--------------+---------+----------------------+-----------------------------------------------+------------+-----------+
```

We are ready now with the needed infrastructure!


# 
Return to: [main menu](https://github.com/jimenezcorzo/Kubernetes-The-Hard-Way-15.3-LXC/blob/master/Readme.md)
