# Kubernetes Testing Tasks

The time is arrived, let's test our Kubernetes cluster, manual deployment *"The Hard Way"*. I am going to present below, a series of commands to test if our Kubernetes cluster is now working as expected.

Showing our infrastructure, Controller Manager Servers, Worker Nodes and Load Balancer server:
```
# lxc list
+--------------+---------+---------------------+-----------------------------------------------+------------+-----------+
|     NAME     |  STATE  |        IPV4         |                     IPV6                      |    TYPE    | SNAPSHOTS |
+--------------+---------+---------------------+-----------------------------------------------+------------+-----------+
| apiserverlb  | RUNNING | 10.12.91.252 (eth0) | fd42:dad8:7f46:dd67:216:3eff:feca:5947 (eth0) | PERSISTENT | 0         |
+--------------+---------+---------------------+-----------------------------------------------+------------+-----------+
| controller-0 | RUNNING | 10.12.91.206 (eth0) | fd42:dad8:7f46:dd67:216:3eff:fec9:7cbc (eth0) | PERSISTENT | 0         |
+--------------+---------+---------------------+-----------------------------------------------+------------+-----------+
| controller-1 | RUNNING | 10.12.91.148 (eth0) | fd42:dad8:7f46:dd67:216:3eff:fe17:311a (eth0) | PERSISTENT | 0         |
+--------------+---------+---------------------+-----------------------------------------------+------------+-----------+
| controller-2 | RUNNING | 10.12.91.175 (eth0) | fd42:dad8:7f46:dd67:216:3eff:fec3:50c4 (eth0) | PERSISTENT | 0         |
+--------------+---------+---------------------+-----------------------------------------------+------------+-----------+
| worker-0     | RUNNING | 10.200.0.1 (cnio0)  | fd42:dad8:7f46:dd67:216:3eff:fe5b:d7c9 (eth0) | PERSISTENT | 0         |
|              |         | 10.12.91.178 (eth0) |                                               |            |           |
+--------------+---------+---------------------+-----------------------------------------------+------------+-----------+
| worker-1     | RUNNING | 10.200.1.1 (cnio0)  | fd42:dad8:7f46:dd67:216:3eff:fe65:da9a (eth0) | PERSISTENT | 0         |
|              |         | 10.12.91.207 (eth0) |                                               |            |           |
+--------------+---------+---------------------+-----------------------------------------------+------------+-----------+
| worker-2     | RUNNING | 10.200.2.1 (cnio0)  | fd42:dad8:7f46:dd67:216:3eff:fe58:9f65 (eth0) | PERSISTENT | 0         |
|              |         | 10.12.91.233 (eth0) |                                               |            |           |
+--------------+---------+---------------------+-----------------------------------------------+------------+-----------+
# 
```
#

Checking basic information about our brand new cluster:
```
# kubectl cluster-info
Kubernetes master is running at https://10.12.91.252:6443

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

# kubectl get cs
NAME                 STATUS    MESSAGE             ERROR
controller-manager   Healthy   ok                  
scheduler            Healthy   ok                  
etcd-2               Healthy   {"health":"true"}   
etcd-0               Healthy   {"health":"true"}   
etcd-1               Healthy   {"health":"true"}   

# kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://10.12.91.252:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    namespace: paco
    user: kubernetes-admin
  name: admin
- context:
    cluster: kubernetes
    user: admin
  name: kubernetes
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes
kind: Config
preferences: {}
users:
- name: admin
  user:
    client-certificate: /root/HARD/admin.crt
    client-key: /root/HARD/admin.key
- name: kubernetes-admin
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
# 
```
#

Let's see some important difference between our manually deployed Kubernetes Cluster, and any other created with kubeadm tool:
```
# kubectl get all --all-namespaces 


NAMESPACE   NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
default     service/kubernetes   ClusterIP   10.32.0.1    <none>        443/TCP   15d

```
*As you can see, no other pods were created on kube-system namespace* as is common when we used the kubeadm tool to deploy a new Kubernetes cluster, using High Availavility or not. 

