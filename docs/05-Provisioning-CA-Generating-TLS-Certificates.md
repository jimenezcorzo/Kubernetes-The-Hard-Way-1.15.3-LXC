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
\
Also, we need to create our own CA Certificate Authority. 

## The process of generating a TSL certificate using openssl

The process has 3 stages identified:
1. **Private key** generation based on 2048 bits
2. Creating the *Certificate Signing Request csr* file, not used on the kubernetes components, just an intermadiate step.
3. Generating the **Public certificate**, using the information identified on the csr phase

**Note 1:** Some cases the amount of information neeed to generate the csr and crt files are big, then we can use info.config
\
**Note 2:** By convention the name of the certificate **.crt** is usually the **public certificate**, and **.key** is the **private key**. 
\
**Note 3:** Keep in mind that communication between workers and apiserver is trought the load balancer, in this case apiserverlb container, this is the reason the IP is pointed by certificates instead each controller ip addresses.

## The Certificate Authority

Absolutely needed to generate the rest of TLS certificates. For CA generation, *no CSR stage is needed*:

```
openssl genrsa -out ca.key 2048
openssl req -x509 -new -nodes -key ca.key -subj "/CN=Kubernetes" -days 3650 -out ca.crt
```
output:
```
ca.key
ca.crt
```

We can verify the information encoded in the certificate at any moment with the command:
```
openssl x509 -in ca.crt -text -noout
```
Fragment of the certificate main information:
```
# openssl x509 -in ca.crt -text -noout
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            5e:fa:bd:2a:fa:08:1e:ae:88:7a:4d:cb:a5:80:68:6b:c7:7a:02:fe
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN = Kubernetes
        Validity
            Not Before: Sep  8 18:23:46 2019 GMT
            Not After : Sep  5 18:23:46 2029 GMT
        Subject: CN = Kubernetes
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                RSA Public-Key: (2048 bit)
                Modulus:
                    00:bd:f1:96:fd:b1:ed:3f:ed:33:5e:6d:25:81:4c:
***
```

## Admin:
Used to enable an Admin to access the kubernetes cluster and process any kubectl command. 
```
openssl genrsa -out admin.key 2048
openssl req -new -key admin.key -subj "/CN=admin/O=system:masters/OU=Kubernetes" -out admin.csr
openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -CAcreateserial -days 365 -out admin.crt
```
output:
```
admin.key
admin.csr
admin.crt
```
## The Worker nodes
The XLC service assigned the IP Addresses using its DHCP server. The current assignation: 
```
+--------------+---------+----------------------+-----------------------------------------------+------------+-----------+
| worker-0     | RUNNING | 10.12.91.178 (eth0)  | fd42:dad8:7f46:dd67:216:3eff:fe5b:d7c9 (eth0) | PERSISTENT | 0         |
+--------------+---------+----------------------+-----------------------------------------------+------------+-----------+
| worker-1     | RUNNING | 10.12.91.207 (eth0)  | fd42:dad8:7f46:dd67:216:3eff:fe65:da9a (eth0) | PERSISTENT | 0         |
+--------------+---------+----------------------+-----------------------------------------------+------------+-----------+
| worker-2     | RUNNING | 10.12.91.233 (eth0)  | fd42:dad8:7f46:dd67:216:3eff:fe58:9f65 (eth0) | PERSISTENT | 0         |
+--------------+---------+----------------------+-----------------------------------------------+------------+-----------+
```

### Node 0:

#### Creating the information file: csr-worker-0.cnf
```
[ req ]
default_bits = 2048
prompt = no
default_md = sha256
req_extensions = req_ext
distinguished_name = dn

[ dn ]
C = MX
ST = Campeche
L = Carmen
O = system:nodes
OU = Kubernetes
CN = system:node:worker-0

[ req_ext ]
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = worker-0
IP.1 = 10.12.91.178
IP.2 = 127.0.0.1

[ v3_ext ]
authorityKeyIdentifier=keyid,issuer:always
basicConstraints=CA:FALSE
keyUsage=keyEncipherment,dataEncipherment
extendedKeyUsage=serverAuth,clientAuth
subjectAltName=@alt_names
```
Generating the certificates
```
openssl genrsa -out worker-0.key 2048
openssl req -new -key worker-0.key -out worker-0.csr -config csr-worker-0.cnf
openssl x509 -req -in worker-0.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out worker-0.crt  -days 365 -extensions v3_ext -extfile csr-worker-0.cnf
```
output:
```
worker-0.key
worker-0.csr
worker-0.crt
```

