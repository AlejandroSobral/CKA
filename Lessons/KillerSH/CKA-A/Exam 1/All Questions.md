## Resources
- https://cloudutsuk.com/posts/certification/cka/cka-pratice-test-1/


## TO DO

Answer remaining questions.

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

```bash
k create ns minio

helm install minio minio/operator -n minio

helm repo add minio-operator https://operator.min.io ## Configured during the exam
```

```YAML
apiVersion: minio.min.io/v2
kind: Tenant
metadata:
  name: tenant
  namespace: minio
  labels:
    app: minio
spec:
  features:
    bucketDNS: false
    enableSFTP: true                     # ADD
  image: quay.io/minio/minio:latest
  pools:
    - servers: 1
      name: pool-0
      volumesPerServer: 0
      volumeClaimTemplate:
        apiVersion: v1
        kind: persistentvolumeclaims
        metadata: { }
        spec:
          accessModes:
            - ReadWriteOnce
          resources:
            requests:
              storage: 10Mi
          storageClassName: standard
        status: { }
  requestAutoCert: true
```


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

```YAML

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

Next create a new PersistentVolumeClaim in Namespace project-t230 named safari-pvc. It should request 2Gi storage, accessMode ReadWriteOnce and should not define a storageClassName. The PVC should bound to the PV correctly.

Finally create a new Deployment safari in Namespace project-t230 which mounts that volume at /tmp/safari-data. The Pods of that Deployment should be of image httpd:2-alpine.

<details>

<summary>q6.yaml</summary>

```YAML
apiVersion: v1
kind: PersistentVolume
metadata:
  name: safari-pv
spec:
  capacity:
    storage: 2Gi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  hostPath:
    path: "/Volumes/Data"
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: safari-pvc
  namespace: project-t230
spec:
  accessModes:
  - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 2Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: q5-deploy
  name: q5-deploy
  namespace: project-t230
spec:
  replicas: 1
  selector:
    matchLabels:
      app: q5-deploy
  template:
    metadata:
      labels:
        app: q5-deploy
    spec:
      volumes:
      - name: safari-pv
        persistentVolumeClaim:
          claimName: safari-pvc
      containers:
      - name: httpd
        image: httpd:2-alpine
        volumeMounts:
        - mountPath: "/tmp/safari-data"
          name: safari-pv
```

Check the status:
```bash
controlplane:~# k get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                     STORAGECLASS   VOLUMETRIBUTESCLASS   REASON   AGE
pvc-3203d2c9-c3a4-4172-85e5-7ab67291a226   2Gi        RWO            Delete           Bound       project-t230/safari-pvc   local-path     <unset>               15s
safari-pv                                  2Gi        RWO            Retain           Available                               <unset>        <unset>                        6m1s
controlplane:~# k get pvc
NAME          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMETRIBUTESCLASS   AGE
safari-pvc    Bound    pvc-3203d2c9-c3a4-4172-85e5-7ab67291a226   2Gi        RWO            local-path     <unset>               5m30s
controlplane:~#
```

</details>




----------------------------------------------------
### Question 7

Solve this question on: ssh cka5774

The metrics-server has been installed in the cluster. Write two bash scripts which use kubectl:

- Script /opt/course/7/node.sh should show resource usage of Nodes
- Script /opt/course/7/pod.sh should show resource usage of Pods and their containers


To make it work in any CKA simulator; consider the following:

#### Scenario creation

Resource:
 -  https://github.com/kubernetes-sigs/metrics-server/issues/917

```
curl -LJO kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
Add: --kubelet-insecure-tls
```

```YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server
  namespace: kube-system
spec:
  selector:
    matchLabels:
      k8s-app: metrics-server
  strategy:
    rollingUpdate:
      maxUnavailable: 0
  template:
    metadata:
      labels:
        k8s-app: metrics-server
    spec:
      containers:
      - args:
        - --cert-dir=/tmp
        - --secure-port=10250
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
        - --kubelet-insecure-tls
```



Resources to list:
	k create deployment my-dep --image=nginx --replicas=2 
Nodes:
	(Deploy in any two-nodes Cluster).
	
Scripts:
  ```bash
	#/bin/bash!
	kubectl top pods --containers=true
	```
	
  ```bash
	#/bin/bash!
	kubectl top nodes
  ```



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

<details>

<summary>  List resources using the API. </summary>

- Resource:
  - https://kubernetes.io/docs/tasks/run-application/access-api-from-pod/

```bash
k create sa my-sa
k create role my-role --verb=get,create,list --resource=pod
k create rolebinding my-rb --role=my-role --serviceaccount=default:my-sa
k run pod --image=nginx --dry-run=client -o yaml --command -- /bin/bash -c "tail -f /dev/null" > pod.yaml