In our case, **the Kubernetes components are not running inside Pods**, they are running on *physical boxes*, also in our case, the servers are in fact **LXC containers**. 

Running a testing pod
```
# kubectl run --generator=run-pod/v1 webapp --image nginx --port 80
pod/webapp created

# kubectl get pods 
NAME     READY   STATUS    RESTARTS   AGE
webapp   1/1     Running   0          11s

# kubectl get pods -o wide
NAME     READY   STATUS    RESTARTS   AGE   IP            NODE       NOMINATED NODE   READINESS GATES
webapp   1/1     Running   0          25s   10.200.2.13   worker-2   <none>           <none>
# 
```
Testing access to pod and service running on port 80
```
# curl 10.200.2.13:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

# kubectl exec -it webapp bash
root@webapp:/# hostname 
webapp
root@webapp:/# hostname -i
10.200.2.13
root@webapp:/#
```
Deleting the testing pod
```
# kubectl get pods
NAME     READY   STATUS    RESTARTS   AGE
webapp   1/1     Running   0          5m20s

# kubectl delete pod webapp
pod "webapp" deleted

# kubectl get pods --all-namespaces 
No resources found.
# 
```
#

Testing a new deployment
```
# kubectl run webapp --image nginx:1.17.3 --port 80 --expose --replicas 3 --record
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
service/webapp created
deployment.apps/webapp created

# kubectl get all --all-namespaces 
NAMESPACE   NAME                         READY   STATUS    RESTARTS   AGE
default     pod/webapp-795bbd7d8-2wjtt   1/1     Running   0          24s
default     pod/webapp-795bbd7d8-77ggn   1/1     Running   0          24s
default     pod/webapp-795bbd7d8-7ltdh   1/1     Running   0          24s


NAMESPACE   NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
default     service/kubernetes   ClusterIP   10.32.0.1    <none>        443/TCP   15d
default     service/webapp       ClusterIP   10.32.0.75   <none>        80/TCP    24s


NAMESPACE   NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
default     deployment.apps/webapp   3/3     3            3           24s

NAMESPACE   NAME                               DESIRED   CURRENT   READY   AGE
default     replicaset.apps/webapp-795bbd7d8   3         3         3       24s


```
Checking new created pods, and corresponding IP Address assigned
```
# kubectl get pods -o wide 
NAME                     READY   STATUS    RESTARTS   AGE   IP            NODE       NOMINATED NODE   READINESS GATES
webapp-795bbd7d8-2wjtt   1/1     Running   0          47s   10.200.2.14   worker-2   <none>           <none>
webapp-795bbd7d8-77ggn   1/1     Running   0          47s   10.200.0.14   worker-0   <none>           <none>
webapp-795bbd7d8-7ltdh   1/1     Running   0          47s   10.200.1.14   worker-1   <none>           <none>
```
Checking web service using Internal IP address assigned
```
# curl 10.200.2.14
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
Changing the webapp service from ClusterIP to NodePort
```
# kubectl edit service/webapp 
service/webapp edited

