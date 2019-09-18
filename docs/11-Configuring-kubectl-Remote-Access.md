# Configuring kubectl for Remote Access

As we did on the section when we created the kubeconfig files needed for all the services, we will now to create the admin kubeconfig file:

Remember to use the apiserverlb ipaddress:
```
  KUBERNETES_PUBLIC_ADDRESS=10.12.91.252
```
Creating the admin kubeconfig file
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
## Verification
Check the health of the remote Kubernetes cluster:
```
# kubectl get cs
NAME                 STATUS    MESSAGE             ERROR
scheduler            Healthy   ok                  
etcd-0               Healthy   {"health":"true"}   
etcd-1               Healthy   {"health":"true"}   
etcd-2               Healthy   {"health":"true"}   
controller-manager   Healthy   ok                  
```
Checking nodes:
```
# kubectl get nodes
NAME       STATUS   ROLES    AGE   VERSION
worker-0   Ready    <none>   8d    v1.15.3
worker-1   Ready    <none>   8d    v1.15.3
worker-2   Ready    <none>   8d    v1.15.3
```
Checking cluster info
```
# kubectl cluster-info
Kubernetes master is running at https://10.12.91.252:6443

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

#
Return to: [main menu](https://github.com/jimenezcorzo/Kubernetes-The-Hard-Way-15.3-LXC/blob/master/Readme.md)
