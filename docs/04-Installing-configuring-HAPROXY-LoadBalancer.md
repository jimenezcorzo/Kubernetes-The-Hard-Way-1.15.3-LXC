# Installing and configuring the HAPROXY LoadBalancer

We will use HAPROXY as the load balancer that will allow communicate the Worker nodes and the Controller Managers nodes, through port 6443 (apiserver). 

> You can review as reference the image of the stacked kubernetes configuration.

## Accessing the XLC container - apiserverlb
```
# lxc exec apiserverlb bash
# hostname
apiserverlb
# 
```

## Updating and installing HAPROXY
```
# apt-get update && apt-get upgrade
# apt-get install haproxy
```

## Configuring HPROXY service as needed

Configuration file: **/etc/haproxy/haproxy.cfg**, please leave  *global*  and *default* sections as is, and add the following at the end of the file.
```
frontend k8s
    bind 10.12.91.252:6443
    mode tcp
    default_backend k8s

backend k8s
    balance roundrobin
    mode tcp
    option tcplog
    option tcp-check
    server controller-0 10.12.91.206:6443 check
    server controller-1 10.12.91.148:6443 check
    server controller-2 10.12.91.175:6443 check
```
**Explanation:** We will recieve incoming traffic (at **tcp** protocol level) on apiserverlb container with IP: 10.12.91.252 and Port: 6443 (apiserver), and distribute it using a roundrobin algorithm, across the three containers controller-0, controller-1, and controller-2 using the same port. 

Let's start the service:
```
systemctl enable haproxy
systemctl start haproxy
systemctl status haproxy
```
## Testing the HAPROXY configuration from inside the apiserverlb container
```
# netstat -an | grep 6443
tcp        0      0 10.12.91.252:6443       0.0.0.0:*               LISTEN     
```
## Testing the HAPROXY configuration from our HOST machine
```
# telnet 10.12.91.252 6443
Trying 10.12.91.252...
Connected to 10.12.91.252.
Escape character is '^]'.

```

We have successfully configured and started the HAPROXY LoadBalancer service. 

# 
Return to: [main menu](https://github.com/jimenezcorzo/Kubernetes-The-Hard-Way-15.3-LXC/blob/master/Readme.md)
