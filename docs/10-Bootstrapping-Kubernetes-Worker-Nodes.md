# Bootstrapping the Kubernetes Worker Nodes
## Provisioning a Kubernetes Worker Node

**Note:** The Following tasks need to be executed on the three *Worker nodes*. You can use tmux or manually.

Install the OS dependencies:
```
  apt-get update
  apt-get -y install socat conntrack ipset
```
## Disabling Swap

As per we are doing this labs using LXC Containers technology, we cannot to disable the swap on the running containers, because that depents of the host machine. We are configuring the services to ignore the swap on error. 

## Download and Install Worker Binaries
```
wget -q --show-progress --https-only --timestamping \
  https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.15.0/crictl-v1.15.0-linux-386.tar.gz \
  https://storage.googleapis.com/kubernetes-the-hard-way/runsc-50c283b9f56bb7200938d9e207355f05f79f0d17 \
  https://github.com/opencontainers/runc/releases/download/v1.0.0-rc8/runc.amd64 \
  https://github.com/containernetworking/plugins/releases/download/v0.6.0/cni-plugins-amd64-v0.6.0.tgz \
  https://github.com/containerd/containerd/releases/download/v1.2.9/containerd-1.2.9.linux-amd64.tar.gz \
  https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kubectl \
  https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kube-proxy \
  https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kubelet
```
Creating the installation folders:
```
mkdir -p \
  /etc/cni/net.d \
  /opt/cni/bin \
  /var/lib/kubelet \
  /var/lib/kube-proxy \
  /var/lib/kubernetes \
  /var/run/kubernetes
```
Installing the worker nodes binaries:
```
  mv runsc-50c283b9f56bb7200938d9e207355f05f79f0d17 runsc
  mv runc.amd64 runc
  chmod +x kubectl kube-proxy kubelet runc runsc
  mv kubectl kube-proxy kubelet runc runsc /usr/local/bin/
  tar -xvf crictl-v1.15.0-linux-386.tar.gz -C /usr/local/bin/
  tar -xvf cni-plugins-amd64-v0.6.0.tgz -C /opt/cni/bin/
  tar -xvf containerd-1.2.9.linux-amd64.tar.gz -C /
  ```
## Configure CNI Networking

Configure the POD_CIDR environment variable, one for each node:

> POD_CIDR=10.200.0.0/24
\
> POD_CIDR=10.200.1.0/24
\
> POD_CIDR=10.200.2.0/24


Create the **bridge network** configuration file:
```
cat <<EOF | sudo tee /etc/cni/net.d/10-bridge.conf
{
    "cniVersion": "0.3.1",
    "name": "bridge",
    "type": "bridge",
    "bridge": "cnio0",
    "isGateway": true,
    "ipMasq": true,
    "ipam": {
        "type": "host-local",
        "ranges": [
          [{"subnet": "${POD_CIDR}"}]
        ],
        "routes": [{"dst": "0.0.0.0/0"}]
    }
}
EOF
```
Create the **loopback network** configuration file:
```
cat <<EOF | tee /etc/cni/net.d/99-loopback.conf
{
    "cniVersion": "0.3.1",
    "type": "loopback"
}
EOF
```

## Configure containerd

Create the **containerd** configuration file:
```
mkdir -p /etc/containerd/
```
```
cat << EOF | tee /etc/containerd/config.toml
[plugins]
  [plugins.cri.containerd]
    snapshotter = "overlayfs"
    [plugins.cri.containerd.default_runtime]
      runtime_type = "io.containerd.runtime.v1.linux"
      runtime_engine = "/usr/local/bin/runc"
      runtime_root = ""
    [plugins.cri.containerd.untrusted_workload_runtime]
      runtime_type = "io.containerd.runtime.v1.linux"
      runtime_engine = "/usr/local/bin/runsc"
      runtime_root = "/run/containerd/runsc"
    [plugins.cri.containerd.gvisor]
      runtime_type = "io.containerd.runtime.v1.linux"
      runtime_engine = "/usr/local/bin/runsc"
      runtime_root = "/run/containerd/runsc"
EOF
```
Create the **containerd.service** systemd unit file:
```
cat <<EOF | tee /etc/systemd/system/containerd.service
[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target

[Service]
ExecStart=/bin/containerd
Restart=always
RestartSec=5
Delegate=yes
KillMode=process
OOMScoreAdjust=-999
LimitNOFILE=1048576
LimitNPROC=infinity
LimitCORE=infinity

[Install]
WantedBy=multi-user.target
EOF
```
## Configure the Kubelet

For worker-0:
```
  mv worker-0.key worker-0.crt /var/lib/kubelet/
  mv worker-0.kubeconfig /var/lib/kubelet/kubeconfig
  mv ca.crt /var/lib/kubernetes/
```
For worker-1:
```
  mv worker-1.key worker-1.crt /var/lib/kubelet/
  mv worker-1.kubeconfig /var/lib/kubelet/kubeconfig
  mv ca.crt /var/lib/kubernetes/
```
For worker-2:
```
  mv worker-2.key worker-2.crt /var/lib/kubelet/
  mv worker-2.kubeconfig /var/lib/kubelet/kubeconfig
  mv ca.crt /var/lib/kubernetes/
```
> **Note:** I used the Google DNS: 8.8.8.8 for Containers resolution, to avoid the need to deploy a DNS solution. 