## Add spec.serviceAccountName: my-sa
k apply -f pod.yaml 
VAR=$(k exec pod -it -- cat /var/run/secrets/kubernetes.io/serviceaccount/token)
k exec pod -it -- curl -k https://kubernetes.default/api/v1/namespaces/default/pods -H "Authorization: Bearer $VAR"
```

</details>

----------------------------------------------------

### Question 10

Create a new ServiceAccount "processor" in Namespace project-hamster. 
Create a Role and RoleBinding, both named processor as well. These should allow the new SA to only create Secrets and ConfigMaps in that Namespace.

<details>
<summary> Q10 resolution </summary>

RBAC entities creation:
```bash
k create sa processor ## ServiceAccount
k create role processor --verb=create  --resource=secret,configmaps # Role
k create rolebinding processor --role=processor --serviceaccount=default:processor # RoleBinding
```
Confirmation:
```bash
controlplane:~$ k auth can-i create secrets --as=system:serviceaccount:default:processor
yes
controlplane:~$ k auth can-i create configmap --as=system:serviceaccount:default:processor
yes
```

</details>

----------------------------------------------------

### Question 11

Solve this question on: ssh cka2556

Use Namespace project-tiger for the following. Create a DaemonSet named ds-important with image httpd:2-alpine and labels id=ds-important and uuid=18426a0b-5f59-4e10-923f-c0e078e82462. The Pods it creates should request 10 millicore cpu and 10 mebibyte memory. The Pods of that DaemonSet should run on all nodes, also controlplanes.


```bash
controlplane:~$ k get pods -o wide -n project-tiger
NAME             READY   STATUS    RESTARTS   AGE     IP            NODE         NOMINATED NODE   READINESS GATES
ds-important-jg7k9   1/1     Running   0          11s     192.168.0.4   controlplane   <none>           <none>
ds-important-wj4c4   1/1     Running   0          11s     192.168.1.4   node01       <none>           <none>
```


```bash
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: ds-important
  namespace: project-tiger
  labels:
    id: ds-important
    uuid: 18426a0b-5f59-4e10-923f-c0e078e82462
spec:
  selector:
    matchLabels:
      id: ds-important
      uuid: 18426a0b-5f59-4e10-923f-c0e078e82462
  template:
    metadata:
      labels:
        id: ds-important
    spec:
      tolerations:
        # these tolerations are to have the daemonset runnable on control plane nodes
        # remove them if your control plane nodes should not run pods
        - key: node-role.kubernetes.io/control-plane
          operator: Exists
          effect: NoSchedule
        - key: node-role.kubernetes.io/master
          operator: Exists
          effect: NoSchedule
      containers:
        - name: project-tiger
          image: httpd:2-alpine
          resources:
            requests:
              cpu: 10m
              memory: 20Mi
```

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

```YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-important
  labels:
    app: deploy-important
spec:
  replicas: 3
  selector:
    matchLabels:
      id: deploy-important
  strategy: {}
  template:
    metadata:
      labels:
        id: deploy-important
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                  - key: id
                    operator: In
                    values:
                      - deploy-important
              topologyKey: kubernetes.io/hostname
      containers:
        - image: nginx
          name: nginx-alpine
          resources: {}
        - image: google/pause
          name: pause
status: {}
```

```bash
kubectl get pods -o wide
NAME                                READY   STATUS         RESTARTS   AGE   IP           NODE            NOMINATED NODE   READINESS GATES
deploy-important-7b5b6b4565-4jnz4   0/2     Pending        0          20s   <none>       <none>          <none>           <none>
deploy-important-7b5b6b4565-9t2ww   1/2     ErrImagePull   0          20s   10.244.0.4   talos-r7a-pwx   <none>           <none>
deploy-important-7b5b6b4565-zpnbf   1/2     ErrImagePull   0          20s   10.244.2.2   talos-126-mts   <none>           <none>
```

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

Setup scenario:
```bash
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.0.0/standard-install.yaml
k create ns project-r500
```

[Resources](https://gateway-api.sigs.k8s.io/api-types/httproute/#backendrefs-optional)

<details>
<summary>Provided YAML </summary>

```YAML
## Existing gateway
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: main
  namespace: project-r500
spec:
  gatewayClassName: nginx
  listeners:
  - allowedRoutes:
      namespaces:
        from: Same
    name: http
    port: 80
    protocol: HTTP
...


