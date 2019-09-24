# Some Helpful Troubleshooting Commands

In case you need to do some troubleshooting during the deployment of your Kubernetes Cluster "The Hard Way", I am leaving to you some useful commands, and some use cases.

To check the status of the service:
```
# systemctl status etcd
```
```
# systemctl status kube-scheduler
```
```
# systemctl status kube-apiserver
```
```
# systemctl status kube-controller-manager
```
```
# systemctl status kubelet
```
Example of one output:
```
# systemctl status kubelet
● kubelet.service - Kubernetes Kubelet
   Loaded: loaded (/etc/systemd/system/kubelet.service; enabled; vendor preset: enabled)
   Active: active (running) since Sat 2019-09-21 20:25:08 UTC; 3 days ago
     Docs: https://github.com/kubernetes/kubernetes
 Main PID: 47377 (kubelet)
    Tasks: 16
   Memory: 59.7M
      CPU: 1h 9min 58.328s
   CGroup: /system.slice/kubelet.service
           └─47377 /usr/local/bin/kubelet --config=/var/lib/kubelet/kubelet-config.yaml --container-runtime=remote --container-runtime-endpoint=unix:///var/run/

Sep 24 18:09:50 worker-0 kubelet[47377]: I0924 18:09:50.731118   47377 kubelet.go:1929] SyncLoop (PLEG): "webapp-8544985cc7-nnffw_default(a2bfda0f-36f0-40f8-b7e
Sep 24 18:09:51 worker-0 kubelet[47377]: I0924 18:09:51.733564   47377 kubelet.go:1929] SyncLoop (PLEG): "webapp-8544985cc7-nnffw_default(a2bfda0f-36f0-40f8-b7e
Sep 24 18:09:51 worker-0 kubelet[47377]: I0924 18:09:51.866181   47377 reconciler.go:177] operationExecutor.UnmountVolume started for volume "default-token-phn2
Sep 24 18:09:51 worker-0 kubelet[47377]: I0924 18:09:51.873617   47377 operation_generator.go:860] UnmountVolume.TearDown succeeded for volume "kubernetes.io/se
Sep 24 18:09:51 worker-0 kubelet[47377]: I0924 18:09:51.966940   47377 reconciler.go:297] Volume detached for volume "default-token-phn2m" (UniqueName: "kuberne
Sep 24 18:09:52 worker-0 kubelet[47377]: I0924 18:09:52.736600   47377 kubelet.go:1929] SyncLoop (PLEG): "webapp-795bbd7d8-csd2c_default(5b188aa6-849f-4579-a443
Sep 24 18:09:53 worker-0 kubelet[47377]: I0924 18:09:53.740807   47377 kubelet.go:1929] SyncLoop (PLEG): "webapp-795bbd7d8-csd2c_default(5b188aa6-849f-4579-a443
Sep 24 18:10:03 worker-0 kubelet[47377]: I0924 18:10:03.428461   47377 kubelet.go:1900] SyncLoop (DELETE, "api"): "webapp-8544985cc7-nnffw_default(a2bfda0f-36f0
Sep 24 18:10:03 worker-0 kubelet[47377]: I0924 18:10:03.438149   47377 kubelet.go:1894] SyncLoop (REMOVE, "api"): "webapp-8544985cc7-nnffw_default(a2bfda0f-36f0
Sep 24 18:10:03 worker-0 kubelet[47377]: I0924 18:10:03.438213   47377 kubelet.go:2097] Failed to delete pod "webapp-8544985cc7-nnffw_default(a2bfda0f-36f0-40f8
lines 1-21/21 (END)

```
To check related informational messages on system logs:
### In general
```
# journalctl -xe
```
### For a particular service
```
# journalctl -xeu etcd
```
```
# journalctl -xeu kube-scheduler
```
```
# journalctl -xeu kube-apiserver
```
```
# journalctl -xeu kube-controller-manager
```
```
# journalctl -xeu kubelet
```
Fragment of the output:
```
# journalctl -xeu kubelet
...
...
...
Sep 24 18:08:23 worker-0 kubelet[47377]: I0924 18:08:23.553685   47377 kubelet.go:1929] SyncLoop (PLEG): "webapp-795bbd7d8-77ggn_default(5b64476f-a042-4ba2-b88e
Sep 24 18:08:23 worker-0 kubelet[47377]: I0924 18:08:23.575664   47377 reconciler.go:177] operationExecutor.UnmountVolume started for volume "default-token-phn2
Sep 24 18:08:23 worker-0 kubelet[47377]: I0924 18:08:23.583949   47377 operation_generator.go:860] UnmountVolume.TearDown succeeded for volume "kubernetes.io/se
Sep 24 18:08:23 worker-0 kubelet[47377]: I0924 18:08:23.675944   47377 reconciler.go:297] Volume detached for volume "default-token-phn2m" (UniqueName: "kuberne
Sep 24 18:08:33 worker-0 kubelet[47377]: I0924 18:08:33.423550   47377 kubelet.go:1900] SyncLoop (DELETE, "api"): "webapp-795bbd7d8-77ggn_default(5b64476f-a042-
Sep 24 18:08:33 worker-0 kubelet[47377]: I0924 18:08:33.434870   47377 kubelet.go:1894] SyncLoop (REMOVE, "api"): "webapp-795bbd7d8-77ggn_default(5b64476f-a042-
Sep 24 18:08:33 worker-0 kubelet[47377]: I0924 18:08:33.436125   47377 kubelet.go:2097] Failed to delete pod "webapp-795bbd7d8-77ggn_default(5b64476f-a042-4ba2-
Sep 24 18:09:49 worker-0 kubelet[47377]: I0924 18:09:49.890060   47377 kubelet.go:1900] SyncLoop (DELETE, "api"): "webapp-8544985cc7-nnffw_default(a2bfda0f-36f0
Sep 24 18:09:49 worker-0 kubelet[47377]: I0924 18:09:49.890448   47377 kuberuntime_container.go:581] Killing container "containerd://66902eae9958b172bbfedcb2a6e
Sep 24 18:09:50 worker-0 kubelet[47377]: I0924 18:09:50.127806   47377 kubelet.go:1884] SyncLoop (ADD, "api"): "webapp-795bbd7d8-csd2c_default(5b188aa6-849f-457
Sep 24 18:09:50 worker-0 kubelet[47377]: I0924 18:09:50.235839   47377 reconciler.go:203] operationExecutor.VerifyControllerAttachedVolume started for volume "d
Sep 24 18:09:50 worker-0 kubelet[47377]: I0924 18:09:50.336187   47377 reconciler.go:248] operationExecutor.MountVolume started for volume "default-token-phn2m"
Sep 24 18:09:50 worker-0 kubelet[47377]: I0924 18:09:50.429743   47377 operation_generator.go:713] MountVolume.SetUp succeeded for volume "default-token-phn2m" 
Sep 24 18:09:50 worker-0 kubelet[47377]: I0924 18:09:50.530414   47377 kuberuntime_manager.go:404] No sandbox for pod "webapp-795bbd7d8-csd2c_default(5b188aa6-8
Sep 24 18:09:50 worker-0 kubelet[47377]: I0924 18:09:50.731118   47377 kubelet.go:1929] SyncLoop (PLEG): "webapp-8544985cc7-nnffw_default(a2bfda0f-36f0-40f8-b7e
Sep 24 18:09:51 worker-0 kubelet[47377]: I0924 18:09:51.733564   47377 kubelet.go:1929] SyncLoop (PLEG): "webapp-8544985cc7-nnffw_default(a2bfda0f-36f0-40f8-b7e
Sep 24 18:09:51 worker-0 kubelet[47377]: I0924 18:09:51.866181   47377 reconciler.go:177] operationExecutor.UnmountVolume started for volume "default-token-phn2
Sep 24 18:09:51 worker-0 kubelet[47377]: I0924 18:09:51.873617   47377 operation_generator.go:860] UnmountVolume.TearDown succeeded for volume "kubernetes.io/se
Sep 24 18:09:51 worker-0 kubelet[47377]: I0924 18:09:51.966940   47377 reconciler.go:297] Volume detached for volume "default-token-phn2m" (UniqueName: "kuberne
Sep 24 18:09:52 worker-0 kubelet[47377]: I0924 18:09:52.736600   47377 kubelet.go:1929] SyncLoop (PLEG): "webapp-795bbd7d8-csd2c_default(5b188aa6-849f-4579-a443
Sep 24 18:09:53 worker-0 kubelet[47377]: I0924 18:09:53.740807   47377 kubelet.go:1929] SyncLoop (PLEG): "webapp-795bbd7d8-csd2c_default(5b188aa6-849f-4579-a443
Sep 24 18:10:03 worker-0 kubelet[47377]: I0924 18:10:03.428461   47377 kubelet.go:1900] SyncLoop (DELETE, "api"): "webapp-8544985cc7-nnffw_default(a2bfda0f-36f0
Sep 24 18:10:03 worker-0 kubelet[47377]: I0924 18:10:03.438149   47377 kubelet.go:1894] SyncLoop (REMOVE, "api"): "webapp-8544985cc7-nnffw_default(a2bfda0f-36f0
Sep 24 18:10:03 worker-0 kubelet[47377]: I0924 18:10:03.438213   47377 kubelet.go:2097] Failed to delete pod "webapp-8544985cc7-nnffw_default(a2bfda0f-36f0-40f8
lines 963-1001/1001 (END)

```

# 
Return to: [main menu](https://github.com/jimenezcorzo/Kubernetes-The-Hard-Way-15.3-LXC/blob/master/Readme.md)


