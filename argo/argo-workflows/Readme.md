## **Argo Workflows**

In this document, we demonstrate how to install Argo Workflows on GKE. Then, we will show some use cases.

**NOTE:** It is very important to verify the cluster pod pool configuration on GKE before Argo Workflows installation. You should make sure that `Nodes/ImageType` is set to `Container-Optimized OS with Docker (cos)`. You can apply this configuration in cluster settings.

**_If you don't use this configuration, workflows get stuck and never finish._**

```
Clusters -> Nodes -> Node Pools -> Nodes/ImageType
```

---

On GKE we need a `clusterrolebinding` RBAC. Let's create it:

```
kubectl create clusterrolebinding YOURNAME-cluster-admin-binding --clusterrole=cluster-admin --user=YOUREMAIL@gmail.com
```

Then, we will need a namespace to deploy argo workflow.

```
kubectl create ns argo
```

The latest version at the time of writing this document is v3.2.6. More details are [here](https://github.com/argoproj/argo-workflows/releases/tag/v3.2.6). If you don't need any extra configurations, you can install the argo workflow with the following command:

```
kubectl apply -n argo -f https://github.com/argoproj/argo-workflows/releases/download/v3.2.6/install.yaml
```

However, I recommend to download the installation yaml file and deploy from local or make your desired changes to the manifest, based on the cluster and then apply the manifest. For example, in our case, we are installing the argo workflow on GKE `Autopilot` and based on the argo workflow documentations, on GKE autopilot, only `emissary` and `k8sapi` executors are supported. `emissary` is the default. We do not need to change anything.
Download the configmap from [here](https://argoproj.github.io/argo-workflows/workflow-controller-configmap.yaml)

**_The executor to be used in your workflows can be changed in the `configmap` under the `containerRuntimeExecutor` key._**

## **Security**

Argo uses serviceaccount to access resources on kubernetes. You can specify the serviceaccount to be used in argo workflows. To do this, while creating workflows, mention the serviceaccount name that you want to use:

```
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: basic-wf-
  labels:
    workflows.argoproj.io/archive-strategy: "false"
spec:
  entrypoint: hello
  serviceAccountName: argo
```

Also, keep in mind that this service account should have suffcient privileges. For test purposes, you can use the following command. **!!!NOT SECURE!!!**

```
kubectl create rolebinding default-admin --clusterrole=admin --serviceaccount=argo:default -n argo
```

With this command you are giving the admin permission to argo serviceaccount.

## **Access the Argo Workflows UI**

To access the UI, there are different methods based on your environment and choice.

- Port Forwarding
- LoadBalancer
- Ingress

If you installed argo workflows on your local cluster, you can easily access the UI using the following command:

```
kubectl -n argo port-forward svc/argo-server 2746:2746
```

Or, if you're using a remote cluster you can choose one the following methods:

**Expose a LoadBalancer**

```
kubectl patch svc argo-server -n argo -p '{"spec": {"type": "LoadBalancer"}}'
```

Then you should wait for an external IP to be assigned to your service. You can see the service using the following command:

```
kubectl get svc argo-server -n argo
```

## **Ingerss**

In this guide, we will be configuring `nginx.ingress` on GKE to access to the Argo Workflows UI.
[Ingress Configuration](./ingress-configuration.md)

## **Install CLI**

We need to CLI to get authorization token to login to the UI. So, let's install the CLI.

```
# Download the binary
curl -sLO https://github.com/argoproj/argo-workflows/releases/download/v3.2.6/argo-linux-amd64.gz

# Unzip
gunzip argo-linux-amd64.gz

# Make binary executable
chmod +x argo-linux-amd64

# Move binary to path
mv ./argo-linux-amd64 /usr/local/bin/argo

# Test installation
argo version
```

Then, you can get a login token with the following command:

```
argo auth token
```

The rest of the document is as follow:

- [Core Concepts and Workflow Templates](./templates.md)
- [Variables](./variables.md)
- [Service Account](./service-account.md)
