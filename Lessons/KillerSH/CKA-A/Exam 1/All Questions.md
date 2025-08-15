### Question 1 

Solve this question on: ssh cka9412

You're asked to extract the following information out of kubeconfig file /opt/course/1/kubeconfig on cka9412:

- Write all kubeconfig context names into /opt/course/1/contexts, one per line
- Write the name of the current context into /opt/course/1/current-context
- Write the client-certificate of user account-0027 base64-decoded into /opt/course/1/cert



```bash
#1st
controlplane:~$ k config get-contexts
CURRENT   NAME                          CLUSTER      AUTHINFO           NAMESPACE
*         kubernetes-admin@kubernetes   kubernetes   kubernetes-admin 

#2nd
kubernetes-admin@kubernetes

#3rd 
cat /var/lib/kubelet/pki/kubelet-client-current.pem
```
----------------------------------------------

### Question 2

Solve this question on: ssh cka7968

Install the MinIO Operator using Helm in Namespace minio. Then configure and create the Tenant CRD:

- Create Namespace minio
- Install Helm chart minio/operator into the new Namespace. The Helm Release should be called minio-operator
- Update the Tenant resource in /opt/course/2/minio-tenant.yaml to include enableSFTP: true under features
- Create the Tenant resource from /opt/course/2/minio-tenant.yaml

----------------------------------------------------

### Question 3

Solve this question on: ssh cka3962

- There are two Pods named o3db-* in Namespace project-h800. The Project H800 management asked you to scale these down to one replica to save resources.

----------------------------------------------------

### Question 4

Solve this question on: ssh cka2556

Check all available Pods in the Namespace project-c13 and find the names of those that would probably be terminated first if the nodes run out of resources (cpu or memory).

- Write the Pod names into /opt/course/4/pods-terminated-first.txt.



- Manual scenario creation:

<details>

<summary>q4.yaml</summary>

```

---
apiVersion: v1
kind: Namespace
metadata:
  name: project-c13

---
apiVersion: v1
kind: Pod
metadata:
  name: pod-alpha
  namespace: project-c13
spec:
  containers:
  - name: container-one
    image: busybox
    command: ["sh", "-c", "echo 'Hello from Pod Alpha' && sleep 3600"]
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-beta
  namespace: project-c13
spec:
  containers:
  - name: container-two
    image: busybox
    command: ["sh", "-c", "echo 'Hello from Pod Beta' && sleep 3600"]
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-gamma
  namespace: project-c13
spec:
  containers:
  - name: container-three
    image: busybox
    command: ["sh", "-c", "echo 'Hello from Pod Gamma' && sleep 3600"]
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-delta
  namespace: project-c13
spec:
  containers:
  - name: container-four
    image: busybox
    command: ["sh", "-c", "echo 'Hello from Pod Delta' && sleep 3600"]
    resources:
      requests:
        memory: "128Mi"
        cpu: "500m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

</details>

```bash
controlplane:~$ k get pods --sort-by {.status.qosClass}  -o custom-columns='NAME:.metadata.name,QOS:.status.qosClass'
NAME        QOS
pod-alpha   BestEffort
pod-beta    BestEffort
pod-gamma   Burstable
pod-delta   Guaranteed
```

As per the output, the Alpha & Beta will be evicted first; gamma afterwards. Delta is guaranteed; as long as possible.

----------------------------------------------------

### Question 5

Solve this question on: ssh cka5774

Previously the application api-gateway used some external autoscaler which should now be replaced with a HorizontalPodAutoscaler (HPA). The application has been deployed to Namespaces api-gateway-staging and api-gateway-prod like this:

```bash
kubectl kustomize /opt/course/5/api-gateway/staging | kubectl apply -f -
kubectl kustomize /opt/course/5/api-gateway/prod | kubectl apply -f -
```

- Using the Kustomize config at /opt/course/5/api-gateway do the following:

  - Remove the ConfigMap horizontal-scaling-config completely
  - Add HPA named api-gateway for the Deployment api-gateway with min 2 and max 4 replicas. It should scale at 50% average CPU utilisation
  - In prod the HPA should have max 6 replicas
  - Apply your changes for staging and prod so they're reflected in the cluster


```bash
k autoscale -h

