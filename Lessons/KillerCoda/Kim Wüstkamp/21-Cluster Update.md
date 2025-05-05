### Cluster Update

- Upgrade the controlplane Node to a newer patch version.
- Also upgrade kubectl and kubelet.

--------------------------


### Update kubeadm

Having this [Docs](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/) handy, proceed:

&nbsp;

Check the current version:
```bash
kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"32", GitVersion:"v1.32.1", GitCommit:"e9c9be4007d1664e68796af02b8978640d2c1b26", GitTreeState:"clean", BuildDate:"2025-01-15T14:39:14Z", GoVersion:"go1.23.4", Compiler:"gc", Platform:"linux/amd64"}
```

&nbsp;
List available packages:
```bash
sudo apt update
sudo apt-cache madison kubeadm
```

Output:
```bash
...
kubeadm | 1.32.4-1.1 | https://pkgs.k8s.io/core:/stable:/v1.32/deb  Packages
kubeadm | 1.32.3-1.1 | https://pkgs.k8s.io/core:/stable:/v1.32/deb  Packages
kubeadm | 1.32.2-1.1 | https://pkgs.k8s.io/core:/stable:/v1.32/deb  Packages
kubeadm | 1.32.1-1.1 | https://pkgs.k8s.io/core:/stable:/v1.32/deb  Packages
kubeadm | 1.32.0-1.1 | https://pkgs.k8s.io/core:/stable:/v1.32/deb  Packages
```

&nbsp;

```bash
# replace x in 1.33.x-* with the latest patch version
sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.32.4-1.1' && \
sudo apt-mark hold kubeadm
```

&nbsp;

<details>
<summary> Output </summary>

```bash
kubeadm was already not on hold.
Hit:1 http://archive.ubuntu.com/ubuntu noble InRelease
Hit:3 http://security.ubuntu.com/ubuntu noble-security InRelease    
Hit:4 http://archive.ubuntu.com/ubuntu noble-updates InRelease      
Hit:5 https://packages.mozilla.org/apt mozilla InRelease
Hit:6 http://archive.ubuntu.com/ubuntu noble-backports InRelease 
Hit:2 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.32/deb  InRelease
Reading package lists... Done                             
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following packages will be upgraded:
  kubeadm
1 upgraded, 0 newly installed, 0 to remove and 145 not upgraded.
Need to get 12.2 MB of archives.
After this operation, 0 B of additional disk space will be used.
Get:1 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.32/deb  kubeadm 1.32.4-1.1 [12.2 MB]
Fetched 12.2 MB in 1s (21.7 MB/s)
(Reading database ... 140016 files and directories currently installed.)
Preparing to unpack .../kubeadm_1.32.4-1.1_amd64.deb ...
Unpacking kubeadm (1.32.4-1.1) over (1.32.1-1.1) ...
Setting up kubeadm (1.32.4-1.1) ...
Scanning processes...                                                                                                                                                                      
Scanning linux images...                                                                                                                                                                   

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
kubeadm set on hold.
```

</details>

&nbsp;

Check then the version:
```bash
controlplane:~$ kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"32", GitVersion:"v1.32.4", GitCommit:"59526cd4867447956156ae3a602fcbac10a2c335", GitTreeState:"clean", BuildDate:"2025-04-22T16:02:27Z", GoVersion:"go1.23.6", Compiler:"gc", Platform:"linux/amd64"}
```

Verify plan; "This command checks that your cluster can be upgraded, and fetches the versions you can upgrade to. It also shows a table with the component config version states."
```bash
sudo kubeadm upgrade plan
```

&nbsp;

