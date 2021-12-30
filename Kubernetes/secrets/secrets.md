## **Decoupling Secrets**

It's always a good practice to decouple the secrets from the application source.
When developers deploy their application, they just need to refer to secret by its name. Kubernetes mounts the secrets to the pod and then the application can easily read the secret value by its name.

So, like the configuration, it's good practice to have a separate directory for secrets.

```
\app
    \configs
    \secrets
    \src
```

We can create secret object from a file:

```
kubectl apply -f {path-to-secret.yaml}
```

or

```
kubectl create secret generic --from-file {path-to-secret-file}
```

The secret file looks like this:
![secret.yaml](/images/secret-yaml.jpg)

Verify the secrets:

```
kubectl get secrets
```

Then, we should change the **deployment.yaml** to mount the secrets to pods.
![mount-secret](/images/secret-mount.jpg)

Then, we should apply the **deployment.yaml** again.

```
kubectl apply -f {path-to-deployment.yaml}
```
