# **ServiceAccount**

Workflows need sufficient access to do their jobs and it is dependant to the job they need to do. The permissions are assigned and applied using `ServiceAccount`. So, you need to create a service account on the cluster and use this service account while defining the workflow. If no service account is specified in the workflow, then the `default` service account will be used. You can specify the service account to be used in workflow like the following example:

```
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: basic-wf-
spec:
  entrypoint: hello
  serviceAccountName: workflow
```