```bash
[preflight] Running pre-flight checks.
[upgrade/config] Reading configuration from the "kubeadm-config" ConfigMap in namespace "kube-system"...
[upgrade/config] Use 'kubeadm init phase upload-config --config your-config.yaml' to re-upload it.
[upgrade] Running cluster health checks
[upgrade] Fetching available versions to upgrade to
[upgrade/versions] Cluster version: 1.32.1
[upgrade/versions] kubeadm version: v1.32.4
I0505 16:26:37.507004   30391 version.go:261] remote version is much newer: v1.33.0; falling back to: stable-1.32
[upgrade/versions] Target version: v1.32.4
[upgrade/versions] Latest version in the v1.32 series: v1.32.4

Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   NODE           CURRENT   TARGET
kubelet     controlplane   v1.32.1   v1.32.4
kubelet     node01         v1.32.1   v1.32.4

Upgrade to the latest version in the v1.32 series:

COMPONENT                 NODE           CURRENT    TARGET
kube-apiserver            controlplane   v1.32.1    v1.32.4
kube-controller-manager   controlplane   v1.32.1    v1.32.4
kube-scheduler            controlplane   v1.32.1    v1.32.4
kube-proxy                               1.32.1     v1.32.4
CoreDNS                                  v1.11.3    v1.11.3
etcd                      controlplane   3.5.16-0   3.5.16-0

You can now apply the upgrade by executing the following command:

        kubeadm upgrade apply v1.32.4

_____________________________________________________________________


The table below shows the current state of component configs as understood by this version of kubeadm.
Configs that have a "yes" mark in the "MANUAL UPGRADE REQUIRED" column require manual config upgrade or
resetting to kubeadm defaults before a successful upgrade can be performed. The version to manually
upgrade to is denoted in the "PREFERRED VERSION" column.

API GROUP                 CURRENT VERSION   PREFERRED VERSION   MANUAL UPGRADE REQUIRED
kubeproxy.config.k8s.io   v1alpha1          v1alpha1            no
kubelet.config.k8s.io     v1beta1           v1beta1             no
_____________________________________________________________________
```

&nbsp;

Apply the updgrade:
```bash
kubeadm upgrade apply v1.32.4
```

<details>
<summary> Large output</summary>

