## **Argo Workflows**

In this document, we demonstrate how to install Argo Workflows on GKE. Then, we will show some use cases.

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

### Access the Argo Workflows UI

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

**Ingerss**
Follow the instructions in this thread: [Ingress](https://argoproj.github.io/argo-workflows/argo-server/#ingress)

### **Understand Workflow Manifests**

This simple example demonstrate the template of the manifests for workflow. Attention to the following:

- `kind: Workflow`
- It's better to use `generateName` instead of `name`. Because you specify and prefix, and each workflow will have a unique generated name.

### Sample 1:

```
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: basic-wf-
  labels:
    workflows.argoproj.io/archive-strategy: "false"
spec:
  entrypoint: demo # the name of a template to start running. Templates are listed in `templates` section.
  serviceAccountName: workflow
  templates:
  - name: demo
    container:
      image: alpine:latest
      command: [ls]
      args: ["-l"]
```

In `templates` section, you can add as many as templates you would like. But, if you follow the above syntax, they will run sequentially and step by step. For more advanced workflow see the following sample:

### Sample 2:

```
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: basic-wf-
  labels:
    workflows.argoproj.io/archive-strategy: "false"
spec:
  entrypoint: hello
  serviceAccountName: workflow
  templates:
  - name: hello
    steps:
    - - name: ls
        template: template-ls
    - - name: sleep-a
        template: template-sleep
      - name: sleep-b
        template: template-sleep
    - - name: delay
        template: template-delay
    - - name: sleep
        template: template-sleep
  - name: template-ls
    container:
      image: alpine
      command: [ls]
      args: ["-l"]
  - name: template-sleep
    script:
      image: alpine
      command: [sleep]
      args: ["10"]
  - name: template-delay
    suspend:
      duration: "600s"

```

In this example, we are defining

### **DAG**

DAGs help to specify the tasks in any order and specify the dependencies. So, the tasks could run in parallel unless there is a dependency specified for the task.

```
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: dag-
  labels:
    workflows.argoproj.io/archive-strategy: "false"
spec:
  entrypoint: full
  serviceAccountName: workflow
  volumes:
  - name: kaniko-secret
    secret:
      secretName: regcred
      items:
        - key: .dockerconfigjson
          path: config.json
  templates:
  - name: full
    dag:
      tasks:
      - name: task-a
        template: my-task
        arguments:
          parameters:
          - name: message
            value: This is task-a
      - name: task-b
        template: my-task
        arguments:
          parameters:
          - name: message
            value: This is task-b
      - name: task-c
        template: my-task
        arguments:
          parameters:
          - name: message
            value: This is task-c
      - name: task-d
        template: my-task
        arguments:
          parameters:
          - name: message
            value: This is task-d
        dependencies:
        - task-a
      - name: task-e
        template: my-task
        arguments:
          parameters:
          - name: message
            value: This is task-e
        dependencies:
        - task-a
      - name: task-f
        template: my-task
        arguments:
          parameters:
          - name: message
            value: This is task-f
        dependencies:
        - task-a
        - task-e
      - name: task-g
        template: my-task
        arguments:
          parameters:
          - name: message
            value: This is task-g
  - name: my-task
    inputs:
      parameters:
      - name: message
    container:
      image: alpine
      command: [echo]
      args:
      - "{{inputs.parameters.message}}"

```
