### Join Cluster Node

- The Node node01 is not yet part of the cluster, join it using kubeadm.

-----------------

This is a very simple exercise. First, will get the "join" command from control-plane.

Then, execute the very same command from the node.

Check.

That's it!.

```bash
# Show Join Command
controlplane:~$ kubeadm token create --print-join-command
kubeadm join 172.30.1.2:6443 --token 9dvy0g.gw6cqgum0ifvqao6 --discovery-token-ca-cert-hash sha256:42f649e6dafad7a214b094d456ee585ed7968a47be023edd39051f11b577ffb2 

# Get existing Nodes
controlplane:~$ k get nodes
NAME           STATUS   ROLES           AGE    VERSION
controlplane   Ready    control-plane   7d7h   v1.32.1

# Connect to Node01
controlplane:~$ ssh node01
Last login: Mon Feb 10 22:06:42 2025 from 10.244.0.131

# Run the -join- command
node01:~$ kubeadm join 172.30.1.2:6443 --token 9dvy0g.gw6cqgum0ifvqao6 --discovery-token-ca-cert-hash sha256:42f649e6dafad7a214b094d456ee585ed7968a47be023edd39051f11b577ffb2
[preflight] Running pre-flight checks
[preflight] Reading configuration from the "kubeadm-config" ConfigMap in namespace "kube-system"...
[preflight] Use 'kubeadm init phase upload-config --config your-config.yaml' to re-upload it.
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-check] Waiting for a healthy kubelet at http://127.0.0.1:10248/healthz. This can take up to 4m0s
[kubelet-check] The kubelet is healthy after 1.002634306s
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.

node01:~$ 
logout
Connection to node01 closed.

# Back to control plane, check the existing nodes.
controlplane:~$ k get nodes
NAME           STATUS   ROLES           AGE    VERSION
controlplane   Ready    control-plane   7d7h   v1.32.1
node01         Ready    <none>          5s     v1.32.1
```

&nbsp;

And that's it!.