```bash
[upgrade] Reading configuration from the "kubeadm-config" ConfigMap in namespace "kube-system"...
[upgrade] Use 'kubeadm init phase upload-config --config your-config.yaml' to re-upload it.
[upgrade/preflight] Running preflight checks
[upgrade] Running cluster health checks
[upgrade/preflight] You have chosen to upgrade the cluster version to "v1.32.4"
[upgrade/versions] Cluster version: v1.32.1
[upgrade/versions] kubeadm version: v1.32.4
[upgrade] Are you sure you want to proceed? [y/N]: y
[upgrade/preflight] Pulling images required for setting up a Kubernetes cluster
[upgrade/preflight] This might take a minute or two, depending on the speed of your internet connection
[upgrade/preflight] You can also perform this action beforehand using 'kubeadm config images pull'
W0505 16:28:04.685612   31184 checks.go:846] detected that the sandbox image "registry.k8s.io/pause:3.5" of the container runtime is inconsistent with that used by kubeadm.It is recommended to use "registry.k8s.io/pause:3.10" as the CRI sandbox image.
[upgrade/control-plane] Upgrading your static Pod-hosted control plane to version "v1.32.4" (timeout: 5m0s)...
[upgrade/staticpods] Writing new Static Pod manifests to "/etc/kubernetes/tmp/kubeadm-upgraded-manifests370359958"
[upgrade/staticpods] Preparing for "etcd" upgrade
[upgrade/staticpods] Renewing etcd-server certificate
[upgrade/staticpods] Renewing etcd-peer certificate
[upgrade/staticpods] Renewing etcd-healthcheck-client certificate
[upgrade/staticpods] Moving new manifest to "/etc/kubernetes/manifests/etcd.yaml" and backing up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2025-05-05-16-28-23/etcd.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This can take up to 5m0s
E0505 16:28:35.168171   31184 request.go:1332] Unexpected error when reading response body: net/http: request canceled (Client.Timeout or context cancellation while reading body)
E0505 16:29:05.173310   31184 request.go:1332] Unexpected error when reading response body: net/http: request canceled (Client.Timeout or context cancellation while reading body)
E0505 16:29:15.175384   31184 request.go:1332] Unexpected error when reading response body: net/http: request canceled (Client.Timeout or context cancellation while reading body)
E0505 16:29:25.176829   31184 request.go:1332] Unexpected error when reading response body: net/http: request canceled (Client.Timeout or context cancellation while reading body)
E0505 16:29:35.177934   31184 request.go:1332] Unexpected error when reading response body: net/http: request canceled (Client.Timeout or context cancellation while reading body)
[apiclient] Found 1 Pods for label selector component=etcd
[upgrade/staticpods] Component "etcd" upgraded successfully!
[upgrade/etcd] Waiting for etcd to become available
[upgrade/staticpods] Preparing for "kube-apiserver" upgrade
[upgrade/staticpods] Renewing apiserver certificate
[upgrade/staticpods] Renewing apiserver-kubelet-client certificate
[upgrade/staticpods] Renewing front-proxy-client certificate
[upgrade/staticpods] Renewing apiserver-etcd-client certificate
[upgrade/staticpods] Moving new manifest to "/etc/kubernetes/manifests/kube-apiserver.yaml" and backing up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2025-05-05-16-28-23/kube-apiserver.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This can take up to 5m0s
[apiclient] Found 1 Pods for label selector component=kube-apiserver
[upgrade/staticpods] Component "kube-apiserver" upgraded successfully!
[upgrade/staticpods] Preparing for "kube-controller-manager" upgrade
[upgrade/staticpods] Renewing controller-manager.conf certificate
[upgrade/staticpods] Moving new manifest to "/etc/kubernetes/manifests/kube-controller-manager.yaml" and backing up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2025-05-05-16-28-23/kube-controller-manager.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This can take up to 5m0s
[apiclient] Found 1 Pods for label selector component=kube-controller-manager
[upgrade/staticpods] Component "kube-controller-manager" upgraded successfully!
[upgrade/staticpods] Preparing for "kube-scheduler" upgrade
[upgrade/staticpods] Renewing scheduler.conf certificate
[upgrade/staticpods] Moving new manifest to "/etc/kubernetes/manifests/kube-scheduler.yaml" and backing up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2025-05-05-16-28-23/kube-scheduler.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This can take up to 5m0s
[apiclient] Found 1 Pods for label selector component=kube-scheduler
[upgrade/staticpods] Component "kube-scheduler" upgraded successfully!
[upgrade/control-plane] The control plane instance for this node was successfully upgraded!
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[upgrad/kubeconfig] The kubeconfig files for this node were successfully upgraded!
W0505 16:30:50.226974   31184 postupgrade.go:117] Using temporary directory /etc/kubernetes/tmp/kubeadm-kubelet-config164705771 for kubelet config. To override it set the environment variable KUBEADM_UPGRADE_DRYRUN_DIR
[upgrade] Backing up kubelet config file to /etc/kubernetes/tmp/kubeadm-kubelet-config164705771/config.yaml
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[upgrade/kubelet-config] The kubelet configuration for this node was successfully upgraded!
[upgrade/bootstrap-token] Configuring bootstrap token and cluster-info RBAC rules
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy
```

</details>

&nbsp;

Short log version:
```bash
...
..
[upgrade] SUCCESS! A control plane node of your cluster was upgraded to "v1.32.4"

controlplane:~$ kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"32", GitVersion:"v1.32.4", GitCommit:"59526cd4867447956156ae3a602fcbac10a2c335", GitTreeState:"clean", BuildDate:"2025-04-22T16:02:27Z", GoVersion:"go1.23.6", Compiler:"gc", Platform:"linux/amd64"}
```

### Update kubectl and kubelet

Check avialable versions:

```bash
controlplane:~$ sudo apt-cache madison kubelet
   kubelet | 1.32.4-1.1 | https://pkgs.k8s.io/core:/stable:/v1.32/deb  Packages
   kubelet | 1.32.3-1.1 | https://pkgs.k8s.io/core:/stable:/v1.32/deb  Packages
   kubelet | 1.32.2-1.1 | https://pkgs.k8s.io/core:/stable:/v1.32/deb  Packages
   kubelet | 1.32.1-1.1 | https://pkgs.k8s.io/core:/stable:/v1.32/deb  Packages
   kubelet | 1.32.0-1.1 | https://pkgs.k8s.io/core:/stable:/v1.32/deb  Packages
controlplane:~$ sudo apt-cache madison kubectl
   kubectl | 1.32.4-1.1 | https://pkgs.k8s.io/core:/stable:/v1.32/deb  Packages
   kubectl | 1.32.3-1.1 | https://pkgs.k8s.io/core:/stable:/v1.32/deb  Packages
   kubectl | 1.32.2-1.1 | https://pkgs.k8s.io/core:/stable:/v1.32/deb  Packages
   kubectl | 1.32.1-1.1 | https://pkgs.k8s.io/core:/stable:/v1.32/deb  Packages
   kubectl | 1.32.0-1.1 | https://pkgs.k8s.io/core:/stable:/v1.32/deb  Packages
```

&nbsp;
Upgrade:
```bash
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.32.4-1.1' kubectl='1.32.4-1.1' && \
sudo apt-mark hold kubelet kubectl
```

<details>
<summary> Large output</summary>

```bash
Hit:1 http://security.ubuntu.com/ubuntu noble-security InRelease
Hit:2 http://archive.ubuntu.com/ubuntu noble InRelease                                                                 
Hit:3 http://archive.ubuntu.com/ubuntu noble-updates InRelease                                                         
Hit:4 http://archive.ubuntu.com/ubuntu noble-backports InRelease                                                       
Hit:6 https://packages.mozilla.org/apt mozilla InRelease                                                               
Hit:5 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.32/deb  InRelease
Reading package lists... Done
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following packages will be upgraded:
  kubectl kubelet
2 upgraded, 0 newly installed, 0 to remove and 143 not upgraded.
Need to get 26.4 MB of archives.
After this operation, 8192 B of additional disk space will be used.
Get:1 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.32/deb  kubectl 1.32.4-1.1 [11.2 MB]
Get:2 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.32/deb  kubelet 1.32.4-1.1 [15.2 MB]
Fetched 26.4 MB in 1s (23.3 MB/s)  
(Reading database ... 140016 files and directories currently installed.)
Preparing to unpack .../kubectl_1.32.4-1.1_amd64.deb ...
Unpacking kubectl (1.32.4-1.1) over (1.32.1-1.1) ...
Preparing to unpack .../kubelet_1.32.4-1.1_amd64.deb ...
Unpacking kubelet (1.32.4-1.1) over (1.32.1-1.1) ...
Setting up kubectl (1.32.4-1.1) ...
Setting up kubelet (1.32.4-1.1) ...
Scanning processes...                                                                                                                                                                      
Scanning candidates...                                                                                                                                                                     
Scanning linux images...                                                                                                                                                                   

Running kernel seems to be up-to-date.

Restarting services...
 systemctl restart kubelet.service

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
kubelet set on hold.
kubectl set on hold.
```

</details>

&nbsp;


Restart the services:

```bash
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

&nbsp;

Check the versions:
```bash
controlplane:~$ kubectl version
Client Version: v1.32.4
Kustomize Version: v5.5.0
Server Version: v1.32.4

controlplane:~$ k get nodes
NAME           STATUS   ROLES           AGE    VERSION
controlplane   Ready    control-plane   7d4h   v1.32.4
node01         Ready    <none>          7d4h   v1.32.1

controlplane:~$ kubelet --version 
Kubernetes v1.32.4
```

&nbsp;

The Control Plane is now ready and updated.

&nbsp;

------------------------------------------------

### Upgrade Worker Node -- node01

Connect to Worker Node and get the Versions:

```bash
controlplane:~$ ssh node01
Last login: Mon Feb 10 22:06:42 2025 from 10.244.0.131

node01:~$ k version
Client Version: v1.32.1
Kustomize Version: v5.5.0
The connection to the server localhost:8080 was refused - did you specify the right host or port?

