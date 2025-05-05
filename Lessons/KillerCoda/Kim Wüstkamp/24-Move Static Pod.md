## Move a static Pod to another Node

- Move static Pod resource-reserver-beta to Node controlplane.
- Also change the name to resource-reserver-v1.

-----------------------------------------------------

## Demo

Get the status of the Pods & Nodes:

```bash
controlplane:~$ k get pods -o wide
NAME                            READY   STATUS    RESTARTS   AGE   IP            NODE     NOMINATED NODE   READINESS GATES
resource-reserver-beta-node01   1/1     Running   0          13s   192.168.1.4   node01   <none>           <none>

controlplane:/etc/kubernetes/manifests$ k get nodes
NAME           STATUS   ROLES           AGE    VERSION
controlplane   Ready    control-plane   7d7h   v1.32.1
node01         Ready    <none>          7d7h   v1.32.1
```
&nbsp;

Get the Static Pods out of schedule in the node01:

```bash

controlplane:~$ ssh node01
Last login: Mon Feb 10 22:06:42 2025 from 10.244.0.131

# Copy the file
node01:~$ cp /etc/kubernetes/manifests/resource-reserver.yaml /root/bkp.yaml
node01:~$ ls
bkp.yaml  filesystem

# Take the Static Pod Out.
node01:~$ rm /etc/kubernetes/manifests/resource-reserver.yaml 
node01:~$ 
logout
Connection to node01 closed.

# Check Pods
controlplane:~$ k get pods
No resources found in default namespace.

# Copy the file
controlplane:~$ scp node01:/root/bkp.yaml /root/bkp.yaml
bkp.yaml     100%  190   143.1KB/s   00:00    

# Modify it to change the Pod Name
controlplane:~$ nano bkp.yaml 

# Set the Pod Into Schedule
controlplane:~$ cp bkp.yaml /etc/kubernetes/manifests/resource-reserver-v1.yaml

# Verify it is running
controlplane:~$ k get pods
NAME                                READY   STATUS    RESTARTS   AGE
resource-reserver-v1-controlplane   1/1     Running   0          7s
```
&nbsp;

And that's it!.