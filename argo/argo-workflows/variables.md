# **Variables**

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

### Simple:

```
args: [ "{{ inputs.parameters.message }}" ]
```

### Expression:

These tag will be evaluated as expression and the result is being used. The expression syntax can be learnt from here: [Expression Syntax](https://github.com/antonmedv/expr/blob/master/docs/Language-Definition.md)

```
args: [ "{{=asInt(lastRetry.exitCode) >= 2}}" ]
```

### Embeded Functions

There are some core functions in argo workflow to be used:

```
asInt(inputs.parameters['my-int-param'])

asFloat(inputs.parameters['my-float-param'])

string(1)

# Convert to JSON
toJson([1, 2])

# Extract data from JSON
jsonpath(inputs.parameters.json, '$.some.path')
```

In each template type there different variables available. Some important variable are listed bellow:

### **All Templates**

| Variable                   | Description                                         |
| -------------------------- | --------------------------------------------------- |
| `inputs.parameters.<NAME>` | Input parameter to a template                       |
| `inputs.parameters`        | All input parameters to a template as a JSON string |

### **DAG Templates**

| Variable                                     | Description                                                                                                                                                                  |
| -------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `tasks.<TASKNAME>.ip`                        | IP address of a previous daemon container task                                                                                                                               |
| `tasks.<TASKNAME>.exitCode`                  | Exit code of any previous script or container task                                                                                                                           |
| `tasks.<TASKNAME>.outputs.result`            | Output result of any previous container or script task                                                                                                                       |
| `tasks.<TASKNAME>.outputs.parameters`        | When the previous task uses 'withItems' or 'withParams', this contains a JSON array of the output parameter maps of each invocation                                          |
| `tasks.<TASKNAME>.outputs.parameters.<NAME>` | Output parameter of any previous task. When the previous task uses 'withItems' or 'withParams', this contains a JSON array of the output parameter values of each invocation |

### **Example: Set Output Parameter and Pass to other Template**

```
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: output-parameter-
spec:
  entrypoint: output-parameter
  templates:
  - name: output-parameter
    steps:
    - - name: generate-parameter
        template: whalesay
    - - name: consume-parameter
        template: print-message
        arguments:
          parameters:
          # Pass the hello-param output from the generate-parameter step as the message input to print-message
          - name: message
            value: "{{steps.generate-parameter.outputs.parameters.hello-param}}"

  - name: whalesay
    container:
      image: docker/whalesay:latest
      command: [sh, -c]
      args: ["echo -n hello world > /tmp/hello_world.txt"]  # generate the content of hello_world.txt
    outputs:
      parameters:
      - name: hello-param		# name of output parameter
        valueFrom:
          path: /tmp/hello_world.txt	# set the value of hello-param to the contents of this hello-world.txt

  - name: print-message
    inputs:
      parameters:
      - name: message
    container:
      image: docker/whalesay:latest
      command: [cowsay]
      args: ["{{inputs.parameters.message}}"]
```

### **Example: DAG and parameters**

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