# kubectl get svc 
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.32.0.1    <none>        443/TCP        15d
webapp       NodePort    10.32.0.75   <none>        80:32710/TCP   2m9s
```
Verifying the connectivity to web service using the External IP of the Worker node (we can use any of the three nodes IP address)
```
# curl 10.12.91.178:32710
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
Checking the rollout history of the deployment
```
# kubectl rollout history deployment.apps/webapp
deployment.apps/webapp 
REVISION  CHANGE-CAUSE
1         kubectl run webapp --image=nginx:1.17.3 --port=80 --expose=true --replicas=3 --record=true
```
Scaling UP the number of replicas
```
# kubectl scale deployment.apps/webapp --replicas 6
deployment.apps/webapp scaled

# kubectl get pods -o wide
NAME                     READY   STATUS    RESTARTS   AGE     IP            NODE       NOMINATED NODE   READINESS GATES
webapp-795bbd7d8-2jh5m   1/1     Running   0          7s      10.200.1.15   worker-1   <none>           <none>
webapp-795bbd7d8-2wjtt   1/1     Running   0          4m12s   10.200.2.14   worker-2   <none>           <none>
webapp-795bbd7d8-77ggn   1/1     Running   0          4m12s   10.200.0.14   worker-0   <none>           <none>
webapp-795bbd7d8-7ltdh   1/1     Running   0          4m12s   10.200.1.14   worker-1   <none>           <none>
webapp-795bbd7d8-tpnxs   1/1     Running   0          7s      10.200.0.15   worker-0   <none>           <none>
webapp-795bbd7d8-v5qqh   1/1     Running   0          7s      10.200.2.15   worker-2   <none>           <none>
```
Scaling DOWN the number of replicas
```
# kubectl scale deployment.apps/webapp --replicas 3
deployment.apps/webapp scaled

# kubectl get pods -o wide
NAME                     READY   STATUS    RESTARTS   AGE     IP            NODE       NOMINATED NODE   READINESS GATES
webapp-795bbd7d8-2wjtt   1/1     Running   0          9m27s   10.200.2.14   worker-2   <none>           <none>
webapp-795bbd7d8-77ggn   1/1     Running   0          9m27s   10.200.0.14   worker-0   <none>           <none>
webapp-795bbd7d8-7ltdh   1/1     Running   0          9m27s   10.200.1.14   worker-1   <none>           <none>
root@sbybz220101:~# 
```
Testing deployment rollout and rollback
```
# kubectl rollout history deployment.apps/webapp
deployment.apps/webapp 
REVISION  CHANGE-CAUSE
1         kubectl run webapp --image=nginx:1.17.3 --port=80 --expose=true --replicas=3 --record=true

# kubectl set image deployment.v1.apps/webapp webapp=nginx:1.17.0 --record=true
deployment.apps/webapp image updated
```
Checking the pods replacement based on the rollout strategy: RollingUpdate
```
# kubectl get pods -o wide
NAME                      READY   STATUS              RESTARTS   AGE   IP            NODE       NOMINATED NODE   READINESS GATES
webapp-795bbd7d8-2wjtt    1/1     Running             0          22m   10.200.2.14   worker-2   <none>           <none>
webapp-795bbd7d8-77ggn    1/1     Running             0          22m   10.200.0.14   worker-0   <none>           <none>
webapp-795bbd7d8-7ltdh    1/1     Running             0          22m   10.200.1.14   worker-1   <none>           <none>
webapp-8544985cc7-dnxgx   0/1     ContainerCreating   0          8s    <none>        worker-1   <none>           <none>

# kubectl get pods -o wide
NAME                      READY   STATUS              RESTARTS   AGE   IP            NODE       NOMINATED NODE   READINESS GATES
webapp-795bbd7d8-2wjtt    1/1     Running             0          22m   10.200.2.14   worker-2   <none>           <none>
webapp-795bbd7d8-77ggn    1/1     Running             0          22m   10.200.0.14   worker-0   <none>           <none>
webapp-8544985cc7-dnxgx   1/1     Running             0          17s   10.200.1.16   worker-1   <none>           <none>
webapp-8544985cc7-nnffw   0/1     ContainerCreating   0          8s    <none>        worker-0   <none>           <none>

# kubectl get pods -o wide
NAME                      READY   STATUS        RESTARTS   AGE   IP            NODE       NOMINATED NODE   READINESS GATES
webapp-795bbd7d8-2wjtt    0/1     Terminating   0          23m   10.200.2.14   worker-2   <none>           <none>
webapp-795bbd7d8-77ggn    0/1     Terminating   0          23m   <none>        worker-0   <none>           <none>
webapp-8544985cc7-9x2jf   1/1     Running       0          12s   10.200.2.16   worker-2   <none>           <none>
webapp-8544985cc7-dnxgx   1/1     Running       0          32s   10.200.1.16   worker-1   <none>           <none>
webapp-8544985cc7-nnffw   1/1     Running       0          23s   10.200.0.16   worker-0   <none>           <none>

# kubectl get pods -o wide
NAME                      READY   STATUS    RESTARTS   AGE   IP            NODE       NOMINATED NODE   READINESS GATES
webapp-8544985cc7-9x2jf   1/1     Running   0          25s   10.200.2.16   worker-2   <none>           <none>
webapp-8544985cc7-dnxgx   1/1     Running   0          45s   10.200.1.16   worker-1   <none>           <none>
webapp-8544985cc7-nnffw   1/1     Running   0          36s   10.200.0.16   worker-0   <none>           <none>

```
Verifying the rollout application
```
# kubectl rollout status deployment.v1.apps/webapp
deployment "webapp" successfully rolled out

# kubectl rollout history deployment.v1.apps/webapp
deployment.apps/webapp 
REVISION  CHANGE-CAUSE
1         kubectl run webapp --image=nginx:1.17.3 --port=80 --expose=true --replicas=3 --record=true
2         kubectl set image deployment.v1.apps/webapp webapp=nginx:1.17.0 --record=true

```
Rolling back
```
# kubectl rollout undo deployment.v1.apps/webapp
deployment.apps/webapp rolled back

# kubectl rollout status deployment.v1.apps/webapp
Waiting for deployment "webapp" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "webapp" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "webapp" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "webapp" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "webapp" rollout to finish: 1 old replicas are pending termination...
deployment "webapp" successfully rolled out

# kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
webapp-795bbd7d8-csd2c   1/1     Running   0          15s
webapp-795bbd7d8-vpwhk   1/1     Running   0          24s
webapp-795bbd7d8-xp2fj   1/1     Running   0          20s

# kubectl rollout history deployment.v1.apps/webapp
deployment.apps/webapp 
REVISION  CHANGE-CAUSE
2         kubectl set image deployment.v1.apps/webapp webapp=nginx:1.17.0 --record=true
3         kubectl run webapp --image=nginx:1.17.3 --port=80 --expose=true --replicas=3 --record=true
#
```

