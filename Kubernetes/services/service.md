## **Services**

Kubernetes uses services to make connections between pods that are not in the same subnet. To make services work properly, couple of things should be done.

### **Labels**

Define lables in **deployment.yaml**

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-deploy
  labels:
    app: example-app
```

Every pod running using this deployment, has a label **example-app**
We can also see the labels of pods:

```
kubectl get pods --show-lebels
```

### **Service**

In the **service.yaml** file we define a service and then use a **selector** to choose the label the be chosen by this service. Then the service creates an endpoint for these lables.
So, make sure that the labels are correct and defined in the **deployment.yaml**

```
apiVersion: v1
kind: Service
metadata:
  name: example-service
  labels:
    app: example-app
spec:
  type: LoadBalancer
  selector:
    app: example-app
  ports:
    - protocol: TCP
      name: http
      port: 80
      targetPort: 5000
```

Then apply the **service.yaml**:

```
kubectl apply -f {path-to-service.yaml}
```
