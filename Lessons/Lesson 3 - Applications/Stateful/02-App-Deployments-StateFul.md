## Stateful & Stateless Apps

Briefly, this run groups of nodes which mantains always the same identifier for Pods.
Useful when managing apps that requires persistent storage.

[StatefulSet Docs](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)

--------------------

The stateful example should be ran using MiniKube since it is required to have a Storage Provisioner, and at this point is not yet feasible to do it manually.



<details>
<summary> Install Mini-Kube (Ubuntu)</summary>

Install Docker first, as Mini-Kube needs drivers:
```bash
https://docs.docker.com/engine/install/ubuntu/
```
Consider installing this driver as well:
```bash
https://github.com/Mirantis/cri-dockerd
```
```bash
sudo usermod -aG docker $USER && newgrp docker
```
<details2>
<summary> Install Cri-DockerD on Ubuntu </summary2>
```bash
wget https://github.com/Mirantis/cri-dockerd/releases/download/v0.2.0/cri-dockerd-v0.2.0-linux-amd64.tar.gz
```
```bash
tar xvf cri-dockerd-v0.2.0-linux-amd64.tar.gz
```
```bash
sudo mv ./cri-dockerd /usr/local/bin/
```

Test it out:
```bash
cri-dockerd --help
```

Set the service:

```bash
wget https://raw.githubusercontent.com/Mirantis/cri-dockerd/master/packaging/systemd/cri-docker.service
wget https://raw.githubusercontent.com/Mirantis/cri-dockerd/master/packaging/systemd/cri-docker.socket
sudo mv cri-docker.socket cri-docker.service /etc/systemd/system/
sudo sed -i -e 's,/usr/bin/cri-dockerd,/usr/local/bin/cri-dockerd,' /etc/systemd/system/cri-docker.service
```

Check service status:

```bash
systemctl daemon-reload
systemctl enable cri-docker.service
systemctl enable --now cri-docker.socket
```

```bash
apt get install VirtualBox
```

```bash
minikube start --driver=docker
```

</details2>

```bash
https://minikube.sigs.k8s.io/docs/start/?arch=%2Flinux%2Fx86-64%2Fstable%2Fbinary+download

- curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
- sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64
```
</details>

---------------------------------

Apply Stateful:

```bash
kubectl apply -f 00-stateful-example.yaml
```

Verify:


```bash
kubectl get all
```