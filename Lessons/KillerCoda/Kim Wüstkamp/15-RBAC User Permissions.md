## 15-RBAC User Permissions.md

- There is existing Namespace applications.
  - User smoke should be allowed to create and delete Pods, Deployments and StatefulSets in Namespace applications.
  - User smoke should have view permissions (like the permissions of the default ClusterRole named view) in all Namespaces but not in kube-system.
  - Verify everything using kubectl auth can-i.
  
-------------------------------------------------------------------------------

### Demo

I'll start by checking the existing Namespaces:

```bash
controlplane:~$ k get ns
NAME                 STATUS   AGE
applications         Active   15m
default              Active   29d
kube-node-lease      Active   29d
kube-public          Active   29d
kube-system          Active   29d
local-path-storage   Active   29d
```
&nbsp;

Then, proceed with the first role, that should grant access to pods, deployments and statefulsets. That role will be binding to the user "smoke".
```bash
controlplane:~$ kubectl create role my-create-delete  --verb=get,list,watch,delete,create --resource=pods,deployments,statefulsets -n applications 
role.rbac.authorization.k8s.io/my-create-delete created
controlplane:~$ kubectl create rolebinding create-delete-rolebinding --role=my-create-delete --user=smoke -n applications
rolebinding.rbac.authorization.k8s.io/create-delete-rolebinding created
```
&nbsp;


As per the second item, and considering that K8s so far does not allow to set any -deny- permission, we can manually set the "view" ClusteRole to all existing Namespaces, except the kube-system namespace.

```bash
controlplane:~$ kubectl create rolebinding view-rb --clusterrole=view --user=smoke -n applications                          
controlplane:~$ kubectl create rolebinding view-rb --clusterrole=view --user=smoke -n default
controlplane:~$ kubectl create rolebinding view-rb --clusterrole=view --user=smoke -n kube-node-lease
controlplane:~$ kubectl create rolebinding view-rb --clusterrole=view --user=smoke -n kube-public
```
&nbsp;

And that's it. Keep the verification commands handy.


### Verification and expected output:
```bash
# applications
k auth can-i create deployments --as smoke -n applications # YES
k auth can-i delete deployments --as smoke -n applications # YES
k auth can-i delete pods --as smoke -n applications # YES

k auth can-i delete sts --as smoke -n applications # YES
k auth can-i delete secrets --as smoke -n applications # NO
k auth can-i list deployments --as smoke -n applications # YES

k auth can-i list secrets --as smoke -n applications # NO
k auth can-i get secrets --as smoke -n applications # NO

# view in all namespaces but not kube-system
k auth can-i list pods --as smoke -n default # YES
k auth can-i list pods --as smoke -n applications # YES
k auth can-i list pods --as smoke -n kube-public # YES

k auth can-i list pods --as smoke -n kube-node-lease # YES
k auth can-i list pods --as smoke -n kube-system # NO
```