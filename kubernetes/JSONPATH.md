# JSONPATH Documentation

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

Example:
```json
{
  "name": "myapp",
  "replicas": 3,
  "spec": {
    "containers": [
      { "name": "app", "image": "nginx" }
    ]
  }
}
```

A JSONPath expression might look like:
```text
$.spec.containers[0].image
```

This would return:
```text
nginx
```

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

JSONPath is useful when working with structured data, although Kubernetes usually uses YAML manifests instead of raw JSON for most configuration files.
