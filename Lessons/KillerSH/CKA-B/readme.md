
## To Do

- Get familiar with jobs
- Understand again how PV and PVCs work.
- What CDR are?


## Question 3

Assignment:
```
Node cka5248-node1 has been added to the cluster using kubeadm and TLS bootstrapping.
Find the Issuer and Extended Key Usage values on cka5248-node1 for:
Kubelet Client Certificate, the one used for outgoing connections to the kube-apiserver
Kubelet Server Certificate, the one used for incoming connections from the kube-apiserver
Write the information into file /opt/course/3/certificate-info.txt.
```


Resolution:
```bash
openssl x509 -in /var/lib/kubelet/pki/kubelet-client-current.pem -issuer -noout >> /opt/course/3/certificate-info.txt 
openssl x509 -in /var/lib/kubelet/pki/kubelet.crt -issuer -noout >> /opt/course/3/certificate-info.txt 

openssl x509 -in /var/lib/kubelet/pki/kubelet-client-current.pem -text |  grep -A 1 "Extended Key Usage"
            X509v3 Extended Key Usage: 
                TLS Web Client Authentication  ## This give the last answer to identify server/client side.

openssl x509 -in /var/lib/kubelet/pki/kubelet.crt -text |  grep -A 1 "Extended Key Usage"
            X509v3 Extended Key Usage: 
                TLS Web Server Authentication ## This give the last answer to identify server/client side.
```

## Question 6

```bash
Check the services files at /usr/lib/systemd/kubelet.service and /usr/lib/systemd/kubelet.service.d
```

Q10
    - No crea el volumen pero crea el PVC y el volumen se crea? WTF


## Question 10

```
There is a backup Job which needs to be adjusted to use a PVC to store backups.
Create a StorageClass named local-backup which uses provisioner: rancher.io/local-path and volumeBindingMode: WaitForFirstConsumer. To prevent possible data loss the StorageClass should keep a PV retained even if a bound PVC is deleted.
Adjust the Job at /opt/course/10/backup.yaml to use a PVC which request 50Mi storage and uses the new StorageClass.
Deploy your changes, verify the Job completed once and the PVC was bound to a newly created PV.
```

```bash
k create ns project-bern
```

```YAML
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-backup
provisioner: rancher.io/local-path
reclaimPolicy: Retain # default value is Delete
volumeBindingMode: WaitForFirstConsumer
```

```YAML
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
  namespace: project-bern
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 50Mi
  storageClassName: local-backup
```


```YAML
apiVersion: batch/v1
kind: Job
metadata:
  name: backup
  namespace: project-bern
spec:
  backoffLimit: 0
  template:
    spec:
      volumes:
        - name: backup
          emptyDir: {}
          persistentVolumeClaim:
             claimName: my-pvc
      containers:
        - name: bash
          image: bash:5
          command:
            - bash
            - -c
            - |
              set -x
              touch /backup/backup-$(date +%Y-%m-%d-%H-%M-%S).tar.gz
              sleep 15
          volumeMounts:
            - name: backup
              mountPath: /backup
      restartPolicy: Never
```

## Question 17


```
There is Kustomize config available at /opt/course/17/operator. It installs an operator which works with different CRDs. 
It has been deployed like this:
```

> kubectl kustomize /opt/course/17/operator/prod | kubectl apply -f -

```
Perform the following changes in the Kustomize base config:
```

- The operator needs to list certain CRDs. 
  - Check the logs to find out which ones and adjust the permissions for Role operator-role
  - Add a new Student resource called student4 with any name and description

- Deploy your Kustomize config changes to prod.


```YAML
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: classes.education.killer.sh
spec:
  group: education.killer.sh
...
---
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: students.education.killer.sh
spec:
  group: education.killer.sh
...
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: operator
  namespace: NAMESPACE_REPLACE
...
```

Prod:
```YAML
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: classes.education.killer.sh
spec:
  group: education.killer.sh
...
---
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: students.education.killer.sh
spec:
  group: education.killer.sh
...
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: operator
  namespace: operator-prod
```
