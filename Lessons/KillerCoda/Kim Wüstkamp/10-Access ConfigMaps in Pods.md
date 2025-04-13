### Access ConfigMaps in Pods

1st part:
- Create a ConfigMap named trauerweide with content tree=trauerweide
- Create the ConfigMap stored in existing file /root/cm.yaml

2nd part:
- Create a Pod named pod1 of image nginx:alpine
- Make key tree of ConfigMap trauerweide available as environment variable TREE1
- Mount all keys of ConfigMap birke as volume. The files should be available under /etc/birke/*
- Test env+volume access in the running Pod

-------------------------------------------------------------------------

### Demo

### 1st Part:

```bash
k create configmap trauerweide --dry-run=client -o yaml >> trauerweide.yaml

k apply -f trauerweide.yaml

k apply -f cm.yaml
```

```YAML
apiVersion: v1
kind: ConfigMap
metadata:
  creationTimestamp: null
  name: trauerweide
data:
    tree: trauerweide
```
&nbsp;


cm.yaml
```YAML
apiVersion: v1
data:
  tree: birke
  level: "3"
  department: park
kind: ConfigMap
metadata:
  name: birke
```
&nbsp;

--------------

### 2nd Part:

For this 2nd part, it will be very useful to have this [Docs](https://kubernetes.io/docs/concepts/configuration/configmap/) handy.


Taking [this](https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/configmap/configure-pod.yaml) as example, let's produce a version that matches the requirements.

<details>

  <summary>Docs example</summary>

```YAML
apiVersion: v1
kind: Pod
metadata:
  name: configmap-demo-pod
spec:
  containers:
    - name: demo
      image: alpine
      command: ["sleep", "3600"]
      env:
        # Define the environment variable
        - name: PLAYER_INITIAL_LIVES # Notice that the case is different here
                                     # from the key name in the ConfigMap.
          valueFrom:
            configMapKeyRef:
              name: game-demo           # The ConfigMap this value comes from.
              key: player_initial_lives # The key to fetch.
        - name: UI_PROPERTIES_FILE_NAME
          valueFrom:
            configMapKeyRef:
              name: game-demo
              key: ui_properties_file_name
      volumeMounts:
      - name: config
        mountPath: "/config"
        readOnly: true
  volumes:
  # You set volumes at the Pod level, then mount them into containers inside that Pod
  - name: config
    configMap:
      # Provide the name of the ConfigMap you want to mount.
      name: game-demo
      # An array of keys from the ConfigMap to create as files
      items:
      - key: "game.properties"
        path: "game.properties"
      - key: "user-interface.properties"
        path: "user-interface.properties"
        
```

</details>

Considering the objectives and the given example:

```YAML
apiVersion: v1
kind: Pod
metadata:
  name: pod1
spec:
  volumes:
  # You set volumes at the Pod level, then mount them into containers inside that Pod
  - name: birke
    configMap:
      # Provide the name of the ConfigMap you want to mount.
      name: birke
  containers:
    - name: nginx
      image: nginx:alpine
      env:
        # Define the environment variable
        - name: TREE1 # Notice that the case is different here
                                     # from the key name in the ConfigMap.
          valueFrom:
            configMapKeyRef:
              name: trauerweide           # The ConfigMap this value comes from.
              key:  tree # The key to fetch.
      volumeMounts:
      - name: birke
        mountPath: /etc/birke/
        readOnly: true

```

&nbsp;


Provided solution:

<details>
<summary> The given solution.</summary>

```YAML
apiVersion: v1
kind: Pod
metadata:
  name: pod1
spec:
  volumes:
  - name: birke
    configMap:
      name: birke
  containers:
  - image: nginx:alpine
    name: pod1
    volumeMounts:
      - name: birke
        mountPath: /etc/birke
    env:
      - name: TREE1
        valueFrom:
          configMapKeyRef:
            name: trauerweide
            key: tree
```
</details>


#### Verification 

To verify the achievements, execute the following:

```bash
kubectl exec pod1 -- env | grep "TREE1=trauerweide"
kubectl exec pod1 -- cat /etc/birke/tree
kubectl exec pod1 -- cat /etc/birke/level
kubectl exec pod1 -- cat /etc/birke/department
```