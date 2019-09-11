# Provisioning the CA and Generating TLS Certificates

As I mentioned on the Readme of this repository, I am using **openssl** tool to create the needed TLS certificates, because the openssl is included in the Ubuntu OS by default, so, no needs to install the cfssl and cfssljson tools, and with the benefit of know another method to create the certificates. 

## The controller managers nodes

The XLC service assigned the IP Addresses using its DHCP server. The current assignation: 
```
+--------------+---------+----------------------+-----------------------------------------------+------------+-----------+
| controller-0 | RUNNING | 10.12.91.206 (eth0)  | fd42:dad8:7f46:dd67:216:3eff:fec9:7cbc (eth0) | PERSISTENT | 0         |
+--------------+---------+----------------------+-----------------------------------------------+------------+-----------+
| controller-1 | RUNNING | 10.12.91.148 (eth0)  | fd42:dad8:7f46:dd67:216:3eff:fe17:311a (eth0) | PERSISTENT | 0         |
+--------------+---------+----------------------+-----------------------------------------------+------------+-----------+
| controller-2 | RUNNING | 10.12.91.175 (eth0)  | fd42:dad8:7f46:dd67:216:3eff:fec3:50c4 (eth0) | PERSISTENT | 0         |
+--------------+---------+----------------------+-----------------------------------------------+------------+-----------+
```
## The needed TLS Certificates
We need to generate TLS certificates for the following components: 
- etcd
- kube-apiserver
- kube-controller-manager
- kube-scheduler
- kubelet
- kube-proxy.
An also, we need to create our own CA Certificate Authority. 

## The process of generating a TSL certificate using openssl
The process has 3 stages identified:
1. **Private key** generation based on 2048 bits
2. Creating the *Certificate Signing Request csr* file
3. Generating the **Public certificate**, using the information identified on the csr phase

**Note 1:** Some cases the amount of information neeed to generate the csr and crt files are big, then we can use info.config
**Note 2:** By convention the name of the certificate **.crt** is usually the **public certificate**, and **.key** is the **private key**.

## The Certificate Authority

Absolutely needed to generate the rest of TLS certificates. For CA generation, no CSR stage is needed:

```
openssl genrsa -out ca.key 2048
openssl req -x509 -new -nodes -key ca.key -subj "/CN=Kubernetes" -days 3650 -out ca.crt
```
output:
```
ca.key
ca.crt
```


# 
Return to: [main menu](https://github.com/jimenezcorzo/Kubernetes-The-Hard-Way-15.3-LXC/blob/master/Readme.md)
