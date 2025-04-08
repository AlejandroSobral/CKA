## Kubelet Misconfigured

Someone tried to improve the Kubelet on Node node01 , but broke it instead. Fix it.

---------------------------------------------------------------------

### Demo

Make an analysis of the situation.
```bash
controlplane:~$ k get nodes
NAME           STATUS     ROLES           AGE   VERSION
controlplane   Ready      control-plane   17d   v1.32.1
node01         NotReady   <none>          17d   v1.32.1

controlplane:~$ ssh node01
Last login: Mon Feb 10 22:06:42 2025 from 10.244.0.131
node01:~$ systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; preset: enabled)
    Drop-In: /usr/lib/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: activating (auto-restart) (Result: exit-code) since Tue 2025-04-08 22:17:01 UTC; 8s ago
       Docs: https://kubernetes.io/docs/
    Process: 6527 ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARG>
   Main PID: 6527 (code=exited, status=1/FAILURE)
        CPU: 54ms
```
&nbsp;

node01 is not working. ssh into node01 shows that kubelet exited with error: "code=exited, status=1/FAILURE"

Further checking into the syslogs, shows once again that kubeletd flag --improve-speed is unknown.
```bash
node01:~$ journalctl | tail -f -n 10
Apr 08 22:17:32 node01 kubelet[6602]: E0408 22:17:32.201675    6602 run.go:72] "command failed" err="failed to parse kubelet flag: unknown flag: --improve-speed"
Apr 08 22:17:32 node01 systemd[1]: kubelet.service: Main process exited, code=exited, status=1/FAILURE
Apr 08 22:17:32 node01 systemd[1]: kubelet.service: Failed with result 'exit-code'.
Apr 08 22:17:42 node01 systemd[1]: kubelet.service: Scheduled restart job, restart counter is at 10.
Apr 08 22:17:42 node01 systemd[1]: Started kubelet.service - kubelet: The Kubernetes Node Agent.
Apr 08 22:17:42 node01 kubelet[6622]: Flag --container-runtime-endpoint has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ for more information.
Apr 08 22:17:42 node01 kubelet[6622]: Flag --pod-infra-container-image has been deprecated, will be removed in 1.35. Image garbage collector will get sandbox image information from CRI.
Apr 08 22:17:42 node01 kubelet[6622]: E0408 22:17:42.463143    6622 run.go:72] "command failed" err="failed to parse kubelet flag: unknown flag: --improve-speed"
Apr 08 22:17:42 node01 systemd[1]: kubelet.service: Main process exited, code=exited, status=1/FAILURE
Apr 08 22:17:42 node01 systemd[1]: kubelet.service: Failed with result 'exit-code'.
```
&nbsp;

The issue now comes resides on knowing where those flag came from.
```bash
node01:~$ find / | grep kubeadm
/usr/share/doc/kubeadm
/usr/share/doc/kubeadm/README.md
/usr/share/doc/kubeadm/LICENSE
/usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf
/usr/bin/kubeadm
/var/lib/kubelet/kubeadm-flags.env
/var/lib/dpkg/info/kubeadm.md5sums
/var/lib/dpkg/info/kubeadm.list
/var/cache/apt/archives/kubeadm_1.32.1-1.1_amd64.deb
```
&nbsp;

As per the output, there existed one file containing kubelet flags:
```bash
node01:~$ cat /var/lib/kubelet/kubeadm-flags.env
KUBELET_KUBEADM_ARGS="--container-runtime-endpoint=unix:///var/run/containerd/containerd.sock --pod-infra-container-image=registry.k8s.io/pause:3.10 --improve-speed"
```
Culript founded!

&nbsp;

At this point it is a matter of doing a backup and editing the file. Of course, then restart the service and verify the status of the node.

```bash
node01:~$ cp /var/lib/kubelet/kubeadm-flags.env ~/kubeadm-flags.env.bkp
node01:~$ nano /var/lib/kubelet/kubeadm-flags.env
node01:~$ systemctl restart kubelet

Connection to node01 closed.

controlplane:~$ k get nodes
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   17d   v1.32.1
node01         Ready    <none>          17d   v1.32.1
```
&nbsp;

And that's it!.