kubectl autoscale deployment prod --min=2 --max=4 --cpu-percentage=50
```

Sigue acá:
https://killer.sh/attendee/df264a28-5063-4d6f-a0dd-24574706967d/content
----------------------------------------------------

### Question 6

Solve this question on: ssh cka7968

Create a new PersistentVolume named safari-pv. It should have a capacity of 2Gi, accessMode ReadWriteOnce, hostPath /Volumes/Data and no storageClassName defined.

Next create a new PersistentVolumeClaim in Namespace project-t230 named safari-pvc . It should request 2Gi storage, accessMode ReadWriteOnce and should not define a storageClassName. The PVC should bound to the PV correctly.

Finally create a new Deployment safari in Namespace project-t230 which mounts that volume at /tmp/safari-data. The Pods of that Deployment should be of image httpd:2-alpine.


----------------------------------------------------
### Question 7

Solve this question on: ssh cka5774

The metrics-server has been installed in the cluster. Write two bash scripts which use kubectl:

- Script /opt/course/7/node.sh should show resource usage of Nodes
- Script /opt/course/7/pod.sh should show resource usage of Pods and their containers

----------------------------------------------------
### Question 8

Solve this question on: ssh cka3962

Your coworker notified you that node cka3962-node1 is running an older Kubernetes version and is not even part of the cluster yet.

- Update the node's Kubernetes to the exact version of the controlplane
- Add the node to the cluster using kubeadm



----------------------------------------------------

### Question 9

Solve this question on: ssh cka9412

There is ServiceAccount secret-reader in Namespace project-swan. Create a Pod of image nginx:1-alpine named api-contact which uses this ServiceAccount.

Exec into the Pod and use curl to manually query all Secrets from the Kubernetes Api.

Write the result into file /opt/course/9/result.json.

----------------------------------------------------

### Question 10

Create a new ServiceAccount processor in Namespace project-hamster. Create a Role and RoleBinding, both named processor as well. These should allow the new SA to only create Secrets and ConfigMaps in that Namespace.


----------------------------------------------------

### Question 11

Solve this question on: ssh cka2556

Use Namespace project-tiger for the following. Create a DaemonSet named ds-important with image httpd:2-alpine and labels id=ds-important and uuid=18426a0b-5f59-4e10-923f-c0e078e82462. The Pods it creates should request 10 millicore cpu and 10 mebibyte memory. The Pods of that DaemonSet should run on all nodes, also controlplanes.

----------------------------------------------------

### Question 12

Solve this question on: ssh cka2556

Implement the following in Namespace project-tiger:

- Create a Deployment named deploy-important with 3 replicas
- The Deployment and its Pods should have label id=very-important
- First container named container1 with image nginx:1-alpine
- Second container named container2 with image google/pause
- There should only ever be one Pod of that Deployment running on one worker node, use topologyKey: kubernetes.io/hostname for this

ℹ️ Because there are two worker nodes and the Deployment has three replicas the result should be that the third Pod won't be scheduled. In a way this scenario simulates the behaviour of a DaemonSet, but using a Deployment with a fixed number of replicas

----------------------------------------------------
### Question 13


Solve this question on: ssh cka7968

The team from Project r500 wants to replace their Ingress (networking.k8s.io) with a Gateway Api (gateway.networking.k8s.io) solution. The old Ingress is available at /opt/course/13/ingress.yaml.

Perform the following in Namespace project-r500 and for the already existing Gateway:

- Create a new HTTPRoute named traffic-director which replicates the routes from the old Ingress
- Extend the new HTTPRoute with path /auto which redirects to mobile if the User-Agent is exactly mobile and to desktop otherwise

The existing Gateway is reachable at http://r500.gateway:30080 which means your implementation should work for these commands:

```bash
curl r500.gateway:30080/desktop
curl r500.gateway:30080/mobile
curl r500.gateway:30080/auto -H "User-Agent: mobile" 
curl r500.gateway:30080/auto
```

----------------------------------------------------
### Question 14

Solve this question on: ssh cka9412

Perform some tasks on cluster certificates:

- Check how long the kube-apiserver server certificate is valid using openssl or cfssl. Write the expiration date into /opt/course/14/expiration. Run the kubeadm command to list the expiration dates and confirm both methods show the same one
- Write the kubeadm command that would renew the kube-apiserver certificate into /opt/course/14/kubeadm-renew-certs.sh

----------------------------------------------------

### Question 15

Solve this question on: ssh cka7968

There was a security incident where an intruder was able to access the whole cluster from a single hacked backend Pod.

To prevent this create a NetworkPolicy called np-backend in Namespace project-snake. It should allow the backend-* Pods only to:

- Connect to db1-* Pods on port 1111
- Connect to db2-* Pods on port 2222


Use the app Pod labels in your policy.

ℹ️ All Pods in the Namespace run plain Nginx images. This allows simple connectivity tests like: k -n project-snake exec POD_NAME -- curl POD_IP:PORT
ℹ️ For example, connections from backend-* Pods to vault-* Pods on port 3333 should no longer work



----------------------------------------------------

### Question 16


Solve this question on: ssh cka5774

The CoreDNS configuration in the cluster needs to be updated:

- Make a backup of the existing configuration Yaml and store it at /opt/course/16/coredns_backup.yaml. You should be able to fast recover from the backup
- Update the CoreDNS configuration in the cluster so that DNS resolution for SERVICE.NAMESPACE.custom-domain will work exactly like and in addition to SERVICE.NAMESPACE.cluster.local


Test your configuration for example from a Pod with busybox:1 image. These commands should result in an IP address:

```bash
nslookup kubernetes.default.svc.cluster.local
nslookup kubernetes.default.svc.custom-domain
```

----------------------------------------------------

### Question 17

Solve this question on: ssh cka2556

In Namespace project-tiger create a Pod named tigers-reunite of image httpd:2-alpine with labels pod=container and container=pod. Find out on which node the Pod is scheduled. Ssh into that node and find the containerd container belonging to that Pod.

Using command crictl:

- Write the ID of the container and the info.runtimeType into /opt/course/17/pod-container.txt
- Write the logs of the container into /opt/course/17/pod-container.log

ℹ️ You can connect to a worker node using ssh cka2556-node1 or ssh cka2556-node2 from cka2556