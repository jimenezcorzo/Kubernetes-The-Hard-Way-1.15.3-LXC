# Generating Kubernetes Configuration Files for Authentication

In this task, we are going to create all the kubeconfig files, needed to use the generated certificates, to authenticate our access to the different Kubernetes components.

## Keep in mind, the Load Balancer IP
**KUBERNETES_PUBLIC_ADDRESS=10.12.91.252**

## Workers kubeconfig
### worker-0.kubeconfig
```
  kubectl config set-cluster kubernetes \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://10.12.91.252:6443 \
    --kubeconfig=worker-0.kubeconfig

  kubectl config set-credentials system:node:worker-0 \
    --client-certificate=worker-0.crt \
    --client-key=worker-0.key \
    --embed-certs=true \
    --kubeconfig=worker-0.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes \
    --user=system:node:worker-0 \
    --kubeconfig=worker-0.kubeconfig

  kubectl config use-context default --kubeconfig=worker-0.kubeconfig
```
### worker-1.kubeconfig
```
  kubectl config set-cluster kubernetes \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://10.12.91.252:6443 \
    --kubeconfig=worker-1.kubeconfig

  kubectl config set-credentials system:node:worker-1 \
    --client-certificate=worker-1.crt \
    --client-key=worker-1.key \
    --embed-certs=true \
    --kubeconfig=worker-1.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes \
    --user=system:node:worker-1 \
    --kubeconfig=worker-1.kubeconfig

  kubectl config use-context default --kubeconfig=worker-1.kubeconfig
```
### worker-2.kubeconfig
```
  kubectl config set-cluster kubernetes \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://10.12.91.252:6443 \
    --kubeconfig=worker-2.kubeconfig

  kubectl config set-credentials system:node:worker-2 \
    --client-certificate=worker-2.crt \
    --client-key=worker-2.key \
    --embed-certs=true \
    --kubeconfig=worker-2.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes \
    --user=system:node:worker-2 \
    --kubeconfig=worker-2.kubeconfig

  kubectl config use-context default --kubeconfig=worker-2.kubeconfig
```
## proxy.kubeconfig

```
  kubectl config set-cluster kubernetes \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://10.12.91.252:6443 \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-credentials system:kube-proxy \
    --client-certificate=kube-proxy.crt \
    --client-key=kube-proxy.key \
    --embed-certs=true \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes \
    --user=system:kube-proxy \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
```
## kube-controller-manager.kubeconfig

```
  kubectl config set-cluster kubernetes \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://10.12.91.252:6443 \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-credentials system:kube-controller-manager \
    --client-certificate=kube-controller-manager.crt \
    --client-key=kube-controller-manager.key \
    --embed-certs=true \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes \
    --user=system:kube-controller-manager \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config use-context default --kubeconfig=kube-controller-manager.kubeconfig
```

## kube-scheduler.kubeconfig

```
  kubectl config set-cluster kubernetes \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://10.12.91.252:6443 \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-credentials system:kube-scheduler \
    --client-certificate=kube-scheduler.crt \
    --client-key=kube-scheduler.key \
    --embed-certs=true \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes \
    --user=system:kube-scheduler \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config use-context default --kubeconfig=kube-scheduler.kubeconfig
```
## admin.kubeconfig

```
  kubectl config set-cluster kubernetes \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://10.12.91.252:6443 \
    --kubeconfig=admin.kubeconfig

  kubectl config set-credentials admin \
    --client-certificate=admin.crt \
    --client-key=admin.key \
    --embed-certs=true \
    --kubeconfig=admin.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes \
    --user=admin \
    --kubeconfig=admin.kubeconfig

  kubectl config use-context default --kubeconfig=admin.kubeconfig
```
Transfering the kubeconfig files to the corresponding containers.
#### To the Worker nodes
```
lxc file push worker-0.kubeconfig kube-proxy.kubeconfig worker-0/root/
lxc file push worker-1.kubeconfig kube-proxy.kubeconfig worker-1/root/               
lxc file push worker-2.kubeconfig kube-proxy.kubeconfig worker-2/root/         
```
#### To the controller manager nodes (master nodes)
```
lxc file push admin.kubeconfig kube-controller-manager.kubeconfig kube-scheduler.kubeconfig controller-0/root/
lxc file push admin.kubeconfig kube-controller-manager.kubeconfig kube-scheduler.kubeconfig controller-1/root/ 
lxc file push admin.kubeconfig kube-controller-manager.kubeconfig kube-scheduler.kubeconfig controller-2/root/ 
```

Now we have created the kubeconfig files, and embed the certificates information accordingly. 

# 
Return to: [main menu](https://github.com/jimenezcorzo/Kubernetes-The-Hard-Way-15.3-LXC/blob/master/Readme.md)