### Node 1:
#### Creating the information file: csr-worker-1.cnf
```
[ req ]
default_bits = 2048
prompt = no
default_md = sha256
req_extensions = req_ext
distinguished_name = dn

[ dn ]
C = MX
ST = Campeche
L = Carmen
O = system:nodes
OU = Kubernetes
CN = system:node:worker-1

[ req_ext ]
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = worker-1
IP.1 = 10.12.91.207
IP.2 = 127.0.0.1

[ v3_ext ]
authorityKeyIdentifier=keyid,issuer:always
basicConstraints=CA:FALSE
keyUsage=keyEncipherment,dataEncipherment
extendedKeyUsage=serverAuth,clientAuth
subjectAltName=@alt_names
```
Generating the certificates
```
openssl genrsa -out worker-1.key 2048
openssl req -new -key worker-1.key -out worker-1.csr -config csr-worker-1.cnf
openssl x509 -req -in worker-1.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out worker-1.crt  -days 365 -extensions v3_ext -extfile csr-worker-1.cnf
```
output:
```
worker-1.key
worker-1.csr
worker-1.crt
```
### Node 2:
#### Creating the information file: csr-worker-2.cnf
```
[ req ]
default_bits = 2048
prompt = no
default_md = sha256
req_extensions = req_ext
distinguished_name = dn

[ dn ]
C = MX
ST = Campeche
L = Carmen
O = system:nodes
OU = Kubernetes
CN = system:node:worker-2

[ req_ext ]
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = worker-2
IP.1 = 10.12.91.233
IP.2 = 127.0.0.1

[ v3_ext ]
authorityKeyIdentifier=keyid,issuer:always
basicConstraints=CA:FALSE
keyUsage=keyEncipherment,dataEncipherment
extendedKeyUsage=serverAuth,clientAuth
subjectAltName=@alt_names
```
Generating the certificates
```
openssl genrsa -out worker-2.key 2048
openssl req -new -key worker-2.key -out worker-2.csr -config csr-worker-2.cnf
openssl x509 -req -in worker-2.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out worker-2.crt  -days 365 -extensions v3_ext -extfile csr-worker-2.cnf
```
output:
```
worker-2.key
worker-2.csr
worker-2.crt
```


## Controllers:
```
openssl genrsa -out kube-controller-manager.key 2048
openssl req -new -key kube-controller-manager.key -subj "/CN=system:kube-controller-manager/O=system:kube-controller-manager/OU=Kubernetes" -out kube-controller-manager.csr
openssl x509 -req -in kube-controller-manager.csr -CA ca.crt -CAkey ca.key -CAcreateserial -days 365 -out kube-controller-manager.crt
```
output:
```
kube-controller-manager.key
kube-controller-manager.csr
kube-controller-manager.crt
```

## Proxy:
```
openssl genrsa -out kube-proxy.key 2048
openssl req -new -key kube-proxy.key -subj "/CN=system:kube-proxy/O=system:node-proxier/OU=Kubernetes" -out kube-proxy.csr
openssl x509 -req -in kube-proxy.csr -CA ca.crt -CAkey ca.key -CAcreateserial -days 365 -out kube-proxy.crt
```
output:
```
kube-proxy.key
kube-proxy.csr
kube-proxy.crt
```

## Scheduler:
```
openssl genrsa -out kube-scheduler.key 2048
openssl req -new -key kube-scheduler.key -subj "/CN=system:kube-scheduler/O=system:kube-scheduler/OU=Kubernetes" -out kube-scheduler.csr
openssl x509 -req -in kube-scheduler.csr -CA ca.crt -CAkey ca.key -CAcreateserial -days 365 -out kube-scheduler.crt
```
output:
```
kube-scheduler.key
kube-scheduler.csr
kube-scheduler.crt
```

## APISERVER
The most complete certificate, because all the components talks with the apiserver.
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
```
#### Creating the information file: Creating the csr.conf 
```
[ req ]
default_bits = 2048
prompt = no
default_md = sha256
req_extensions = req_ext
distinguished_name = dn

[ dn ]
C = MX
ST = Campeche
L = Carmen
O = Kubernetes
OU = Kubernetes
CN = kubernetes

[ req_ext ]
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = kubernetes
DNS.2 = kubernetes.default
DNS.3 = kubernetes.default.svc
DNS.4 = kubernetes.default.svc.cluster
DNS.5 = kubernetes.default.svc.cluster.local
IP.1 = 10.12.91.252
IP.2 = 10.12.91.206
IP.3 = 10.12.91.148
IP.4 = 10.12.91.175

[ v3_ext ]
authorityKeyIdentifier=keyid,issuer:always
basicConstraints=CA:FALSE
keyUsage=keyEncipherment,dataEncipherment
extendedKeyUsage=serverAuth,clientAuth
subjectAltName=@alt_names
```
### Generating the apiserver certificates
```
openssl genrsa -out apiserver.key 2048
openssl req -new -key apiserver.key -out apiserver.csr -config csr.conf
openssl x509 -req -in apiserver.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out apiserver.crt -days 365 -extensions v3_ext -extfile csr.conf
```
output:
```
apiserver.key
apiserver.csr
apiserver.crt
```

## Service Account:
```
openssl genrsa -out sa.key 2048
openssl req -new -key sa.key -subj "/CN=service-accounts/O=kubernetes/OU=Kubernetes" -out sa.csr
openssl x509 -req -in sa.csr -CA ca.crt -CAkey ca.key -CAcreateserial -days 365 -out sa.crt
```
output:
```
sa.key
sa.csr
sa.crt
```
## Transfering the certificates into the LXC Controller Managers containers

We can do this using ssh, enabling the **ssh** server for each controller. In my case, I prefered to use the *lxc file push* command to transfer the recently created certificates, to each controller. 
```
lxc file push ca.crt worker-0.key worker-0.crt worker-0/root/
lxc file push ca.crt worker-1.key worker-1.crt worker-1/root/
lxc file push ca.crt worker-2.key worker-2.crt worker-2/root/
lxc file push ca.crt ca.key apiserver.crt apiserver.key sa.crt sa.key controller-0/root/
lxc file push ca.crt ca.key apiserver.crt apiserver.key sa.crt sa.key controller-1/root/
lxc file push ca.crt ca.key apiserver.crt apiserver.key sa.crt sa.key controller-2/root/
```

We are ready now, with all the TLS certificates that the kubernetes components will be using to comunicate between each other. 

# 
Return to: [main menu](https://github.com/jimenezcorzo/Kubernetes-The-Hard-Way-15.3-LXC/blob/master/Readme.md)
