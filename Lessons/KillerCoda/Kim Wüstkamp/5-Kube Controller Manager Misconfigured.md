## Kube Controller Manager Misconfigured

A custom Kube Controller Manager container image was running in this cluster for testing. It has been reverted back to the default one, but it's not coming back up. Fix it.

----------------------------------------------------------------

### Demo

Check the status of Pods:
```bash
controlplane:~$ k get pods -n kube-system 
NAME                                      READY   STATUS             RESTARTS      AGE
..
kube-controller-manager-controlplane      0/1     CrashLoopBackOff   4 (8s ago)    107s
..
..
```
&nbsp;


Make a backup of the file:
```bash
controlplane:~$ cp /etc/kubernetes/manifests/kube-controller-manager.yaml ~/kube-controller-manager.yaml.bkp
```
&nbsp;


Tail the journalctl logs to see if anything can be found:
```bash
controlplane:~$ tail -f -n 5 /var/log/pods/kube-system_kube-controller-manager-controlplane_1a4f8198011bc5a96cd8421ee1ff1d3c/kube-controller-manager/5.log 
2025-04-08T21:48:02.991524159Z stderr F 
2025-04-08T21:48:02.991526304Z stderr F   -h, --help                     help for kube-controller-manager
2025-04-08T21:48:02.991528583Z stderr F       --version version[=true]   --version, --version=raw prints version information and quits; --version=vX.Y.Z... sets the reported version
2025-04-08T21:48:02.991638237Z stderr F 
2025-04-08T21:48:02.991650392Z stderr F Error: unknown flag: --project-sidecar-insertion
```
&nbsp;

The file looked like. Apparently, the flag --project-sidecar-insertion shouldn't be there.
```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    component: kube-controller-manager
    tier: control-plane
  name: kube-controller-manager
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-controller-manager
    - --allocate-node-cidrs=true
    ..
    ..
    - --leader-elect=true
    - --project-sidecar-insertion=true
    ..
    ..
    image: registry.k8s.io/kube-controller-manager:v1.32.1
```
&nbsp;


Delete the flag and then restart the kubelet service. Other option would be to move the file out of the folder for some minutes and set it back again.
Check again the Pods status:
```bash
controlplane:~$ systemctl restart kubelet
controlplane:~$ k get pods -n kube-system 
NAME                                      READY   STATUS    RESTARTS      AGE
..
kube-controller-manager-controlplane      0/1     Running   0             24s
..
```
&nbsp;


And that's it!.