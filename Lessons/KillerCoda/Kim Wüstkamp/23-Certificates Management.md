### Kubernetes Certificate Management

- 1st part
  - Using kubeadm, read out the expiration date of the apiserver certificate and write it into /root/apiserver-expiration .

- 2nd part
  - Using kubeadm, renew the certificates of the apiserver and scheduler.conf


------------------------------------------

### Demo

#### 1st Part:

Using kubeadm -h, get that using "certs" the information can be easily obtanied:

```bash
controlplane:~$ kubeadm certs check-expiration | grep apiserver | head -n 1 >> /root/apiserver-expiration

controlplane:~$ cat /root/apiserver-expiration 
apiserver                  Apr 28, 2026 12:15 UTC   357d            ca                      no  
```

#### 2nd Part:

Learn how to renew certificates using the Docs:

```bash
controlplane:~$ kubeadm certs renew -h
Renew certificates for a Kubernetes cluster

Usage:
  kubeadm certs renew [flags]
  kubeadm certs renew [command]

Available Commands:
  admin.conf               Renew the certificate embedded in the kubeconfig file for the admin to use and for kubeadm itself
  all                      Renew all available certificates
  apiserver                Renew the certificate for serving the Kubernetes API
  apiserver-etcd-client    Renew the certificate the apiserver uses to access etcd
  apiserver-kubelet-client Renew the certificate for the API server to connect to kubelet
  controller-manager.conf  Renew the certificate embedded in the kubeconfig file for the controller manager to use
  etcd-healthcheck-client  Renew the certificate for liveness probes to healthcheck etcd
  etcd-peer                Renew the certificate for etcd nodes to communicate with each other
  etcd-server              Renew the certificate for serving etcd
  front-proxy-client       Renew the certificate for the front proxy client
  scheduler.conf           Renew the certificate embedded in the kubeconfig file for the scheduler manager to use
  super-admin.conf         Renew the certificate embedded in the kubeconfig file for the super-admin

controlplane:~$ kubeadm certs renew apiserver
[renew] Reading configuration from the "kubeadm-config" ConfigMap in namespace "kube-system"...
[renew] Use 'kubeadm init phase upload-config --config your-config.yaml' to re-upload it.

certificate for serving the Kubernetes API renewed
controlplane:~$ kubeadm certs renew scheduler.conf 
[renew] Reading configuration from the "kubeadm-config" ConfigMap in namespace "kube-system"...
[renew] Use 'kubeadm init phase upload-config --config your-config.yaml' to re-upload it.

certificate embedded in the kubeconfig file for the scheduler manager to use renewed
```
&nbsp;

And that's it!.