# cka7968:/opt/course/13/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: traffic-director
spec:
  ingressClassName: nginx
  rules:
    - host: r500.gateway
      http:
        paths:
          - backend:
              service:
                name: web-desktop
                port:
                  number: 80
            path: /desktop
            pathType: Prefix
          - backend:
              service:
                name: web-mobile
                port:
                  number: 80
            path: /mobile
            pathType: Prefix
```

The HTTPRoute to be created:

```YAML
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: traffic-director
spec:
  parentRefs:
  - name: main
  hostnames:
  - r500.gateway
  rules:

  - matches:
    - path:
        type: PathPrefix
        value: /mobile
    backendRefs:
    - name: web-mobile
      port: 80

  - matches:
    - path:
        type: PathPrefix
        value: /desktop
    backendRefs:
    - name: web-desktop
      port: 80

  - matches:
    - headers:
      - type: Exact
        name: User-Agent
        value: mobile
      path:
        type: PathPrefix
        value: /auto
    backendRefs:
    - name: web-mobile
      port: 80

  - matches:
      path:
        type: PathPrefix
        value: /auto
    backendRefs:
    - name: web-desktop
      port: 80
```

</details>

----------------------------------------------------
### Question 14

Solve this question on: ssh cka9412

Perform some tasks on cluster certificates:

- Check how long the kube-apiserver server certificate is valid using openssl or cfssl. Write the expiration date into /opt/course/14/expiration. Run the kubeadm command to list the expiration dates and confirm both methods show the same one
- Write the kubeadm command that would renew the kube-apiserver certificate into /opt/course/14/kubeadm-renew-certs.sh

</detail>
<summary> Resolution </summary>

```bash
openssl x509 -in apiserver.crt -enddate | grep notAfter
notAfter=Aug 19 09:03:32 2026 GMT

kubeadm certs check-expiration | grep apiserver
apiserver                  Aug 19, 2026 09:03 UTC 

kubeadm certs renew apiserver
[renew] Reading configuration from the "kubeadm-config" ConfigMap in namespace "kube-system"...
[renew] Use 'kubeadm init phase upload-config --config your-config-file' to re-upload it.

certificate for serving the Kubernetes API renewed
```

</details>

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


```YAML
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np-backend
spec:
  podSelector:
    matchLabels:
      app: my-pods
  policyTypes:
    - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          key: pod-label
    ports:
    - port: 1111
  - to:
    - podSelector:
        matchLabels:
          key: pod-label-2
    ports:
    - port: 2222
```


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


Solution:
```bash
# Get the configmap that determines the behaviour of CoreDNS:
k get cm -n kube-system coredns -o yaml > configmap.yaml

# Get the configuration of the deployment
k get deployment coredns -o yaml -n kube-system > coredns.yaml

# Run an example Pod to execute the nslookup
kubectl run my-pod --image=alpine:3.18 --command -- /bin/sh -c "sleep 3600"

# Original ConfigMap

apiVersion: v1
data:
  Corefile: |
    .:53 {
        errors
        health {
           lameduck 5s
        }
        ready
        kubernetes cluster.local in-addr.arpa ip6.arpa {
           pods insecure
           fallthrough in-addr.arpa ip6.arpa
           ttl 30
        }
        prometheus :9153
        forward . /etc/resolv.conf {
           max_concurrent 1000
        }
        cache 30 {
           disable success cluster.local
           disable denial cluster.local
        }
        loop
        reload
        loadbalance
    }
kind: ConfigMap
metadata:
  creationTimestamp: "2025-08-19T09:03:55Z"
  name: coredns
  namespace: kube-system
  resourceVersion: "227"
  uid: f6e43a2b-9471-450b-ae97-beca57093875
...
..
.

# Add the custom-domain line:

...
        kubernetes cluster.local custom-domain in-addr.arpa ip6.arpa {
           pods insecure
           fallthrough in-addr.arpa ip6.arpa
           ttl 30
        }
...

# Apply and rollout coredns Deployment
k -n kube-system rollout restart deploy coredns 
deployment.apps/coredns restarted

# Test the DNS resolution for Svc
k exec my-pod -it -- nslookup kubernetes.default.svc.cluster.local
Server:         10.96.0.10
Address:        10.96.0.10:53

Name:   kubernetes.default.svc.cluster.local
Address: 10.96.0.1


k exec my-pod -it -- nslookup kubernetes.default.svc.custom-domain
Server:         10.96.0.10
Address:        10.96.0.10:53

Name:   kubernetes.default.svc.custom-domain
Address: 10.96.0.1
```


----------------------------------------------------

### Question 17

Solve this question on: ssh cka2556

In Namespace project-tiger create a Pod named tigers-reunite of image httpd:2-alpine with labels pod=container and container=pod. Find out on which node the Pod is scheduled. Ssh into that node and find the containerd container belonging to that Pod.

Using command crictl:

- Write the ID of the container and the info.runtimeType into /opt/course/17/pod-container.txt
- Write the logs of the container into /opt/course/17/pod-container.log

ℹ️ You can connect to a worker node using ssh cka2556-node1 or ssh cka2556-node2 from cka2556

<details>

<summary> Q17 solution </summary>

```bash

