# **Install Argo on GKE**

## Setup GCloud CLI on Docker container

In order to avoid conflicts with kubernetes configurations on the local machine, I will be using a docker to run gcloud commands and connect to GKE. You can skip this section and install `gcloud` directly on you machine. But, in some cases you might see `connection time out` or `x509 certificate` errors.

```
docker run -it --rm -v ${PWD}:/vdrive -w /vdrive --entrypoint /bin/bash google/cloud-sdk:latest
```

This docker contains `kubectl` already. But, if you need to install kubectl sdk, you can use the following:

```
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
chmod +x ./kubectl
mv ./kubectl /usr/local/bin/kubectl
```

This might be useful, if you want to create a GitHub action.

## **Login to gcloud**

```
gcloud auth login
```

## **Setup defaults (Optional)**

```
gcloud init
```

## **Set Default Project**

```
gcloud projects list
```

```
gcloud config set project <PROJECT_ID>
```

## **Create GKE Cluster**

```
PROJECT_ID=$(gcloud config list --format 'value(core.project)')

gcloud container clusters create <CLUSTER_NAME> \
  --release-channel regular \
  --workload-pool=$PROJECT_ID.svc.id.goog
```

Now, configrue `kubectl` to access gke cluster

```
gcloud container clusters get-credentials <CLUSTER_NAME>
```

This command, retrieves cluster information and adds a new entry to `.kubeconfig` file which is located at `{HOME}/.kube/.kubeconfig`. So, when you use the `kubectl` command, from your local command terminal, you can see the cluster which is created on GKE and run kubectl commands agains this cluster.

Then, create a namespace for argocd:

```
kubectl create ns <NAMESPACE>
```

## **Create Service Accounts on GKE & k8s**

Create Service Accounts on GKE:

```
gcloud iam service-accounts create <SERVICE_ACCOUNT>
```

Create service account on kubectl:

```
kubectl create serviceaccount --namespace <NAMESPACE> <SERVICE_ACCOUNT>
```

Link k8s and GKE service accounts:

```
gcloud iam service-accounts add-iam-policy-binding \
  --role roles/iam.workloadIdentityUser \
  --member "serviceAccount:$PROJECT_ID.svc.id.goog[<NAMESPACE>/<SERVICE_ACCOUNT>]" \
  <SERVICE_ACCOUNT>@$PROJECT_ID.iam.gserviceaccount.com
```

```
kubectl annotate serviceaccount \
  --namespace <NAMESPACE> <SERVICE_ACCOUNT> \
  iam.gke.io/gcp-service-account=<SERVICE_ACCOUNT>@$PROJECT_ID.iam.gserviceaccount.com
```

## **Install ArgoCD on cluster**

```
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

```
kubectl create clusterrolebinding cluster-admin-binding \
  --clusterrole=cluster-admin \
  --user="$(gcloud config get-value account)"
```

By default, ArgoCD does not expose external IP. So, you can enable the access following the methods metioned in Argo documentation. [Argo Documentation](https://argo-cd.readthedocs.io/en/stable/getting_started/). In this guide, we use **Port Forwarding** and **LoadBalancer**.

```
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```

Then expose the IP Port of ArgoCD using the following command: (This is not required if you are installing argocd on cloud such as GKE)

```
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

You can verify if argo resources are up and running in the cluster using:

```
kubectl get all -n argocd
```

**Tip:** for the production environment we need to configure the Ingress. [Ingress Configuration](https://argo-cd.readthedocs.io/en/stable/operator-manual/ingress/)

## Ingress (Optional) **Updates and review required**

```
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: argocd-server-ingress
  namespace: argocd
  annotations:
    kubernetes.io/ingress.class: gce
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    nginx.ingress.kubernetes.io/ssl-passthrough: "true"
spec:
  rules:
  - host: argocd.example.com
    http:
      paths:
      - backend:
          serviceName: argocd-server
          servicePort: https

```

```
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: hello-gke-ing
  annotations:
    kubernetes.io/ingress.class: gce
spec:
  rules:
  - http:
      paths:
      - path: /*
        backend:
          serviceName: hello-gke-ing
          servicePort: 80
```

## **Login to ArgoCD**

Argo initializes its admin user with a random password. This passowrd can be accessed using the following command:

```

kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo

```

Use username `admin` and the password to login:

```

argocd login localhost:8080

```

## **Register Cluster to ArgoCD (If Argo and Cluster are not local)**

Get current kubernetes clusters:

```

kubectl config get-contexts -o name

```

Choose a cluster and register it to the ArgoCD:

```

argocd cluster add <cluster-name>

```

## **Create an App on the Argo**

Now, we consider that there is a sample application on a git repository. We want to create an applicatino in the ArgoCD to sync the cluster with this application latest version. ArgoCD pulls the latest version and updates the cluster state.

```

argocd app create <application-name> \
--repo https://github.com/<repo-address> \
--revision <branch-name> \
--path <path-to-kustomization-manifests> \
--dest-server https://kubernetes.default.svc \
--dest-namespace default

```

- `dest-server` : should be the URL of the kubernetes cluster. If it is the local default kubernetes, use `https://kubernetes.default.svc`. Unless otherwise, use the valid path.

- `dest-namespace` : the application deploys on the cluster under this namespace. It is recommended to create a namespace for each application or environment.

## **Sync Cluster and Application**

At this step, you configured an application on the ArgoCD. But, the cluster has not been updated yet. So, the initial status will be `OutOfSync`. You can see the application description and details using the following command:

```

argocd app get <application-name>

```

Run the following command to start the sync:

```

argocd app sync <application-name>

```

## **Delete the cluster if you want to cleanup everything**

```

gcloud container clusters delete <CLUSTER_NAME> --zone <ZONE>

```

```

```