node01:~$ kubelet --version
Kubernetes v1.32.1
node01:~$ kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"32", GitVersion:"v1.32.1", GitCommit:"e9c9be4007d1664e68796af02b8978640d2c1b26", GitTreeState:"clean", BuildDate:"2025-01-15T14:39:14Z", GoVersion:"go1.23.4", Compiler:"gc", Platform:"linux/amd64"}
```

&nbsp;


Drain the Node and proceed with the Update:

```bash
controlplane:~$ kubectl drain node01 --ignore-daemonsets
node/node01 cordoned
Warning: ignoring DaemonSet-managed Pods: kube-system/canal-qg59v, kube-system/kube-proxy-w7jm2
node/node01 drained


controlplane:~$ ssh node01
Last login: Mon May  5 16:42:00 2025 from 10.244.19.53
node01:~$ sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.32.4-1.1' kubectl='1.32.4-1.1' && \
sudo apt-mark hold kubelet kubectl
```

&nbsp;

Output:

<details>
<summary> Large output</summary>

```bash
node01:~$ sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.32.4-1.1' kubectl='1.32.4-1.1' && \
sudo apt-mark hold kubelet kubectl
kubelet was already not on hold.
kubectl was already not on hold.
Get:1 http://security.ubuntu.com/ubuntu noble-security InRelease [126 kB]
Hit:2 http://archive.ubuntu.com/ubuntu noble InRelease                                               
Get:3 http://archive.ubuntu.com/ubuntu noble-updates InRelease [126 kB]                              
Hit:5 http://archive.ubuntu.com/ubuntu noble-backports InRelease                                     
Hit:4 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.32/deb  InRelease
Get:6 http://security.ubuntu.com/ubuntu noble-security/restricted amd64 Packages [960 kB]
Get:7 http://security.ubuntu.com/ubuntu noble-security/restricted Translation-en [197 kB]
Get:8 http://security.ubuntu.com/ubuntu noble-security/multiverse amd64 Packages [17.7 kB]
Get:9 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 Packages [1057 kB]
Get:10 http://archive.ubuntu.com/ubuntu noble-updates/universe amd64 Packages [1060 kB]
Fetched 3544 kB in 2s (2034 kB/s)                               
Reading package lists... Done
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following package was automatically installed and is no longer required:
  squashfs-tools
Use 'sudo apt autoremove' to remove it.
The following packages will be upgraded:
  kubectl kubelet
2 upgraded, 0 newly installed, 0 to remove and 153 not upgraded.
Need to get 26.4 MB of archives.
After this operation, 8192 B of additional disk space will be used.
Get:1 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.32/deb  kubectl 1.32.4-1.1 [11.2 MB]
Get:2 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.32/deb  kubelet 1.32.4-1.1 [15.2 MB]
Fetched 26.4 MB in 1s (29.2 MB/s)
(Reading database ... 82217 files and directories currently installed.)
Preparing to unpack .../kubectl_1.32.4-1.1_amd64.deb ...
Unpacking kubectl (1.32.4-1.1) over (1.32.1-1.1) ...
Preparing to unpack .../kubelet_1.32.4-1.1_amd64.deb ...
Unpacking kubelet (1.32.4-1.1) over (1.32.1-1.1) ...
Setting up kubectl (1.32.4-1.1) ...
Setting up kubelet (1.32.4-1.1) ...
Scanning processes...                                                                                                                                                                      
Scanning candidates...                                                                                                                                                                     
Scanning linux images...                                                                                                                                                                   

Running kernel seems to be up-to-date.

Restarting services...
 systemctl restart kubelet.service

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
kubelet set on hold.
kubectl set on hold.
```


</details>

&nbsp;

Restart the services and Uncordon the Node:
```bash
sudo systemctl daemon-reload
sudo systemctl restart kubelet  

controlplane:~$ kubectl uncordon node01
node/node01 uncordoned
```

&nbsp;

Get the Node Status:

```bash
NAME           STATUS   ROLES           AGE    VERSION
controlplane   Ready    control-plane   7d4h   v1.32.4
node01         Ready    <none>          7d4h   v1.32.4
```

&nbsp;

And that's it!.