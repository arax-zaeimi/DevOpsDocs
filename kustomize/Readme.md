## **Kustomize**

Flux supports **kustomize**.

In order to use the kustomization to apply changes to the cluster, instead of `kubectl apply -f ...`, use the following:

```
kubectl apply -k <path-to-kustomization-files>
```

We can use kustomize to create overlay yaml files (Patch).

It's a good practice to keep your manifest structure as follow:

```
app/
    src/
    kubernetes/
            ├── base
            │   ├── configmap.yml
            │   ├── deployment.yml
            │   ├── kustomization.yml
            │   └── service.yml
            └── overlays
                └── production
                    ├── configmap.yml
                    ├── deployment.yml
                    └── kustomization.yml

```

/base/deployment.yml

```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sammy-app
  namespace: default
  labels:
    app: sammy-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: sammy-app
  template:
    metadata:
      labels:
        app: sammy-app
    spec:
      containers:
      - name: server
        image: nginx:1.17
        volumeMounts:
          - name: sammy-app
            mountPath: /usr/share/nginx/html
        ports:
        - containerPort: 80
          protocol: TCP
        resources:
          requests:
            cpu: 100m
            memory: "128M"
          limits:
            cpu: 100m
            memory: "256M"
        env:
        - name: LOG_LEVEL
          value: "DEBUG"
      volumes:
      - name: sammy-app
        configMap:
          name: sammy-app
          items:
          - key: body
            path: index.html
```

/base/kustomization.yml

```
---
resources:
- configmap.yml
- deployment.yml
- service.yml
```

/overlays/production/kustomization.yml

```
---
bases:
- ../../base
patchesStrategicMerge:
- configmap.yml
- deployment.yml
```

/overlays/production/deployment.yml

```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sammy-app
  namespace: default
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: server
        resources:
          requests:
            cpu: 250m
            memory: "256M"
          limits:
            cpu: 1
            memory: "1G"
        env:
        - name: LOG_LEVEL
          value: "INFO"
```

Instruction from: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-manage-your-kubernetes-configurations-with-kustomize)
