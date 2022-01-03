## **Kubernetes Deployment**

Kubernetes deployment is an object that describes the desired state of our system. **deployment.yaml** file holds the desired state of the application.

### **PODs**

We can think of pods as virtual machines that are running our applications. Each pod can run multiple containers.

### Deploy

Use this commands to apply the deployment.yaml to the Kubernetes

```
kubectl apply -f {path-to-deployment.yaml}
```

Then let's see the deployment object

```
kubectl get deployments

kubectl describe deployments {deployment-object-name}
```

We can verify the pods that are created by deployment object.

```
kubectl get pods

kubectl get logs {pod-name}
```
