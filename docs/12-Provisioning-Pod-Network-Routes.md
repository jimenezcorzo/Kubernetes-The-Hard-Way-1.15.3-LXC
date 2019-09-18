# Provisioning Pod Network Routes

Pods scheduled to a node receive an IP address from the node's Pod CIDR range. At this point pods can not communicate with other pods running on different nodes due to missing network routes.

In this lab you will create a route for each worker node that maps the node's Pod CIDR range to the node's internal IP address on Host machine.
```
route add -net 10.200.0.0 netmask 255.255.255.0 gw 10.12.91.178
route add -net 10.200.1.0 netmask 255.255.255.0 gw 10.12.91.207
route add -net 10.200.2.0 netmask 255.255.255.0 gw 10.12.91.233
```
This will allow communication between pods through main ip assigned to each LXC container as gateway.  

# 
Return to: [main menu](https://github.com/jimenezcorzo/Kubernetes-The-Hard-Way-15.3-LXC/blob/master/Readme.md)
