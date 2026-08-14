# JSONPATH Documentation

## Table of Contents

- [JSONPATH Documentation](#jsonpath-documentation)
  - [Table of Contents](#table-of-contents)
  - [YAML](#yaml)
    - [What is YAML?](#what-is-yaml)
    - [Why YAML is important](#why-yaml-is-important)
    - [YAML vs JSON vs XML](#yaml-vs-json-vs-xml)
      - [YAML](#yaml-1)
      - [JSON](#json)
      - [XML](#xml)
    - [Key differences](#key-differences)
    - [Use case in Kubernetes](#use-case-in-kubernetes)
    - [Why YAML is preferred in Kubernetes](#why-yaml-is-preferred-in-kubernetes)
    - [YAML fundamentals: key-value pairs, list, and dictionary](#yaml-fundamentals-key-value-pairs-list-and-dictionary)
      - [1. Key-value pairs](#1-key-value-pairs)
      - [2. Dictionary](#2-dictionary)
      - [3. List](#3-list)
    - [Advanced YAML: combining key-value pairs, dictionaries, and lists](#advanced-yaml-combining-key-value-pairs-dictionaries-and-lists)
      - [Difficulty: Basic](#difficulty-basic)
      - [Difficulty: Intermediate](#difficulty-intermediate)
      - [Difficulty: Advanced](#difficulty-advanced)
    - [Practical note: why this matters in Kubernetes](#practical-note-why-this-matters-in-kubernetes)
    - [Summary](#summary)
  - [JSONPath](#jsonpath)
    - [How to use JSONPath with Kubernetes](#how-to-use-jsonpath-with-kubernetes)
    - [1. Identify the kubectl command](#1-identify-the-kubectl-command)
    - [2. Familiarize the JSON output](#2-familiarize-the-json-output)
    - [3. Form the JSONPath query](#3-form-the-jsonpath-query)
    - [4. Use the JSON PATH query with kubectl command](#4-use-the-json-path-query-with-kubectl-command)
    - [JSONPath example with Kubernetes](#jsonpath-example-with-kubernetes)
    - [Loops in kubectl JSONPath](#loops-in-kubectl-jsonpath)
    - [Custom columns](#custom-columns)
    - [Wildcard when using criteria or filtering](#wildcard-when-using-criteria-or-filtering)

## YAML

### What is YAML?
YAML stands for "YAML Ain't Markup Language." It is a human-readable data serialization format commonly used for configuration files, infrastructure definitions, and application manifests.

YAML is designed to be easy for people to read and write, while still being structured enough for machines to parse reliably. It uses indentation instead of braces or tags, which makes it cleaner and more readable than JSON or XML for configuration files.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
    - name: myapp-container
      image: nginx
```

### Why YAML is important
YAML is widely used in DevOps and cloud environments because it is:
- Easy to read and edit
- Minimal and clean in syntax
- Good for configuration management
- Commonly used in Kubernetes manifests

### YAML vs JSON vs XML

#### YAML
- Human-friendly
- Uses indentation and key-value pairs
- Easier to read than JSON for configuration files
- Common in Kubernetes, Ansible, Docker Compose, and CI/CD

Example:
```yaml
name: myapp
replicas: 3
image: nginx
```

#### JSON
- Strict and machine-friendly
- Uses braces and quoted keys
- Good for APIs and data exchange
- Less readable for humans than YAML

Example:
```json
{
  "name": "myapp",
  "replicas": 3,
  "image": "nginx"
}
```

#### XML
- Very verbose
- Uses opening and closing tags
- Good for structured documents and enterprise data interchange
- More difficult to read and write than YAML

Example:
```xml
<application>
  <name>myapp</name>
  <replicas>3</replicas>
  <image>nginx</image>
</application>
```

### Key differences
- YAML is simpler and more readable for humans.
- JSON is stricter and more suitable for APIs and machine parsing.
- XML is more verbose and tag-based, which makes it heavier than YAML.

### Use case in Kubernetes
In Kubernetes, YAML is the standard format used to define resources such as Pods, Deployments, Services, ReplicaSets, and Namespaces. These files are sometimes called manifests.

This is directly related to the examples in [kubernetes/KUBERNETES.md](kubernetes/KUBERNETES.md), where Kubernetes objects are defined in YAML.

Example from Kubernetes:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp-container
          image: nginx
```

This YAML tells Kubernetes:
- which API version to use
- the kind of object to create
- the metadata such as the deployment name
- how many replicas should exist
- which labels identify the pods
- which container image to run

### Why YAML is preferred in Kubernetes
Kubernetes YAML manifests are easier to understand and maintain than JSON or XML because:
- they are cleaner to read
- they support comments and indentation
- they are easier for teams to version-control and review
- they are the standard for declarative infrastructure

### YAML fundamentals: key-value pairs, list, and dictionary
YAML is built from a few core structures:
- **Key-value pair**: a field and its value
- **Dictionary**: a collection of key-value pairs
- **List**: a sequence of values

#### 1. Key-value pairs
A key-value pair is the simplest YAML structure. The key is on the left and the value is on the right.

```yaml
name: myapp
replicas: 3
image: nginx
```

This is equivalent to:
```text
name = myapp
replicas = 3
image = nginx
```

**Example 1 - Basic application config**
```yaml
app: frontend
port: 80
```

**Example 2 - Environment variable style**
```yaml
ENV: production
DEBUG: false
```

**Example 3 - Metadata**
```yaml
project: payments-api
owner: platform-team
```

#### 2. Dictionary
A dictionary is a mapping of keys to values. In YAML, a dictionary is usually represented as a nested block where each key has its own value.

```yaml
metadata:
  name: myapp
  labels:
    app: web
    tier: frontend
```

This means:
- `metadata` is a dictionary
- `name` and `labels` are keys inside it
- `labels` is itself a dictionary

**Example 1 - Service metadata**
```yaml
service:
  name: myapp
  type: ClusterIP
```

**Example 2 - Deployment metadata**
```yaml
metadata:
  name: myapp-deploy
  namespace: dev
```

**Example 3 - Nested dictionary**
```yaml
spec:
  selector:
    matchLabels:
      app: myapp
      tier: frontend
```

#### 3. List
A list is an ordered set of values. In YAML, lists are usually written with a dash (`-`).

```yaml
containers:
  - name: app
    image: nginx
  - name: sidecar
    image: busybox
```

This means:
- `containers` is a list
- each item in the list is an object with its own keys

**Example 1 - Simple list**
```yaml
ports:
  - 80
  - 443
```

**Example 2 - List of names**
```yaml
servers:
  - web-01
  - web-02
  - web-03
```

**Example 3 - List of objects**
```yaml
volumes:
  - name: app-data
    mountPath: /var/lib/app
  - name: config
    mountPath: /etc/config
```

### Advanced YAML: combining key-value pairs, dictionaries, and lists
YAML becomes powerful when these structures are combined. This is the style used in Kubernetes manifests.

#### Difficulty: Basic
**Example 1**
```yaml
app:
  name: myapp
  replicas: 3
  image: nginx
```

**Example 2**
```yaml
env:
  production: true
  debug: false
```

**Example 3**
```yaml
ports:
  - 80
  - 8080
```

#### Difficulty: Intermediate
**Example 1 - Pod definition**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
  labels:
    app: myapp
    tier: frontend
spec:
  containers:
    - name: app-container
      image: nginx
      ports:
        - containerPort: 80
```

**Example 2 - Deployment structure**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp-container
          image: nginx
          env:
            - name: ENVIRONMENT
              value: production
```

**Example 3 - Service with selector and ports**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

#### Difficulty: Advanced
**Example 1 - Complex nested configuration**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payments-deployment
  labels:
    app: payments
    environment: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: payments
  template:
    metadata:
      labels:
        app: payments
    spec:
      containers:
        - name: payments-api
          image: myregistry/payments:1.2.0
          ports:
            - containerPort: 8080
          env:
            - name: DB_HOST
              value: mysql.internal
            - name: DB_PORT
              value: "3306"
          resources:
            requests:
              cpu: "250m"
              memory: "256Mi"
            limits:
              cpu: "500m"
              memory: "512Mi"
```

**Example 2 - Multi-service deployment config**
```yaml
services:
  frontend:
    type: LoadBalancer
    ports:
      - port: 80
        targetPort: 8080
  backend:
    type: ClusterIP
    ports:
      - port: 8081
        targetPort: 8081
```

**Example 3 - Full Kubernetes-style config with nested dictionaries and lists**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: team-a
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  namespace: team-a
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
        - name: web-app-container
          image: nginx:latest
          ports:
            - containerPort: 80
          volumeMounts:
            - name: config-volume
              mountPath: /etc/config
      volumes:
        - name: config-volume
          configMap:
            name: web-app-config
```

### Practical note: why this matters in Kubernetes
In Kubernetes, the resources you define are almost always a combination of:
- dictionaries for metadata and spec
- lists for containers, ports, env variables, and volumes
- key-value pairs for labels, annotations, and config values

This is why YAML is such a strong fit for Kubernetes: it allows you to represent complex infrastructure declaratively in a human-readable way.

### Summary
YAML is made from simple building blocks:
- **Key-value pairs** store single values
- **Dictionaries** store grouped values
- **Lists** store ordered collections

When combined, they create complex but readable configuration structures such as Kubernetes Deployment, Service, and Pod manifests.

---

## JSONPath
JSONPath is a way to query values inside JSON-like data structures. It helps locate specific fields or nested values in a document.

### How to use JSONPath with Kubernetes
When working with Kubernetes, the key idea is simple:

1. Identify the `kubectl` command you need.
2. Request JSON output using `-o json` so the result is structured and queryable.
3. Familiarize yourself with the JSON object returned by Kubernetes.
4. Form the JSONPath expression that matches the field you want.
5. Use the JSONPath query in the `kubectl` command.

This workflow is especially useful when you want to extract only the fields you need without printing the entire YAML manifest.

### 1. Identify the kubectl command
Before writing a JSONPath query, decide which resource you want to inspect.

Examples:
```bash
kubectl get pods
kubectl get deployments
kubectl get services
kubectl get nodes
```

If you need structured data for JSONPath, add `-o json`.

### 2. Familiarize the JSON output
Kubernetes returns a JSON object where resources are typically nested under `items` for lists, and each object contains metadata, spec, and status.

Example `kubectl get pods -o json` output:
```json
{
  "apiVersion": "v1",
  "items": [
    {
      "metadata": {
        "name": "api-pod",
        "namespace": "default",
        "labels": {
          "app": "api"
        }
      },
      "status": {
        "phase": "Running"
      },
      "spec": {
        "containers": [
          {
            "name": "api-container",
            "image": "nginx:latest"
          }
        ]
      }
    }
  ],
  "kind": "List"
}
```

This is the output you need before building a JSONPath expression.

### 3. Form the JSONPath query
Once you understand the JSON structure, you can create a JSONPath expression.

Examples:
```text
$.items[*].metadata.name
```
This returns all pod names.

```text
$.items[*].status.phase
```
This returns each pod status.

```text
$.items[*].spec.containers[*].image
```
This returns all container images.

For nested values, JSONPath follows the same structure as the JSON object.

### 4. Use the JSON PATH query with kubectl command
Use `kubectl get ... -o jsonpath='...'` with the JSONPath expression.

Example: list pod names.
```bash
kubectl get pods -o jsonpath='{.items[*].metadata.name}'
```

Example output:
```text
api-pod web-pod worker-pod
```

Example: list pod names one per line.
```bash
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\n"}{end}'
```

Example output:
```text
api-pod
web-pod
worker-pod
```

This is a typical pattern when you want a cleaner, machine-friendly output.

### JSONPath example with Kubernetes
Example JSON object:
```json
{
  "items": [
    { "name": "api", "status": "Running" },
    { "name": "web", "status": "Pending" },
    { "name": "worker", "status": "Running" }
  ]
}
```

Select all names:
```text
$.items[*].name
```

This returns:
```text
api web worker
```

Filter only the running items:
```text
$.items[?(@.status=="Running")].name
```

This returns:
```text
api worker
```

### Loops in kubectl JSONPath
Loops let you iterate through a list and print multiple fields in a single line or one per line.

Example: print each pod name and phase.
```bash
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{" - "}{.status.phase}{"\n"}{end}'
```

Example output:
```text
api-pod - Running
web-pod - Pending
worker-pod - Running
```

Example: print namespace and pod name.
```bash
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.namespace}{" "}{.metadata.name}{"\n"}{end}'
```

### Custom columns
Instead of JSONPath, Kubernetes also supports custom columns for a more readable table.

Example:
```bash
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase,NAMESPACE:.metadata.namespace
```

Example output:
```text
NAME       STATUS    NAMESPACE
a pi-pod   Running   default
web-pod   Pending   default
```

A custom-columns command is often easier to read than JSONPath when you want a table, but JSONPath is more powerful when you need to extract precise nested values.

### Wildcard when using criteria or filtering
The `*` wildcard is used to match all items in a list when you want to select everything, or when you want to filter a collection by a condition.

Example JSON:
```json
{
  "items": [
    { "name": "api", "status": "Running" },
    { "name": "web", "status": "Pending" },
    { "name": "worker", "status": "Running" }
  ]
}
```

Select all item names:
```text
$.items[*].name
```

This returns:
```text
api web worker
```

Filter only the running items:
```text
$.items[?(@.status=="Running")].name
```

This returns:
```text
api worker
```

The `*` helps you select all values in a collection, while the filter `?()` narrows the result based on a condition.

JSONPath is useful when working with structured data, especially when Kubernetes output is formatted as JSON. In practice, you usually pair it with `kubectl get ... -o json` or `kubectl get ... -o jsonpath='...'` to extract exactly the information you need.
