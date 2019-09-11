# Bootstrapping the Kubernetes Control Plane

This task is a bit long, and includes the Provision of the Kubernetes Control Plane - Controller Managers (Master nodes)-.

**Note:** The Following tasks need to be executed on the *three Controller Managers*. You can use tmux or manually. 

## Download and Install the Kubernetes Controller Binaries

>**On controller-0, controller-1 and controller-2**
\
Create the main config folder:
```
mkdir -p /etc/kubernetes/config
```
Download the binaries, kube-apiserver, kube-controller-manager, kube-scheduler and kubectl.
```
wget -q --show-progress --https-only --timestamping \
  "https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kube-apiserver" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kube-controller-manager" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kube-scheduler" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kubectl"
```
Assign execution permission and move them to /usr/local/bin   (Install them)
```
  chmod +x kube-apiserver kube-controller-manager kube-scheduler 
  mv kube-apiserver kube-controller-manager kube-scheduler /usr/local/bin/
```
Create the folder for TLS certificates:
```
  mkdir -p /var/lib/kubernetes/
```
Move the Certificates to the created folder:
```
  mv ca.crt ca.key apiserver.crt apiserver.key \
    sa.crt sa.key \
    encryption-config.yaml /var/lib/kubernetes/
```

Generate the variable:
```
INTERNAL_IP=USED FOR EACH IP CONTROLLER
```
Create the systemd service file:
```
cat <<EOF | tee /etc/systemd/system/kube-apiserver.service
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-apiserver \\
  --advertise-address=${INTERNAL_IP} \\
  --allow-privileged=true \\
  --apiserver-count=3 \\
  --audit-log-maxage=30 \\
  --audit-log-maxbackup=3 \\
  --audit-log-maxsize=100 \\
  --audit-log-path=/var/log/audit.log \\
  --authorization-mode=Node,RBAC \\
  --bind-address=0.0.0.0 \\
  --client-ca-file=/var/lib/kubernetes/ca.crt \\
  --enable-admission-plugins=NamespaceLifecycle,NodeRestriction,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota \\
  --enable-swagger-ui=true \\
  --etcd-cafile=/var/lib/kubernetes/ca.crt \\
  --etcd-certfile=/var/lib/kubernetes/apiserver.crt \\
  --etcd-keyfile=/var/lib/kubernetes/apiserver.key \\
  --etcd-servers=https://10.12.91.206:2379,https://10.12.91.148:2379,https://10.12.91.175:2379 \\
  --event-ttl=1h \\
  --experimental-encryption-provider-config=/var/lib/kubernetes/encryption-config.yaml \\
  --kubelet-certificate-authority=/var/lib/kubernetes/ca.crt \\
  --kubelet-client-certificate=/var/lib/kubernetes/apiserver.crt \\
  --kubelet-client-key=/var/lib/kubernetes/apiserver.key \\
  --kubelet-https=true \\
  --runtime-config=api/all \\
  --service-account-key-file=/var/lib/kubernetes/sa.key \\
  --service-cluster-ip-range=10.32.0.0/24 \\
  --service-node-port-range=30000-32767 \\
  --tls-cert-file=/var/lib/kubernetes/apiserver.crt \\
  --tls-private-key-file=/var/lib/kubernetes/apiserver.key \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Configure the Kubernetes Controller Manager

Put in place the kubeconfig file:
```
 mv kube-controller-manager.kubeconfig /var/lib/kubernetes/
```
Create the systemd service file:
```
cat <<EOF | tee /etc/systemd/system/kube-controller-manager.service
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-controller-manager \\
  --address=0.0.0.0 \\
  --cluster-cidr=10.200.0.0/16 \\
  --cluster-name=kubernetes \\
  --cluster-signing-cert-file=/var/lib/kubernetes/ca.crt \\
  --cluster-signing-key-file=/var/lib/kubernetes/ca.key \\
  --kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig \\
  --leader-elect=true \\
  --root-ca-file=/var/lib/kubernetes/ca.crt \\
  --service-account-private-key-file=/var/lib/kubernetes/sa.key \\
  --service-cluster-ip-range=10.32.0.0/24 \\
  --use-service-account-credentials=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```




Configure the Kubernetes Scheduler
```
mv kube-scheduler.kubeconfig /var/lib/kubernetes/
```

Create the kube-scheduler.yaml configuration file:
```
cat <<EOF | tee /etc/kubernetes/config/kube-scheduler.yaml
apiVersion: kubescheduler.config.k8s.io/v1alpha1
kind: KubeSchedulerConfiguration
clientConnection:
  kubeconfig: "/var/lib/kubernetes/kube-scheduler.kubeconfig"
leaderElection:
  leaderElect: true
EOF
```
Create the systemd service file:
```
cat <<EOF | tee /etc/systemd/system/kube-scheduler.service
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-scheduler \\
  --config=/etc/kubernetes/config/kube-scheduler.yaml \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```
## Start the Controller Services
```
  systemctl daemon-reload
  systemctl enable kube-apiserver kube-controller-manager kube-scheduler
  systemctl start kube-apiserver kube-controller-manager kube-scheduler
```
## Enable HTTP Health Checks

Install the Nginx service:
```
apt-get install -y nginx
```
Create the web service:
```
cat > kubernetes.default.svc.cluster.local <<EOF
server {
  listen      80;
  server_name kubernetes.default.svc.cluster.local;

  location /healthz {
     proxy_pass                    https://127.0.0.1:6443/healthz;
     proxy_ssl_trusted_certificate /var/lib/kubernetes/ca.crt;
  }
}
EOF
```
Put the web service configuration file on the www seb service:
```
  mv kubernetes.default.svc.cluster.local \
    /etc/nginx/sites-available/kubernetes.default.svc.cluster.local
  ln -s /etc/nginx/sites-available/kubernetes.default.svc.cluster.local /etc/nginx/sites-enabled/
```
Enable and Start the nginx service
```
systemctl enable nginx
systemctl restart nginx
```
## Verification from Baremetal HOST


### Testing kubectl connectivity with fresh created Kubernetes Cluster
```
# kubectl get cs --kubeconfig admin.kubeconfig
NAME                 STATUS    MESSAGE             ERROR
etcd-0               Healthy   {"health":"true"}   
scheduler            Healthy   ok                  
etcd-2               Healthy   {"health":"true"}   
etcd-1               Healthy   {"health":"true"}   
controller-manager   Healthy   ok           
```

### Testing the HTTP Health Checks
```
# curl -H "Host: kubernetes.default.svc.cluster.local" -i http://127.0.0.1/healthz
HTTP/1.1 200 OK
Server: nginx/1.10.3 (Ubuntu)
Date: Sun, 08 Sep 2019 21:38:29 GMT
Content-Type: text/plain; charset=utf-8
Content-Length: 2
Connection: keep-alive
X-Content-Type-Options: nosniff

ok
```
We completed the configuration of Master Nodes.

# 
Return to: [main menu](https://github.com/jimenezcorzo/Kubernetes-The-Hard-Way-15.3-LXC/blob/master/Readme.md)
