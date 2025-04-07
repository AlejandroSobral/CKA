## 3 - Apicrash Server

The idea here is to misconfigure the Apiserver in different ways, then check possible log locations for errors.

You should be very comfortable with situations where the Apiserver is not coming back up.

Configure the Apiserver manifest with a new argument --this-is-very-wrong .

Check if the Pod comes back up and what logs this causes.

Fix the Apiserver again

---------------------------------------------------------------

```bash
#########################
##                     ##
##   $ Killer Shell    ##
##                     ##
#########################
Initialising Kubernetes... done
Initialising Scenario... done
controlplane:~$ alias k='kubectl'                                                           ## SET COMMANDS ALIAS
controlplane:~$ k get pods                                                                  ## Test the Alias and get Pods
No resources found in default namespace.
controlplane:~$ cp /etc/kubernetes/manifests/kube-apiserver.yaml ~/kube-apiserver.yaml.ori  ## Copy API SERVER Yaml into Home directory
controlplane:~$ ls
filesystem  kube-apiserver.yaml.bkp                                                         ## Check bkp file
controlplane:~$ nano /etc/kubernetes/manifests/kube-apiserver.yaml
```

Add the --this-is-very-wrong flag:
```YAML
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 172.30.1.2:6443
  creationTimestamp: null
  labels:
    component: kube-apiserver
    tier: control-plane
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - --this-is-very-wrong
```

&nbsp;

```bash
controlplane:~$ k get pods                                  # Check if API works
The connection to the server 172.30.1.2:6443 was refused - did you specify the right host or port?  # As expected, does not work
controlplane:~$ cp ~/kube-apiserver.yaml.bkp /etc/kubernetes/manifests/kube-apiserver.yaml
controlplane:~$ watch crictl ps                             # Check if API comes back
controlplane:~$ k get pods
No resources found in default namespace.                    # Working fine again.
less /var/log/pods/kube-system_kube-apiserver-controlplane_55490d3d587b6f101f6b891c9b153bdd/kube-apiserver/0.log ## Check logs
```

&nbsp;

That's it!

For additional log checkings:

```bash
crictl ps # maybe run a few times, because the apiserver container get's restarted
crictl logs e1eaf1575184c

CONTAINER      IMAGE          CREATED        STATE    NAME                      ATTEMPT   POD ID              POD                                       NAMESPACE

e1eaf1575184c  95c0bda56fc4d  11 minutes ago Running  kube-apiserver            0         4a4360bd74501       kube-apiserver-controlplane               kube-system

c433795735fcc  f9c3c1813269c  11 minutes ago Running  calico-kube-controllers   4         5570eaa8397dc       calico-kube-controllers-fdf5f5495-dgc76   kube-system

c37170c37120c  2b0d6572d062c  13 minutes ago Running  kube-scheduler            2         6677be01194ee       kube-scheduler-controlplane               kube-system

da03b6659f10f  019ee182b58e2  13 minutes ago Running  kube-controller-manager   2         b00a6afb315b4       kube-controller-manager-controlplane      kube-system

df184af458b68  c69fa2e9cbf5f  2 hours ago    Running  coredns                   1         85bcbca1e8200       coredns-7695687499-2bqjp                  kube-system

f7c6950543886  9634f4b99547a  2 hours ago    Running  local-path-provisioner    1         72776f4e07598       local-path-provisioner-5c94487ccb-bm6vp   local-path-storage

308135bd5254b  c69fa2e9cbf5f  2 hours ago    Running  coredns                   1         b61a0d5f0d295       coredns-7695687499-glfxn                  kube-system

f70d108f5bcfc  e6ea68648f0cd  2 hours ago    Running  kube-flannel              1         b0f120dd3bb8d       canal-r5r24                               kube-system

92413e1f17edf  75392e3500e36  2 hours ago    Running  calico-node               1         b0f120dd3bb8d       canal-r5r24                               kube-system

68d1982a78b42  e29f9c7391fd9  2 hours ago    Running  kube-proxy                1         372c4a80d45e2       kube-proxy-f7jnk                          kube-system

dc83d27a804d0  a9e7e6b294baf  2 hours ago    Running  etcd                      2         2536ae9a37f88       etcd-controlplane 

> Error while dialing dial tcp: address this-is-very-wrong: missing port in address. Reconnecting...
```

```bash
# syslogs
journalctl | grep apiserver # nothing specific
cat /var/log/syslog | grep apiserver # nothing specific
```
