## **Application Configuration**

Application configuration should be portable. It means that the application configurations such as appsettings.json or settings.py should not be in the same folder as the application is. So, the better practice is to create a separate directory and put the application settings files into that directory. Like:

```
\app
    \src
    \config
        appsettings.json
```

### **ConfigMap**

Kubernetes creates a **configmap** object and mounts the config to the running pods directly. So, in this case the running application in the pod can read the config files.

To create the configmap object:

```
kubectl apply -f {path-to-configmap.yaml}
```

or we can create a configmap directly from a file:

```
kubectl create configmap {config-name} --from-file {path-to-config-file}
```

Verify the created configmaps:

```
kuebctl get configmaps

kubectl get configmaps {configmap-name} -o yaml
```

After creating the configmap, we should mount the configmap to the pod. To do this, we should update the **deployment.yaml**:

1. create a volume that points to the configmap
2. Mount the volume to the path that application code is reading

![configmap-mount](../images/configmap-mounting.jpg)

Then apply the changes:

```
kubectl apply -f {path-to-deployment.yaml}
```
