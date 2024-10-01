## Create the first Cluster using kubeadm


kubeadm: install & manage the cluster
kubelet: service that starts the Pods
kubectl: interface to run & manage applications

During the installation some steps failed and this was used as alternative:
https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

### Communications:

Pod-to-Pod: Handled by network plugin
Container-to-container: handled within the pod

## Pre-steps

Create a new user and make it sudoer:

Execute this in all nodes.

Create new user:
```bash
sudo adduser worker1
```
Make it sudoer:
```bash
sudo usermod -aG sudo worker1
```

### Commands for Lab

Execute this in all nodes.
```bash
git clone https://github.com/sandervanvugt/cka
```
Install CRI (Container Run-time Interface), on all nodes:
```bash
sudo cka/setup-container.sh 
```
Do check using (On Control node):

```bash
sudo systemctl status containerd | grep Active
```

Install kubetools on all nodes:
```bash
sudo cka/setup-kubetools.sh 
```

Start the Kluster on control node:

```bash
sudo kubeadm init
```
Setup up the cluster (This will be prompted on the output message from previous command):
```bash
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Verify:
```bash
kubectl get all
```

Install network addons:
```bash
kubectl apply -f cka/calico.yaml
```
Verify:
```bash
kubectl get pods -n  kube-system
```


(At client)
Take the output of "kubeadm init" and execute it on the Worker nodes:
```bash
sudo kubeadm join <IP:PORT> --token <token> --discovery-token-ca-cert-hash sha256:key
```

In case the join command needs to be prompted out again>
```bash
kubeadm token create --print-join-command
```

Check final status:
```bash
kubectl get nodes
```
Be aware the both nodes can take some time to be ready.

Now it is fully operative.
Check it by

```bash
kubectl run testpod --image=ngnix
```

```bash
kubectl get pods
```

