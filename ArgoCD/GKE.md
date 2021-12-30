## **ArgoCD on Google Kubernetes Engine**

1. Installed gcloud cli on local machine
2. Create google project
3. Activate the google project
4. Enable Kubernetes Engine API on Google account
5. Set default project and region for gcloud cli

```
gcloud init
```

Get projects list:

```
gcloud projects list --sort-by=projectId
```

Set default project:

```
gcloud config set project <PROJECT_ID>
```

Create a cluster on GCloud

```
gcloud container clusters create <CLUSTER_NAME> --num-nodes=1
```

I used `arax-cluster` as cluster name. You will see such result:

```
NAME               LOCATION    MASTER_VERSION   MASTER_IP     MACHINE_TYPE  NODE_VERSION     NUM_NODES  STATUS
arax-cluster  us-east1-b  1.21.5-gke.1302  --.---.--.--  e2-medium     1.21.5-gke.1302  1          RUNNING
```

Then, configure the `kubectl` to access the created cluster on google.

```
gcloud container clusters get-credentials <CLUSTER_NAME>
```

The result is:

```
Fetching cluster endpoint and auth data.
kubeconfig entry generated for <CLUSTER_NAME>.
```

Now, the `kubectl` is pointing to the gke cluster.
