## DaemonSet HostPath Configurator

In K8s DaemonSets are often used to configure certain things on Nodes.

- Create a DaemonSet named configurator, it should:
  - be in Namespace configurator
  - use image bash
  - mount /configurator as HostPath volume on the Node it's running on
  - write aba997ac-1c89-4d64 into file /configurator/config on its Node via the command: section
  - be kept running using sleep 1d or similar after the file write command
  - There are no taints on any Nodes which means no tolerations are needed.


---------------------------------------------------------

### Demo


Having this [Documentation](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) into consideration, let's create the YAML for the DaemonSet:

```YAML

apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: daemonsets-lab
  namespace: configurator
  labels:
    k8s-app: configurator
spec:
  selector:
    matchLabels:
      name: configurator
  template:
    metadata:
      labels:
        name: configurator
    spec:
      containers:
      - name: configurator
        image: bash
        command:
          - sh
          - -c
          - 'echo aba997ac-1c89-4d64 > /mount/config && sleep 1d'
        volumeMounts:
        - name: configurator
          mountPath: /mount
      volumes:
      - name: configurator
        hostPath:
          path: /configurator
```
&nbsp;


To reinforce the idea of the VolumeMount, let's modify the file that resides in the Node and check the content in the Pod itself:
```bash
controlplane:/configurator$ k -n configurator exec daemonsets-lab-bq452 -- cat /mount/config
aba997ac-1c89-4d64
controlplane:/configurator$ echo "Test" >> config 
controlplane:/configurator$ k -n configurator exec daemonsets-lab-bq452 -- cat /mount/config
aba997ac-1c89-4d64
Test
```
&nbsp;