Testing POD DNS resolution, installing some package

```
# kubectl run --generator=run-pod/v1 testing-pod --image nginx
pod/testing-pod created

# kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
testing-pod              1/1     Running   0          5s
webapp-795bbd7d8-csd2c   1/1     Running   0          28m
webapp-795bbd7d8-vpwhk   1/1     Running   0          28m
webapp-795bbd7d8-xp2fj   1/1     Running   0          28m

# kubectl exec -it testing-pod bash
root@testing-pod:/# apt update
Get:1 http://cdn-fastly.deb.debian.org/debian buster InRelease [122 kB]             
Get:2 http://security-cdn.debian.org/debian-security buster/updates InRelease [39.1 kB]
Get:3 http://cdn-fastly.deb.debian.org/debian buster-updates InRelease [49.3 kB]
Get:4 http://cdn-fastly.deb.debian.org/debian buster/main amd64 Packages [7899 kB]
Get:5 http://security-cdn.debian.org/debian-security buster/updates/main amd64 Packages [91.7 kB]
Get:6 http://cdn-fastly.deb.debian.org/debian buster-updates/main amd64 Packages [5792 B]
Fetched 8206 kB in 4s (2256 kB/s)                    
Reading package lists... Done
Building dependency tree       
Reading state information... Done
2 packages can be upgraded. Run 'apt list --upgradable' to see them.

root@testing-pod:/# apt install vim
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following additional packages will be installed:
  libgpm2 vim-common vim-runtime xxd
Suggested packages:
  gpm ctags vim-doc vim-scripts
The following NEW packages will be installed:
  libgpm2 vim vim-common vim-runtime xxd
0 upgraded, 5 newly installed, 0 to remove and 2 not upgraded.
Need to get 7425 kB of archives.
After this operation, 33.8 MB of additional disk space will be used.
Do you want to continue? [Y/n] Y
Get:1 http://cdn-fastly.deb.debian.org/debian buster/main amd64 xxd amd64 2:8.1.0875-5 [140 kB]
Get:2 http://cdn-fastly.deb.debian.org/debian buster/main amd64 vim-common all 2:8.1.0875-5 [195 kB]
Get:3 http://cdn-fastly.deb.debian.org/debian buster/main amd64 libgpm2 amd64 1.20.7-5 [35.1 kB]
Get:4 http://cdn-fastly.deb.debian.org/debian buster/main amd64 vim-runtime all 2:8.1.0875-5 [5775 kB]
Get:5 http://cdn-fastly.deb.debian.org/debian buster/main amd64 vim amd64 2:8.1.0875-5 [1280 kB]
Fetched 7425 kB in 1s (6176 kB/s)
debconf: delaying package configuration, since apt-utils is not installed
Selecting previously unselected package xxd.
(Reading database ... 7203 files and directories currently installed.)
Preparing to unpack .../xxd_2%3a8.1.0875-5_amd64.deb ...
Unpacking xxd (2:8.1.0875-5) ...
Selecting previously unselected package vim-common.
Preparing to unpack .../vim-common_2%3a8.1.0875-5_all.deb ...
Unpacking vim-common (2:8.1.0875-5) ...
Selecting previously unselected package libgpm2:amd64.
Preparing to unpack .../libgpm2_1.20.7-5_amd64.deb ...
Unpacking libgpm2:amd64 (1.20.7-5) ...
Selecting previously unselected package vim-runtime.
Preparing to unpack .../vim-runtime_2%3a8.1.0875-5_all.deb ...
Adding 'diversion of /usr/share/vim/vim81/doc/help.txt to /usr/share/vim/vim81/doc/help.txt.vim-tiny by vim-runtime'
Adding 'diversion of /usr/share/vim/vim81/doc/tags to /usr/share/vim/vim81/doc/tags.vim-tiny by vim-runtime'
Unpacking vim-runtime (2:8.1.0875-5) ...
Selecting previously unselected package vim.
Preparing to unpack .../vim_2%3a8.1.0875-5_amd64.deb ...
Unpacking vim (2:8.1.0875-5) ...
Setting up libgpm2:amd64 (1.20.7-5) ...
Setting up xxd (2:8.1.0875-5) ...
Setting up vim-common (2:8.1.0875-5) ...
Setting up vim-runtime (2:8.1.0875-5) ...
Setting up vim (2:8.1.0875-5) ...
update-alternatives: using /usr/bin/vim.basic to provide /usr/bin/vim (vim) in auto mode
update-alternatives: using /usr/bin/vim.basic to provide /usr/bin/vimdiff (vimdiff) in auto mode
update-alternatives: using /usr/bin/vim.basic to provide /usr/bin/rvim (rvim) in auto mode
update-alternatives: using /usr/bin/vim.basic to provide /usr/bin/rview (rview) in auto mode
update-alternatives: using /usr/bin/vim.basic to provide /usr/bin/vi (vi) in auto mode
update-alternatives: warning: skip creation of /usr/share/man/da/man1/vi.1.gz because associated file /usr/share/man/da/man1/vim.1.gz (of link group vi) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/de/man1/vi.1.gz because associated file /usr/share/man/de/man1/vim.1.gz (of link group vi) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/fr/man1/vi.1.gz because associated file /usr/share/man/fr/man1/vim.1.gz (of link group vi) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/it/man1/vi.1.gz because associated file /usr/share/man/it/man1/vim.1.gz (of link group vi) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/ja/man1/vi.1.gz because associated file /usr/share/man/ja/man1/vim.1.gz (of link group vi) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/pl/man1/vi.1.gz because associated file /usr/share/man/pl/man1/vim.1.gz (of link group vi) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/ru/man1/vi.1.gz because associated file /usr/share/man/ru/man1/vim.1.gz (of link group vi) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/man1/vi.1.gz because associated file /usr/share/man/man1/vim.1.gz (of link group vi) doesn't exist
update-alternatives: using /usr/bin/vim.basic to provide /usr/bin/view (view) in auto mode
update-alternatives: warning: skip creation of /usr/share/man/da/man1/view.1.gz because associated file /usr/share/man/da/man1/vim.1.gz (of link group view) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/de/man1/view.1.gz because associated file /usr/share/man/de/man1/vim.1.gz (of link group view) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/fr/man1/view.1.gz because associated file /usr/share/man/fr/man1/vim.1.gz (of link group view) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/it/man1/view.1.gz because associated file /usr/share/man/it/man1/vim.1.gz (of link group view) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/ja/man1/view.1.gz because associated file /usr/share/man/ja/man1/vim.1.gz (of link group view) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/pl/man1/view.1.gz because associated file /usr/share/man/pl/man1/vim.1.gz (of link group view) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/ru/man1/view.1.gz because associated file /usr/share/man/ru/man1/vim.1.gz (of link group view) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/man1/view.1.gz because associated file /usr/share/man/man1/vim.1.gz (of link group view) doesn't exist
update-alternatives: using /usr/bin/vim.basic to provide /usr/bin/ex (ex) in auto mode
update-alternatives: warning: skip creation of /usr/share/man/da/man1/ex.1.gz because associated file /usr/share/man/da/man1/vim.1.gz (of link group ex) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/de/man1/ex.1.gz because associated file /usr/share/man/de/man1/vim.1.gz (of link group ex) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/fr/man1/ex.1.gz because associated file /usr/share/man/fr/man1/vim.1.gz (of link group ex) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/it/man1/ex.1.gz because associated file /usr/share/man/it/man1/vim.1.gz (of link group ex) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/ja/man1/ex.1.gz because associated file /usr/share/man/ja/man1/vim.1.gz (of link group ex) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/pl/man1/ex.1.gz because associated file /usr/share/man/pl/man1/vim.1.gz (of link group ex) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/ru/man1/ex.1.gz because associated file /usr/share/man/ru/man1/vim.1.gz (of link group ex) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/man1/ex.1.gz because associated file /usr/share/man/man1/vim.1.gz (of link group ex) doesn't exist
update-alternatives: using /usr/bin/vim.basic to provide /usr/bin/editor (editor) in auto mode
update-alternatives: warning: skip creation of /usr/share/man/da/man1/editor.1.gz because associated file /usr/share/man/da/man1/vim.1.gz (of link group editor) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/de/man1/editor.1.gz because associated file /usr/share/man/de/man1/vim.1.gz (of link group editor) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/fr/man1/editor.1.gz because associated file /usr/share/man/fr/man1/vim.1.gz (of link group editor) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/it/man1/editor.1.gz because associated file /usr/share/man/it/man1/vim.1.gz (of link group editor) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/ja/man1/editor.1.gz because associated file /usr/share/man/ja/man1/vim.1.gz (of link group editor) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/pl/man1/editor.1.gz because associated file /usr/share/man/pl/man1/vim.1.gz (of link group editor) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/ru/man1/editor.1.gz because associated file /usr/share/man/ru/man1/vim.1.gz (of link group editor) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/man1/editor.1.gz because associated file /usr/share/man/man1/vim.1.gz (of link group editor) doesn't exist
Processing triggers for libc-bin (2.28-10) ...

root@testing-pod:/# vi /etc/resolv.conf 

root@testing-pod:/# cat /etc/resolv.conf
search default.svc.cluster.local svc.cluster.local cluster.local lxd
nameserver 8.8.8.8
options ndots:5

root@testing-pod:/# exit
exit

# 
```




# 
Return to: [main menu](https://github.com/jimenezcorzo/Kubernetes-The-Hard-Way-15.3-LXC/blob/master/Readme.md)
