```
kubectl
```

```
kubectl config --kubeconfig="C:\my-kubectl\config"
```

Kubectl saves config in the following path **(default)**:

```
${HOME}/.kube/config
```

**kubectl** reads the following env variable for any path to config file (comma-separated).
It's possible to merge config files.

```
$KUBECONFIG
```

## Context

Kubernetes works with active context. Use contexts for different environments. e.g: develop, staging, production

```
kubectl config current-context
kubectl config get-contexts
kubectl config use-context
```

## Namespaces

Kubernetes isolates objects within a namespace

```
kubectl get namespaces
kubectl get ns
kubectl create namespace {name}
kubectl get all -n {namespace-name}
```

## Describe

This command gives detailed information about an object. It's mostly useful for troubleshooting purposes.

```
kubectl describe {resource} {name} -n {namespace}
```

## Logs

Get logs of internal pods. Useful when the application is throwing errors.

```
kubectl logs {pod-name} -n {namespace}
```
