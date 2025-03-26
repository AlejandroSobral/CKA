## Lesson 12


- Question 1: Creating a Kubernetes Cluster
Create a 3-node Kubernetes cluster, using one control plane node and 2 worker nodes.

**Note**: This is out of the exam scope, therefore, it will be skipped.

- Question 2: Scheduling a Pod
Schedule a Pod with the name lab123 that runs the Nginx and redis applications.

- Question 3: Managing Application Initialization
Create a deployment with the name lab1234deploy which runs the Nginx image, but waits 30 seconds before starting the actual Pods.


- Question 4: Setting up Persistent Storage
Create a Persistent Volume with the name lab123 that uses HostPath on the directory /lab125.


- Question 5: Configuring Application Access
Create a Deployment with the name lab126deploy, running 3 instances of the Nginx image.
Configure it such that it can be accessed by external users on port 32567 on each cluster node.


- Question 6: Securing Network Traffic
Create a Namespace with the name restricted, and configure it such that it only allows access to Pods exposing port 80 for Pods coming from the Namespaces access.


- Question 7: Setting up Quota
Create a Namespace with the name limited and configure it such that only 5 Pods can be started and the total amount of available memory for applications running in that Namespace is limited to 2 GiB.
Run a webserver Deployment with the name lab128deploy and using 3 Pods in this Namespace.
Each of the Pods should request 128MiB memory and be limited to 256MiB.


- Question 8: Creating a Static Pod
Configure a Pod with the name lab129pod that will be started by the kubelet on node worker2 as a static Pod.


- Question 9: Troubleshooting Node Services
Assume that node worker2 is not currently available. Ensure that the appropriate service is started on that node which will show the node as running.


- Question 10: Configuring Cluster Access
Create a ServiceAccount that has permissions to create Pods, Deployments, DaemonSets and StatefulSets in the Namespace "access".


- Question 11: Configuring Taints and Tolerations
Configure node worker2 such that it will only allow Pods to run that have been configured with the setting type:db
After verifying this works, remove the node restriction to return to normal operation.
