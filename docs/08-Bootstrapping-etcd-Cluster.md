# Bootstrapping the etcd Cluster

## Download the etcd 
```
wget -q --show-progress --https-only --timestamping "https://github.com/etcd-io/etcd/releases/download/v3.4.0/etcd-v3.4.0-linux-amd64.tar.gz"
```
## Installing the etcd
```
  tar -xvf etcd-v3.4.0-linux-amd64.tar.gz
  mv etcd-v3.4.0-linux-amd64/etcd* /usr/local/bin/
```
## Configure the etcd Server
```
  mkdir -p /etc/etcd /var/lib/etcd
  cp ca.crt apiserver.crt apiserver.key /etc/etcd/
```
## Variables needed on each controller

INTERNAL_IP=10.12.91.206
ETCD_NAME=$(hostname -s)

INTERNAL_IP=10.12.91.148
ETCD_NAME=$(hostname -s)

INTERNAL_IP=10.12.91.175
ETCD_NAME=$(hostname -s)

## ON EACH CONTROLLER-0-1-2, to create the systemd information to start the etcd service
```
cat <<EOF | tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
ExecStart=/usr/local/bin/etcd \\
  --name ${ETCD_NAME} \\
  --cert-file=/etc/etcd/apiserver.crt \\
  --key-file=/etc/etcd/apiserver.key \\
  --peer-cert-file=/etc/etcd/apiserver.crt \\
  --peer-key-file=/etc/etcd/apiserver.key \\
  --trusted-ca-file=/etc/etcd/ca.crt \\
  --peer-trusted-ca-file=/etc/etcd/ca.crt \\
  --peer-client-cert-auth \\
  --client-cert-auth \\
  --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \\
  --advertise-client-urls https://${INTERNAL_IP}:2379 \\
  --initial-cluster-token etcd-cluster-0 \\
  --initial-cluster controller-0=https://10.12.91.206:2380,controller-1=https://10.12.91.148:2380,controller-2=https://10.12.91.175:2380 \\
  --initial-cluster-state new \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```


#Start the etcd Server
```
  systemctl daemon-reload
  systemctl enable etcd
  systemctl start etcd
```


## Verification
### List the etcd cluster members:
```
sudo ETCDCTL_API=3 etcdctl member list \
  --endpoints=https://10.12.91.175:2379 \
  --cacert=/etc/etcd/ca.crt \
  --cert=/etc/etcd/apiserver.crt \
  --key=/etc/etcd/apiserver.key
```
### The Results:
```
# sudo ETCDCTL_API=3 etcdctl member list \
>   --endpoints=https://10.12.91.175:2379 \
>   --cacert=/etc/etcd/ca.crt \
>   --cert=/etc/etcd/apiserver.crt \
>   --key=/etc/etcd/apiserver.key
3673a7c549e4cddc, started, controller-2, https://10.12.91.175:2380, https://10.12.91.175:2379, false
5522dcb0ee08a0fa, started, controller-1, https://10.12.91.148:2380, https://10.12.91.148:2379, false
709d6baeb9a63ba5, started, controller-0, https://10.12.91.206:2380, https://10.12.91.206:2379, false
```
If you see a result similar to this, then the etcd configuration went well, and the etcd service is now running as expected. 

# 
Return to: [main menu](https://github.com/jimenezcorzo/Kubernetes-The-Hard-Way-15.3-LXC/blob/master/Readme.md)