Create the **kubelet-config.yaml** configuration file:
For worker-0:
```
cat <<EOF | tee /var/lib/kubelet/kubelet-config.yaml
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
  x509:
    clientCAFile: "/var/lib/kubernetes/ca.crt"
authorization:
  mode: Webhook
clusterDomain: "cluster.local"
clusterDNS:
  - "8.8.8.8"
podCIDR: "${POD_CIDR}"
resolvConf: "/run/systemd/resolve/resolv.conf"
runtimeRequestTimeout: "15m"
tlsCertFile: "/var/lib/kubelet/worker-0.crt"
tlsPrivateKeyFile: "/var/lib/kubelet/worker-0.key"
EOF
```
For worker-1:
```
cat <<EOF | tee /var/lib/kubelet/kubelet-config.yaml
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
  x509:
    clientCAFile: "/var/lib/kubernetes/ca.crt"
authorization:
  mode: Webhook
clusterDomain: "cluster.local"
clusterDNS:
  - "8.8.8.8"
podCIDR: "${POD_CIDR}"
resolvConf: "/run/systemd/resolve/resolv.conf"
runtimeRequestTimeout: "15m"
tlsCertFile: "/var/lib/kubelet/worker-1.crt"
tlsPrivateKeyFile: "/var/lib/kubelet/worker-1.key"
EOF
```
For worker-2:
```
cat <<EOF | tee /var/lib/kubelet/kubelet-config.yaml
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
  x509:
    clientCAFile: "/var/lib/kubernetes/ca.crt"
authorization:
  mode: Webhook
clusterDomain: "cluster.local"
clusterDNS:
  - "8.8.8.8"
podCIDR: "${POD_CIDR}"
resolvConf: "/run/systemd/resolve/resolv.conf"
runtimeRequestTimeout: "15m"
tlsCertFile: "/var/lib/kubelet/worker-2.crt"
tlsPrivateKeyFile: "/var/lib/kubelet/worker-2.key"
EOF
```
**Same on 3 Workers**, to start the systemd services:

kubelet.service
```
cat <<EOF | sudo tee /etc/systemd/system/kubelet.service
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=containerd.service
Requires=containerd.service

[Service]
ExecStart=/usr/local/bin/kubelet \\
  --config=/var/lib/kubelet/kubelet-config.yaml \\
  --container-runtime=remote \\
  --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock \\
  --image-pull-progress-deadline=2m \\
  --kubeconfig=/var/lib/kubelet/kubeconfig \\
  --network-plugin=cni \\
  --register-node=true \\
  --fail-swap-on=false \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```
## Configure the Kubernetes Proxy
```
mv kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
```
kube-proxy-config.yaml
```
cat <<EOF | tee /var/lib/kube-proxy/kube-proxy-config.yaml
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
clientConnection:
  kubeconfig: "/var/lib/kube-proxy/kubeconfig"
mode: "iptables"
clusterCIDR: "10.200.0.0/16"
EOF
```
kube-proxy.service
```
cat <<EOF | tee /etc/systemd/system/kube-proxy.service
[Unit]
Description=Kubernetes Kube Proxy
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-proxy \\
  --config=/var/lib/kube-proxy/kube-proxy-config.yaml
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```
## Start the Worker Services
```
  systemctl daemon-reload
  systemctl enable containerd kubelet kube-proxy
  systemctl start containerd kubelet kube-proxy
```
# Verification
You can run the kubectl command specifying the kubeconfig file to use:
```
kubectl get nodes --kubeconfig admin.kubeconfig
```
Or taking the time and configure your kubectl client to talk with our brand new Kubernetes cluster:

## Configure *kubectl* on the hosts:
Use the apiserverlb ipaddress, the load balancer IP Address:
```
  KUBERNETES_PUBLIC_ADDRESS=10.12.91.252
```
Create the *kubectl* configuration file for hosts:
```
  kubectl config set-cluster kubernetes \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://${KUBERNETES_PUBLIC_ADDRESS}:6443

  kubectl config set-credentials admin \
    --client-certificate=admin.crt \
    --client-key=admin.key

  kubectl config set-context kubernetes \
    --cluster=kubernetes \
    --user=admin

  kubectl config use-context kubernetes
```
Testing:
```
# kubectl get nodes
NAME       STATUS   ROLES    AGE   VERSION
worker-0   Ready    <none>   8d    v1.15.3
worker-1   Ready    <none>   8d    v1.15.3
worker-2   Ready    <none>   8d    v1.15.3
# 
```
\
\

# 
**Note:**
```
************************
If you selected Ubuntu 16.0 as the LXC OS for your nodes, then a link need to be created:

ln -s /run/resolvconf/ /run/systemd/resolve

In order to reach the file: /run/systemd/resolve/resolv.conf
************************
```

# 
Return to: [main menu](https://github.com/jimenezcorzo/Kubernetes-The-Hard-Way-15.3-LXC/blob/master/Readme.md)