# Create Namespace
controlplane:~$ k create ns project-tiger
namespace/project-tiger created
controlplane:~$ k config set-context --current --namespace project-tiger 
Context "kubernetes-admin@kubernetes" modified.

# Run a pod with its labels
controlplane:~$ k run tigers-reunite --image=httpd:2-alpine --labels="pod=container,container=pod"
pod/tigers-reunite created

# Look for the Node
controlplane:~$ k get pods -o wide
NAME             READY   STATUS    RESTARTS   AGE   IP            NODE     NOMINATED NODE   READINESS GATES
tigers-reunite   1/1     Running   0          13s   192.168.1.5   node01   <none>           <none>

# Get containerID from describe command (1st method, but from controlplane node)
controlplane:~$ k describe pod tigers-reunite | grep containerd
    Container ID:   containerd://e8aafae87e09eca8a49b12197bf86854d9ebc877fd642271c56653a1a0dfbfce

# Connect to node01
controlplane:~$ ssh node01
Last login: Sun Aug 24 17:37:47 2025 from 10.244.5.113

# Get container ID from checking containers logs (2nd)
node01:/var/log/containers$ ls
...
..
tigers-reunite_project-tiger_tigers-reunite-e8aafae87e09eca8a49b12197bf86854d9ebc877fd642271c56653a1a0dfbfce.log ## ->> CointainerID e8aafae87e09eca8a49b12197bf86854d9ebc877fd642271c56653a1a0dfbfce
..
...

# Get ContainerID from crictl
node01:~$ crictl ps --output table --no-trunc | grep tigers-reunite
e8aafae87e09eca8a49b12197bf86854d9ebc877fd642271c56653a1a0dfbfce   sha256:413c1ac706f85ca3fe6b5fb02e8d77fd8368eea52cfaa8d4542e7c3b9025d947   44 minutes ago      Running             tigers-reunite      0                   17c6746c762a0e1e902ef6fbd232952450fa95e3f0d7f4effacb548d69efff20   tigers-reunite             project-tiger

## ContainerID into the file
echo "e8aafae87e09eca8a49b12197bf86854d9ebc877fd642271c56653a1a0dfbfce" >> /opt/course/17/pod-container.txt


node01:~$ crictl inspect e8aafae87e09eca8a49b12197bf86854d9ebc877fd642271c56653a1a0dfbfce | grep runtimeType
    "runtimeType": "io.containerd.runc.v2",

## Container logs
node01:~$ crictl logs e8aafae87e09eca8a49b12197bf86854d9ebc877fd642271c56653a1a0dfbfce
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 192.168.1.5. Set the 'ServerName' directive globally to suppress this message
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 192.168.1.5. Set the 'ServerName' directive globally to suppress this message
[Sun Aug 24 17:35:44.373333 2025] [mpm_event:notice] [pid 1:tid 1] AH00489: Apache/2.4.65 (Unix) configured -- resuming normal operations
[Sun Aug 24 17:35:44.378257 2025] [core:notice] [pid 1:tid 1] AH00094: Command line: 'httpd -D FOREGROUND'
```

</details>


----------------------------------

Extras:

<details>

<summary>  Local Talos setup </summary>



```bash
export CONTROL_PLANE_NODE=192.168.1.22
export WORKER_NODE_1=192.168.1.23
export WORKER_NODE_2=192.168.1.24

talosctl gen config TinyCluster https://$CONTROL_PLANE_NODE:6443 --output-dir _out
talosctl apply-config --insecure --nodes $CONTROL_PLANE_NODE --file _out/controlplane.yaml

talosctl config endpoint $CONTROL_PLANE_NODE
talosctl config node $CONTROL_PLANE_NODE
export TALOSCONFIG="_out/talosconfig"

talosctl bootstrap --nodes $CONTROL_PLANE_NODE

talosctl apply-config --insecure --nodes $WORKER_NODE_1 --file _out/worker.yaml
talosctl apply-config --insecure --nodes $WORKER_NODE_2 --file _out/worker.yaml

talosctl kubeconfig . --talosconfig _out/talosconfig
export KUBECONFIG=./kubeconfig

kubectl label node talos-qfk-og4  node-role.kubernetes.io/worker=worker
kubectl label node talos-126-mts  node-role.kubernetes.io/worker=worker
```

</details>



