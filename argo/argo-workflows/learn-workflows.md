### **Workflow Syntax**

Everything in argo workflow is a template. So, we should learn how to define workflows using templates. Like other kubernetes configuration files, argo workflow structure starts with such syntax:

```
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: hello-world-  # Generate a name for this workflow with this prefix
spec:
```

Then, we define the workflow in the `Workflow.spec` field. The core structure of a Workflow spec is a list of templates and an `entrypoint`. The `entrypoint` defines which template should be executed first.

The follwoing example demonstrates how to define a workflow with a single template:

- `kind: Workflow`
- It's better to use `generateName` instead of `name`. Because you specify and prefix, and each workflow will have a unique generated name.

### Sample 1:

```
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: basic-wf-
spec:
  entrypoint: demo
  templates:
  - name: demo
    container:
      image: alpine:latest
      command: [ls]
      args: ["-l"]
```

Well, as we said each workflow is a list of templates. Each template has a `name` which can be referenced by and a template type. There are 6 template types and they are two categories:

| Category            | Template Type |
| ------------------- | ------------- |
| Template Definition | `container`   |
| Template Definition | `script`      |
| Template Definition | `resource`    |
| Template Definition | `suspend`     |
| Template Ivocator   | `steps`       |
| Template Ivocator   | `dag`         |

### **CONTAINER**

This template schedules a container. The spec of this template is the same as the [K8s container spec](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.19/#container-v1-core). Example:

```
- name: demo
    container:
      image: alpine:latest
      command: [ls]
      args: ["-l"]
```

### **SCRIPT**

You can use this template to execute a script in a container. The result of the script is exported into an `Argo variable`. The variable that will be used would be either `{{tasks.<NAME>.outputs.result}}` or `{{steps.<NAME>.outputs.result}}`, depending how it was called. Example:

```
  - name: gen-random-int
    script:
      image: python:alpine3.6
      command: [python]
      source: |
        import random
        i = random.randint(1, 100)
        print(i)
```

### **RESOURCE**

You can use this template to perform operations on cluster resources. These operations could be such as get, create, apply, delete, replace, or patch.

This example creates a `ConfigMap` resource on the cluster:

```
  - name: k8s-owner-reference
    resource:
      action: create
      manifest: |
        apiVersion: v1
        kind: ConfigMap
        metadata:
          generateName: owned-eg-
        data:
          some: value
```

### **SUSPEND**

This template suspends the execution of the workflow.

```
  - name: delay
    suspend:
      duration: "20s"
```

### **STEPS**

This template allows to define tasks in series of steps. The structure of the template is a "list of lists". Outer lists will run sequentially and inner lists will run in parallel.

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

Also, it is possible to use condition cluases to control the execution of steps based on some conditions. Here is an example:

```
# The coinflip example combines the use of a script result,
# along with conditionals, to take a dynamic path in the
# workflow. In this example, depending on the result of the
# first step, 'flip-coin', the template will either run the
# 'heads' step or the 'tails' step.
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: coinflip-
  annotations:
    workflows.argoproj.io/description: |
      This is an example of coin flip defined as a sequence of conditional steps.
      You can also run it in Python: https://couler-proj.github.io/couler/examples/#coin-flip
spec:
  entrypoint: coinflip
  templates:
  - name: coinflip
    steps:
    - - name: flip-coin
        template: flip-coin
    - - name: heads
        template: heads
        when: "{{steps.flip-coin.outputs.result}} == heads"
      - name: tails
        template: tails
        when: "{{steps.flip-coin.outputs.result}} == tails"

  - name: flip-coin
    script:
      image: python:alpine3.6
      command: [python]
      source: |
        import random
        result = "heads" if random.randint(0,1) == 0 else "tails"
        print(result)

  - name: heads
    container:
      image: alpine:3.6
      command: [sh, -c]
      args: ["echo \"it was heads\""]

  - name: tails
    container:
      image: alpine:3.6
      command: [sh, -c]
      args: ["echo \"it was tails\""]
```

### **DAG**

DAGs help to specify the tasks in any order and specify the dependencies. So, the tasks could run in parallel unless there is a dependency specified for the task. Also, it is not required to define the tasks in the order that you expect it to run. You just need to specify the dependencies. Actually, you define a graph of tasks.

```
  - name: diamond
    dag:
      tasks:
      - name: A
        template: echo
      - name: B
        dependencies: [A]
        template: echo
      - name: C
        dependencies: [A]
        template: echo
      - name: D
        dependencies: [B, C]
        template: echo
```

## **Variables**

Variables are enclosed in `{{}}`:

```
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: hello-world-parameters-
spec:
  entrypoint: whalesay
  arguments:
    parameters:
      - name: message
        value: hello world
  templates:
    - name: whalesay
      inputs:
        parameters:
          - name: message
      container:
        image: docker/whalesay
        command: [ cowsay ]
        args: [ "{{inputs.parameters.message}}" ]
```

There are two type of variable tags:

- **simple** `{{workflow.name}}`
- **expression** `{{=workflow.name}}`

```
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: dag-
spec:
  entrypoint: full
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

[Argo Workflow Examples](https://github.com/argoproj/argo-workflows/blob/master/examples/README.md)
