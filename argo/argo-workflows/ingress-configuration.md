# Ingress on GKE

To access Workflows UI, we need to expose the UI to a public domain address. Pay attention that Argo Workflows application, uses an environment variable to decide on its serving root path.

## **Step 1**

If you want to have argo workflows in and address like this: `https://example.com/` So, you don't need to change the default installation manifest. But, if you would like to have argo on another path like `https://example.com/argo/`, so you need to update the installation manifest. Add the following environment variable and value to the `deployment/argo-server` under the `container`:

```yaml
env:
  - name: BASE_HREF
    value: /argo/
```

I already added this environment variable to the [install.yaml](./manifests/install.yaml).

## **Step 2**

In this step, we will setup ingress on GKE. For ingress, we need to have an `ingress controller` and `ingress resource`s. Ingress Controller is the entry point of the cluster and this controller receives external requests. In order to route the traffic to the right service, Ingress Controller uses Ingress Resources (`Kind: Ingress`) to match the rules and find the right service for the incoming request.

Google Kubernetes Engine comes with a default ingress which is called `gce` and it already exists on the cluster. So, we won't need to install an ingress controller anymore. However, we will be using `nginx.ingress`. So, we need to install the `ingress controller` and make sure it is ready to use.

## **Install Nginx.Ingress**

Open the cloud shell on google cloud platform and use `helm` to install `nginx.ingress`:

Add the `nginx-stable` Helm repository:

```sh
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
```

Deploy `nginx-ingress` using Helm:

```sh
helm install nginx-ingress ingress-nginx/ingress-nginx
```

Verify if the nginx is ready. You should wait until you see an external IP being attached to the nginx ingress service:

```sh
kubectl get deployment nginx-ingress-ingress-nginx-controller
kubectl get service nginx-ingress-ingress-nginx-controller
```

Use the following to watch the service until you see the external IP:

```sh
watch kubectl get service nginx-ingress-ingress-nginx-controller
```

Export the external IP for later use and verify the environment variable value:

```sh
export NGINX_INGRESS_IP=$(kubectl get service nginx-ingress-ingress-nginx-controller -ojson | jq -r '.status.loadBalancer.ingress[].ip')

echo $NGINX_INGRESS_IP
```

**Note:** If you already have a domain, you need to configure the DNS and map your domain to the External IP address that NGINX ingress has received. Then, wait for a while to make sure the DNS is updated and replicated to DNS servers, then go for the next step.

## **Configure Ingress Resource to use NGINX Ingress**

An Ingress Resource object is a collection of L7 rules for routing inbound traffic to Kubernetes Services. Multiple rules can be defined in one Ingress Resource, or they can be split up into multiple Ingress Resource manifests. The Ingress Resource also determines which controller to use to serve traffic. This can be set with an annotation, `kubernetes.io/ingress.class`, in the metadata section of the Ingress Resource. For the NGINX controller, use the value `nginx`:

```yaml
annotations: kubernetes.io/ingress.class: nginx
```

Here is the NGINX ingress resource that we will be using. Note that we want to access argo on a path other than the root domain like `example.com/argo/`:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: argo-server
  namespace: argo
  annotations:
    kubernetes.io/ingress.class: nginx
    # nginx.ingress.kubernetes.io/from-to-www-redirect: "true"
    # nginx.ingress.kubernetes.io/ssl-redirect: "true"
    # nginx.ingress.kubernetes.io/preserve-trailing-slash: "true"
    nginx.ingress.kubernetes.io/backend-protocol: https
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  rules:
    - host: "{{domain.com}}"
      http:
        paths:
          - path: /argo(/|$)(.*)
            pathType: Prefix
            backend:
              service:
                name: argo-server
                port:
                  number: 2746
```
