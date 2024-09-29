## Create the first Cluster using kubeadm


kubeadm: install & manage the cluster
kubelet: service that starts the Pods
kubectl: interface to run & manage applications

### Communications:

Pod-to-Pod: Handled by network plugin
Container-to-container: handled within the pod



### Commands for Lab

Execute this in Control-Plane & Worker nodes as well.
```bash
git clone https://github.com/sandervanvugt/cka
```
```bash
sudo cka/setup-container.sh ## Install CRI (Container Run-time Interface)
```
Do check using:

```bash
sudo systemctl status containerd | grep Active
```
```bash
sudo cka/setup-kubetools.sh 
```

Start the Kluster:

```bash
sudo kubeadm init
```
Setup up the clients (This will be prompted on the output message from previous command):
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

