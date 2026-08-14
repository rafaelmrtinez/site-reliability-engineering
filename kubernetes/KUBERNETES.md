# Kubernetes Fundamentals and Core Objects

## Table of Contents

- [Kubernetes Fundamentals and Core Objects](#kubernetes-fundamentals-and-core-objects)
  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
    - [Key terms](#key-terms)
  - [Containerd vs Docker](#containerd-vs-docker)
  - [Nerdctl](#nerdctl)
  - [Crictl](#crictl)
  - [Pods](#pods)
    - [Pod characteristics](#pod-characteristics)
  - [kubectl Basics](#kubectl-basics)
  - [Commands and Arguments](#commands-and-arguments)
  - [Environment Variables](#environment-variables)
    - [Basic env setting](#basic-env-setting)
    - [Using ConfigMap](#using-configmap)
    - [Using Secrets](#using-secrets)
    - [How to create ConfigMap](#how-to-create-configmap)
      - [Imperative way](#imperative-way)
      - [Declarative way](#declarative-way)
    - [Create ConfigMap and inject to Pod](#create-configmap-and-inject-to-pod)
    - [ENV, Single ENV, and Volume way](#env-single-env-and-volume-way)
      - [1. Direct env list in the Pod](#1-direct-env-list-in-the-pod)
      - [2. Single environment variable from ConfigMap or Secret](#2-single-environment-variable-from-configmap-or-secret)
      - [3. Volume way](#3-volume-way)
  - [Secrets](#secrets)
    - [Imperative vs Declarative](#imperative-vs-declarative)
    - [Declarative with plain text vs hash format](#declarative-with-plain-text-vs-hash-format)
    - [How to convert data to encoded base64](#how-to-convert-data-to-encoded-base64)
    - [How to decode base64 value to original value](#how-to-decode-base64-value-to-original-value)
    - [Injecting Secrets in Pod](#injecting-secrets-in-pod)
      - [1. Inject Secret using ENV](#1-inject-secret-using-env)
      - [2. Inject single ENV from Secret](#2-inject-single-env-from-secret)
      - [3. Inject all Secret keys using envFrom](#3-inject-all-secret-keys-using-envfrom)
      - [4. Inject Secret using Volume](#4-inject-secret-using-volume)
  - [YAML Base Configuration in Kubernetes](#yaml-base-configuration-in-kubernetes)
  - [Replication Controller](#replication-controller)
  - [ReplicaSet](#replicaset)
  - [ReplicaController vs ReplicaSet](#replicacontroller-vs-replicaset)
    - [ReplicaController](#replicacontroller)
    - [ReplicaSet](#replicaset-1)
  - [Rolling Updates](#rolling-updates)
  - [Deployment (Modern Recommendation)](#deployment-modern-recommendation)
  - [Namespaces](#namespaces)
  - [Encrypting Secret Data at Rest](#encrypting-secret-data-at-rest)
    - [How encryption works in Kubernetes](#how-encryption-works-in-kubernetes)
    - [What is etcdctl?](#what-is-etcdctl)
    - [Install `etcdctl` on Linux](#install-etcdctl-on-linux)
      - [Debian / Ubuntu](#debian--ubuntu)
      - [Ubuntu / Debian (alternative if package name differs)](#ubuntu--debian-alternative-if-package-name-differs)
      - [RHEL / CentOS / Rocky / AlmaLinux](#rhel--centos--rocky--almalinux)
      - [Fedora](#fedora)
      - [OpenSUSE / SLES](#opensuse--sles)
      - [Arch Linux](#arch-linux)
      - [Alpine Linux](#alpine-linux)
    - [`kubectl get pods -n kube-system`](#kubectl-get-pods--n-kube-system)
  - [Docker Security](#docker-security)
    - [Running containers with elevated permissions](#running-containers-with-elevated-permissions)
    - [Linux capabilities and root user behavior](#linux-capabilities-and-root-user-behavior)
    - [`CAP_SYS_ADMIN` and similar capabilities](#cap_sys_admin-and-similar-capabilities)
    - [Where capabilities are managed](#where-capabilities-are-managed)
    - [Best practices for Docker security](#best-practices-for-docker-security)
    - [Practical secure example](#practical-secure-example)
  - [Final Notes](#final-notes)
  - [Kubernetes Pod Security Context](#kubernetes-pod-security-context)
    - [What is security context?](#what-is-security-context)
    - [What is `runAsUser`?](#what-is-runasuser)
    - [What are capabilities?](#what-are-capabilities)
  - [Services](#services)
    - [Service Type: ClusterIP](#service-type-clusterip)
    - [Service Type: NodePort](#service-type-nodeport)
    - [Service Type: LoadBalancer](#service-type-loadbalancer)
  - [kubectl Explain and API Resources](#kubectl-explain-and-api-resources)
  - [kubectl Imperative Commands](#kubectl-imperative-commands)

**Certified Kubernetes Application Developer**
- https://www.cncf.io/certification/ckad/
- https://www.cncf.io/certification/candidate-handbook
- https://docs.linuxfoundation.org/tc-docs/certification/tips-cka-and-ckad

## Overview
Kubernetes is a container orchestration platform used to run, scale, and manage containerized applications. It helps automate deployment, self-healing, scaling, and service discovery.

### Key terms
- **Node**: A machine that runs workloads. It can be physical or virtual.
- **Cluster**: A group of nodes working together.
- **Master Node**: Handles orchestration and control-plane functions.
- **API Server**: The front door for all Kubernetes communication.
- **etcd**: A distributed key-value store that keeps cluster state.
- **Scheduler**: Decides which node should run a workload.
- **Container Runtime**: Software that runs containers such as containerd or CRI-O.
- **Kubelet**: An agent running on each node that ensures containers are healthy.

**Summary**: Nodes run workloads, while the control plane manages the cluster and keeps it healthy.

```bash
# Basic kubectl checks
kubectl cluster-info
kubectl get nodes
kubectl get pods -A
```

---

## Containerd vs Docker
**Quick explanation**: Docker is a developer-focused tool, but Kubernetes typically uses a lower-level container runtime such as containerd.

- **Docker**: A container platform that includes the Docker Engine.
- **Containerd**: A lightweight runtime used to run containers efficiently.
- **Docker flow**: Docker Engine -> dockerd -> containerd -> runc

**Why it matters**: In Kubernetes, containerd is commonly used behind the scenes instead of Docker as the direct runtime.

```bash
# Containerd examples
ctr images ls
ctr images pull docker.io/library/nginx:latest
```

**Example flow**: A container image is pulled by the runtime, then the runtime starts the pod container on a selected node.

---

## Nerdctl
**Quick explanation**: Nerdctl is a tool that resembles Docker but is designed to work with containerd-compatible runtimes.

```bash
# Nerdctl examples
nerdctl images ls
nerdctl run --name myapp nginx
```

**Summary**: Nerdctl provides a Docker-like experience for containerd-based systems, especially in environments where Docker is not the primary runtime.

---

## Crictl
**Quick explanation**: Crictl is used to inspect and debug containers through the CRI-compatible runtime. It is not the main tool for creating workloads.

```bash
# Crictl examples
crictl ps -a
crictl images
crictl logs <container_id>
crictl exec -it <container_id> /bin/sh
```

**Summary**: Use crictl for troubleshooting and inspecting low-level runtime behavior.

---

## Pods
**Quick explanation**: A pod is the smallest unit in Kubernetes. It usually contains one container, but it can contain multiple containers when they are tightly related.

### Pod characteristics
- Smallest object in Kubernetes
- Usually one application per pod
- Can run on different nodes
- Has a one-to-one relationship with the main container
- Multiple containers in one pod are only used for tightly related helper processes

**Note**: Helper containers are rare and only used when one container depends on another in the same pod.

```yaml
# Pod example
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
    type: frontend
spec:
  containers:
    - name: myapp-container
      image: nginx
```

```bash
# Create and inspect the pod
kubectl create -f pod.yaml
kubectl get pods
kubectl describe pod myapp-pod
```

**Summary**: Pods are the basic runtime unit; most workloads are then managed by higher-level objects such as ReplicaSet or Deployment.

---

## kubectl Basics
**Quick explanation**: kubectl is the main command-line tool used to interact with a Kubernetes cluster.

```bash
# Basic commands
kubectl run nginx --image=nginx
kubectl get pods
kubectl cluster-info
```

**Summary**: These commands help you create a workload, list running resources, and confirm cluster connectivity.

---

## Commands and Arguments
**Quick explanation**: In Kubernetes, the `command` and `args` fields inside a container definition control how the container starts. They are the Kubernetes equivalent of how a Docker image defines its startup behavior.

**Important relationship to Docker**:
- `command` in Kubernetes maps most closely to Docker `ENTRYPOINT`
- `args` in Kubernetes maps most closely to Docker `CMD`
- If you pass extra arguments at runtime, Docker treats them as arguments to the `ENTRYPOINT`

**Pod YAML definition with container commands and arguments**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: command-demo
spec:
  containers:
    - name: app
      image: busybox
      command: ["/bin/sh", "-c"]
      args: ["echo Hello from Kubernetes; sleep 3600"]
```

**What this means**:
- The container starts with `/bin/sh -c`
- The command receives the string `echo Hello from Kubernetes; sleep 3600`
- This is effectively the same idea as a Docker image starting with a shell and then running a command

**Docker analogy**:
```dockerfile
FROM busybox
ENTRYPOINT ["/bin/sh", "-c"]
CMD ["echo Hello from Docker; sleep 3600"]
```

In this Docker image:
- `ENTRYPOINT` sets the main executable that always runs
- `CMD` sets the default argument(s) passed to that executable

**Kubernetes to Docker mapping**:
- `command: ["/bin/sh", "-c"]` → Docker `ENTRYPOINT ["/bin/sh", "-c"]`
- `args: ["echo Hello from Kubernetes; sleep 3600"]` → Docker `CMD ["echo Hello from Kubernetes; sleep 3600"]`

**Override behavior**:
- `command` replaces the container's default startup command
- `args` replaces the default command arguments
- This lets you override what a Docker image does without rebuilding the image

**Example override**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: override-demo
spec:
  containers:
    - name: app
      image: nginx
      command: ["nginx"]
      args: ["-g", "daemon off;"]
```

This starts NGINX explicitly, passing the runtime flags it needs.

**Key takeaway**:
- Kubernetes `command` is the startup executable you want to run.
- Kubernetes `args` is the parameter list given to that executable.
- In Docker terms, `command` aligns with `ENTRYPOINT`, and `args` aligns with `CMD`.

**Summary**: Understanding `command` and `args` is essential because Kubernetes does not just run a container image blindly. It can override the image's default startup behavior in a way that mirrors how Docker configures `ENTRYPOINT` and `CMD`.

---

## Environment Variables
**Quick explanation**: Environment variables are key-value pairs passed into a container at runtime. They are used to configure application behavior without baking values directly into the container image.

**Why environment variables matter**:
- Keep configuration separate from the application code
- Make it easy to change values between environments
- Support dev, test, and production settings
- Work well with Kubernetes ConfigMaps and Secrets

### Basic env setting
The simplest way is to define environment variables directly inside the pod manifest.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-basic
spec:
  containers:
    - name: app
      image: nginx
      env:
        - name: APP_ENV
          value: production
        - name: LOG_LEVEL
          value: info
```

**This is a single pod definition**: the environment variables are included directly in the manifest.

**Example with one variable**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: single-env
spec:
  containers:
    - name: app
      image: busybox
      env:
        - name: MESSAGE
          value: "Hello from Kubernetes"
```

**The `env` field is a list of key-value entries**. Each entry includes:
- `name`: the variable name
- `value`: the literal value assigned to that variable

### Using ConfigMap
A ConfigMap stores non-sensitive configuration data. It can be mounted as environment variables or as a volume.

**Basic ConfigMap YAML**:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: production
  LOG_LEVEL: debug
```

**Inject ConfigMap as environment variables in a Pod**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-from-configmap
spec:
  containers:
    - name: app
      image: nginx
      env:
        - name: APP_ENV
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_ENV
        - name: LOG_LEVEL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: LOG_LEVEL
```

**Inject all ConfigMap key/value pairs into a Pod**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-from-configmap-all
spec:
  containers:
    - name: app
      image: busybox
      envFrom:
        - configMapRef:
            name: app-config
```

**This is the declarative way**: the ConfigMap is defined separately and then referenced by the pod.

### Using Secrets
A Secret is used for sensitive data such as passwords, tokens, and keys.

**Create a Secret**:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
stringData:
  DB_PASSWORD: supersecret
```

**Inject Secret as environment variables**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-from-secret
spec:
  containers:
    - name: app
      image: nginx
      env:
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: DB_PASSWORD
```

**Inject all Secret keys into a Pod**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-from-secret-all
spec:
  containers:
    - name: app
      image: busybox
      envFrom:
        - secretRef:
            name: app-secret
```

**Tip**: Use ConfigMap for configuration values and Secret for sensitive values.

### How to create ConfigMap
There are two common ways to create a ConfigMap: imperative and declarative.

#### Imperative way
```bash
kubectl create configmap app-config --from-literal=APP_ENV=production --from-literal=LOG_LEVEL=debug
```

**Check it**:
```bash
kubectl get configmap app-config -o yaml
```

#### Declarative way
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: production
  LOG_LEVEL: debug
```

```bash
kubectl apply -f configmap.yaml
```

### Create ConfigMap and inject to Pod
**Example 1: ConfigMap and Pod in the same file**:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: production
  LOG_LEVEL: debug
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-configmap
spec:
  containers:
    - name: app
      image: nginx
      env:
        - name: APP_ENV
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_ENV
        - name: LOG_LEVEL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: LOG_LEVEL
```

**Example 2: ConfigMap defined separately**:
```yaml
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: production
  LOG_LEVEL: info
```

```yaml
# pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-configmap-separate
spec:
  containers:
    - name: app
      image: nginx
      envFrom:
        - configMapRef:
            name: app-config
```

```bash
kubectl apply -f configmap.yaml
kubectl apply -f pod.yaml
```

### ENV, Single ENV, and Volume way
There are multiple patterns for injecting configuration values.

#### 1. Direct env list in the Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: direct-env
spec:
  containers:
    - name: app
      image: nginx
      env:
        - name: MODE
          value: prod
        - name: PORT
          value: "8080"
```

#### 2. Single environment variable from ConfigMap or Secret
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: single-env-ref
spec:
  containers:
    - name: app
      image: nginx
      env:
        - name: APP_ENV
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_ENV
```

#### 3. Volume way
A ConfigMap can also be mounted as a file inside the container using a volume.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  app.properties: |
    APP_ENV=production
    LOG_LEVEL=debug
---
apiVersion: v1
kind: Pod
metadata:
  name: configmap-volume
spec:
  containers:
    - name: app
      image: nginx
      volumeMounts:
        - name: config-volume
          mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: app-config
```

Inside the container, the files appear in `/etc/config`.

**Summary**: Environment variables are the standard way to pass configuration to containers. Use literal `env` entries for simple values, ConfigMap for non-sensitive settings, and Secret for sensitive information. You can inject them directly as environment variables or mount them as files through a volume.

---

## Secrets
**Quick explanation**: A Secret is used to store sensitive data such as passwords, API keys, certificates, and tokens. Kubernetes keeps this data in a dedicated object and injects it into pods only when needed.

**Why Secrets matter**:
- Protect sensitive values from hardcoding in manifests
- Keep credentials separate from application code
- Support secure runtime configuration
- Work with both environment variables and mounted files

### Imperative vs Declarative
**Imperative creation**:
```bash
kubectl create secret generic app-secret --from-literal=DB_PASSWORD=supersecret
```

**Declarative creation**:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
stringData:
  DB_PASSWORD: supersecret
```

```bash
kubectl apply -f secret.yaml
```

**Difference**:
- Imperative is quick and useful for testing or one-off tasks.
- Declarative is preferred for production and version-controlled infrastructure.

### Declarative with plain text vs hash format
A Secret can be written in either a readable plain-text form or the encoded data form.

**Plain text form**:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
stringData:
  DB_PASSWORD: supersecret
```

**Encoded/base64 form**:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  DB_PASSWORD: c3VwZXJzZWNyZXQ=
```

The value `c3VwZXJzZWNyZXQ=` is the base64 encoding of `supersecret`.

**Important note**:
- Kubernetes stores Secret data in base64 form in the API object.
- `stringData` is simply a more readable way to write the secret in YAML.
- `data` is the encoded form used behind the scenes.

### How to convert data to encoded base64
On Linux/macOS:
```bash
echo -n 'supersecret' | base64
```

Output:
```bash
c3VwZXJzZWNyZXQ=
```

On Windows PowerShell:
```powershell
[Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes('supersecret'))
```

### How to decode base64 value to original value
On Linux/macOS:
```bash
echo -n 'c3VwZXJzZWNyZXQ=' | base64 --decode
```

Output:
```bash
supersecret
```

On Windows PowerShell:
```powershell
[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String('c3VwZXJzZWNyZXQ='))
```

**Important**: Base64 is not encryption. It is only an encoding method used to safely transport data inside Kubernetes API objects.

### Injecting Secrets in Pod
Secrets can be injected in several ways.

#### 1. Inject Secret using ENV
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
stringData:
  DB_PASSWORD: supersecret
---
apiVersion: v1
kind: Pod
metadata:
  name: secret-env
spec:
  containers:
    - name: app
      image: nginx
      env:
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: DB_PASSWORD
```

This makes the variable `DB_PASSWORD` available inside the container.

#### 2. Inject single ENV from Secret
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: single-secret-env
spec:
  containers:
    - name: app
      image: busybox
      env:
        - name: API_KEY
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: API_KEY
```

Use this when only one secret value is needed.

#### 3. Inject all Secret keys using envFrom
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-from
spec:
  containers:
    - name: app
      image: busybox
      envFrom:
        - secretRef:
            name: app-secret
```

This injects all keys from the Secret as environment variables.

#### 4. Inject Secret using Volume
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
stringData:
  DB_PASSWORD: supersecret
  API_KEY: abc123
---
apiVersion: v1
kind: Pod
metadata:
  name: secret-volume
spec:
  containers:
    - name: app
      image: nginx
      volumeMounts:
        - name: secret-volume
          mountPath: /etc/secrets
          readOnly: true
  volumes:
    - name: secret-volume
      secret:
        secretName: app-secret
```

Inside the container, the Secret values appear as files under `/etc/secrets`.

**Summary**: Secrets should be used for sensitive information. They can be created imperatively or declaratively, stored in readable `stringData` or encoded `data`, and then injected into pods as environment variables or mounted files.

---

## YAML Base Configuration in Kubernetes
**Quick explanation**: Kubernetes objects are commonly defined as YAML files. The YAML describes the object type, metadata, and the desired state of the workload.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
    type: frontend
spec:
  containers:
    - name: myapp-container
      image: nginx
```

```bash
# Create and inspect the pod from YAML
kubectl create -f pod.yaml
kubectl get pods
kubectl describe pod myapp-pod
```

**Summary**: This example shows the same application pattern we will reuse in the ReplicaController, ReplicaSet, and Deployment examples.

---

## Replication Controller
**Quick explanation**: A ReplicationController ensures that a specific number of identical pod replicas are always running.

- High availability
- Restarts failed pods automatically
- Maintains a target number of replicas
- Older object, mostly replaced by ReplicaSet and Deployment

**Sample ReplicaController configuration**
```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: myapp-rc
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: myapp
        type: frontend
    spec:
      containers:
        - name: myapp-container
          image: nginx
          ports:
            - containerPort: 80
```

```bash
# Create and inspect the controller
kubectl create -f rc.yaml
kubectl get rc
kubectl get pods
kubectl describe rc myapp-rc
```

**Create, update, and delete**
```bash
# Create a controller from a YAML file
kubectl create -f rc.yaml

# Apply or update the controller definition
kubectl apply -f rc.yaml

# Increase or decrease replica count
kubectl scale rc myapp-rc --replicas=5

# Delete the controller and all managed pods
kubectl delete rc myapp-rc
```

**Summary**: The controller makes sure the cluster keeps the desired number of pod copies running even if some fail.

**Note**: A ReplicationController does not explicitly declare a selector in the manifest. Instead, it relies on the labels in the pod template to manage the matching pods.

---

## ReplicaSet
**Quick explanation**: A ReplicaSet is the modern replacement for ReplicationController. It ensures that a specific number of matching pods are running and uses a selector to determine which pods belong to it.

- More modern than ReplicationController
- Maintains desired replicas
- Uses selectors for matching pods
- Common building block for Deployment

**Sample ReplicaSet configuration**
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      type: frontend
  template:
    metadata:
      labels:
        type: frontend
    spec:
      containers:
        - name: myapp-container
          image: nginx
          ports:
            - containerPort: 80
```

```bash
# Create and inspect the ReplicaSet
kubectl create -f rs.yaml
kubectl get rs
kubectl get pods
kubectl describe rs myapp-rs
```

**Create, scale, update, and delete**
```bash
# Create a ReplicaSet from YAML
kubectl create -f rs.yaml

# Apply updates to the ReplicaSet definition
kubectl apply -f rs.yaml

# Scale to a specific replica count
kubectl scale rs myapp-rs --replicas=5

# Scale up from the current value to a higher count
kubectl scale rs myapp-rs --replicas=10

# Scale down to a smaller count
kubectl scale rs myapp-rs --replicas=2

# Update the replica count directly in the YAML and re-apply
kubectl apply -f rs.yaml

# Delete the ReplicaSet and its managed pods
kubectl delete rs myapp-rs
```

**Summary**: The selector is required in ReplicaSet because it tells Kubernetes which pods belong to this set and must be kept at the desired replica count.

**Note**: The selector must match the labels in the template. In this example, both use `type: frontend`.

---

## ReplicaController vs ReplicaSet
**Quick explanation**: Both objects maintain a desired number of pod replicas, but ReplicaSet is newer and more flexible.

### ReplicaController
- Older object
- Less flexible selector model
- Does not explicitly declare a selector in the manifest
- Works well for simple replication scenarios

### ReplicaSet
- Modern object
- Must define a selector
- Supports `matchLabels` and more robust matching logic
- Commonly used as the base for Deployment

**Example comparison**
```bash
# ReplicationController example
kubectl create -f rc.yaml
kubectl scale rc myapp-rc --replicas=3
kubectl delete rc myapp-rc

# ReplicaSet example
kubectl create -f rs.yaml
kubectl scale rs myapp-rs --replicas=3
kubectl delete rs myapp-rs
```

**Summary**: ReplicaSet is preferred over ReplicationController because it is a newer and more capable object.

**Note**: In real Kubernetes usage, Deployment is usually preferred over both because it adds rolling updates and rollback support.

---

## Rolling Updates
**Quick explanation**: Updating application code, image versions, or configuration changes can temporarily affect users if pods are replaced all at once. Rolling updates solve this by changing pods gradually, keeping the service available during the update.

- Keeps the application available while new versions are deployed
- Replaces older pods gradually instead of all at once
- Reduces downtime and user impact
- Usually managed by a Deployment

**Why this matters**: If a service is live and you update the container image or configuration, a sudden restart of every pod could interrupt user traffic. Rolling updates reduce this risk by creating new pods first, validating them, and then removing old ones.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deploy
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
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
          image: nginx:1.23
```

```bash
# Create the deployment
kubectl apply -f deployment.yaml

# Check rollout status
kubectl rollout status deployment/myapp-deploy

# Update the image to a newer version
kubectl set image deployment/myapp-deploy myapp-container=nginx:1.25

# Watch the rollout progress
kubectl rollout history deployment/myapp-deploy
kubectl rollout undo deployment/myapp-deploy
```

**Summary**: Rolling updates allow you to change application versions safely without taking the service fully offline. This is especially important for production workloads where user experience matters.

**Note**: A bad update can be rolled back with `kubectl rollout undo`, which helps recover quickly if a new version introduces issues.

---

## Deployment (Modern Recommendation)
**Quick explanation**: A Deployment is the recommended Kubernetes workload object for running applications in production. It manages ReplicaSets behind the scenes and provides rolling updates, scaling, and rollback support.

**Definition**: A Deployment tells Kubernetes how to create and maintain the desired number of pod replicas for an application.

**Configuration fields**:
- **metadata**: Name and labels of the Deployment
- **replicas**: Number of desired pod instances
- **selector**: Matches the labels used by the pod template
- **template**: Defines the pod specification and container image
- **strategy**: Controls how updates happen, usually with rolling updates

**Example deployment configuration**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deploy
  labels:
    app: myapp
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
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
          image: nginx:latest
          ports:
            - containerPort: 80
```

```bash
# Create the deployment
kubectl apply -f deployment.yaml

# Check deployment and pods
kubectl get deployment
kubectl get rs
kubectl get pods

# Wait until the rollout is complete
kubectl rollout status deployment/myapp-deploy

# Scale the deployment
kubectl scale deployment myapp-deploy --replicas=5

# Update the image version
kubectl set image deployment/myapp-deploy myapp-container=nginx:1.25

# Check rollout history
kubectl rollout history deployment/myapp-deploy

# Roll back to the previous version if needed
kubectl rollout undo deployment/myapp-deploy

# Delete the deployment
kubectl delete deployment myapp-deploy
```

**Summary**: A Deployment creates and manages ReplicaSets for you, then keeps the application available while it updates, scales, and rolls back safely.

**Important clarification**: Deployment and ReplicaSet do not have major differences in how they manage replica pods. A Deployment is essentially a higher-level controller that manages a ReplicaSet for you. The main difference is the **kind** and the additional features Deployment adds, such as rolling updates, revisions, and rollback support.

**Note**: Deployment is the preferred object for most real-world Kubernetes workloads because it simplifies updates and reduces downtime. The `RollingUpdate` strategy ensures that pods are replaced gradually instead of all at once.

---

## Namespaces
**Quick explanation**: A namespace is a logical partition inside a Kubernetes cluster. It is used to organize resources, isolate workloads, and divide a cluster into smaller, safer environments such as development, staging, and production.

**Definition**: A namespace is a virtual boundary that groups related Kubernetes resources, such as Pods, Services, Deployments, and ConfigMaps.

**Real-world analogy**: A Kubernetes cluster is like a large office building. Each department can have its own floor, room, or section. Namespaces are like those departments: they keep different teams and workloads separated so they do not collide with each other.

**Example analogy**:
- **Development team**: lives in the `dev` namespace
- **Production team**: lives in the `prod` namespace
- **Shared tools**: may live in the `kube-system` namespace

**Default namespace**:
- Kubernetes has a default namespace called `default`.
- It is used when no namespace is explicitly specified.
- Any resource created without a namespace goes into `default`.

```bash
kubectl get namespaces
kubectl get pods
kubectl get pods -n default
```

**Namespace isolation**:
- Resources in one namespace are isolated from resources in another namespace.
- Names can repeat across namespaces, but not within the same namespace.
- This allows teams to reuse names without conflicts.

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

```bash
# Create a namespace using YAML
kubectl apply -f namespace-dev.yaml

# Create a namespace directly from command line
kubectl create namespace dev

# Create a namespace with a custom YAML file
kubectl create -f namespace-dev.yaml
```

**All possible ways to create a new namespace**:
```bash
# 1. Create from YAML
kubectl apply -f namespace-dev.yaml

# 2. Create directly with CLI
kubectl create namespace dev

# 3. Create with a YAML resource definition
cat <<EOF > namespace-dev.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
EOF
kubectl apply -f namespace-dev.yaml

# 4. Create from a file generated in the current directory
kubectl create -f namespace-dev.yaml
```

**How to add a namespace in a YAML manifest**:
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: prod
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: prod
spec:
  replicas: 2
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

**How to use the `--namespace` flag**:
```bash
kubectl get pods --namespace=dev
kubectl get services --namespace=prod
kubectl logs myapp-pod --namespace=dev
kubectl delete pod myapp-pod --namespace=dev
```

**Set the namespace for the current context**:
```bash
kubectl config set-context --current --namespace=dev
kubectl get pods
```

**Set the namespace explicitly using a variable**:
```bash
kubectl config set-context --current --namespace=${NAMESPACE}
```

**Alternative pattern**:
```bash
kubectl config set-context --current --namespace=my-team
kubectl get pods
kubectl get svc
```

**How to view the active namespace**:
```bash
kubectl config view --minify | grep namespace
kubectl config get-contexts
```

**Use all namespaces**:
```bash
kubectl get pods --all-namespaces
kubectl get svc --all-namespaces
kubectl get all -A
```

**Namespace DNS and service resolution**:
Kubernetes provides automatic DNS names for Services. Pods can resolve Services using a DNS name instead of direct IP addresses.

**Service DNS format**:
- `<service-name>`
- `<service-name>.<namespace>`
- `<service-name>.<namespace>.svc.cluster.local`

**Example**:
```bash
kubectl get svc -A
```

If you have a Service named `frontend` in the `dev` namespace, then the DNS names would be:
- `frontend`
- `frontend.dev`
- `frontend.dev.svc.cluster.local`

**How it works**:
- Each namespace gets its own DNS domain.
- Pods can resolve the Service name using the namespace scope.
- A pod in the same namespace can usually just call `frontend`.
- A pod in a different namespace must use `frontend.dev` or the full FQDN.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend
  namespace: dev
spec:
  selector:
    app: frontend
  ports:
    - port: 80
      targetPort: 80
```

```bash
# Inside a pod in the same namespace
curl http://frontend

# From another namespace
curl http://frontend.dev
```

**Automatically created DNS format**:
Every Service gets a DNS entry in this format:
- `<service-name>.<namespace>.svc.cluster.local`

Breakdown:
- `service-name`: name of the Kubernetes Service
- `namespace`: namespace where the Service lives
- `svc`: indicates it is a Kubernetes Service DNS record
- `cluster.local`: default cluster domain

Example:
- `frontend.dev.svc.cluster.local`

This format is the fully qualified domain name (FQDN) used internally by the cluster.

**Namespace resource quota**:
A ResourceQuota limits how many resources a namespace can consume, helping prevent one team from exhausting cluster capacity.

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-quota
  namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
```

```bash
kubectl apply -f resourcequota-dev.yaml
kubectl get resourcequota -n dev
kubectl describe quota -n dev
```

**Summary**: Namespaces provide structure, isolation, and resource governance inside a cluster. They are essential for separating dev, test, and production environments and for managing DNS and quotas effectively.

**Note**: The `default` namespace is the fallback namespace. If you do not specify a namespace, Kubernetes places the resource there unless the context is configured differently.

**Example full workflow**:
```bash
# 1. Create a namespace
kubectl create namespace dev

# 2. Set current context to dev
kubectl config set-context --current --namespace=dev

# 3. Create a deployment in that namespace
kubectl create deployment myapp --image=nginx

# 4. Check resources in the namespace
kubectl get pods
kubectl get svc

# 5. Check all namespaces
kubectl get all -A
```

---

## Encrypting Secret Data at Rest
**Official Kubernetes documentation**: https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/

**Quick explanation**: Kubernetes can encrypt API resource data before it is stored in etcd. This is especially important for sensitive objects like `Secret`, but the same mechanism can also apply to `ConfigMap` or other configured resources. This protects data at rest in etcd, even if someone gains access to the etcd database files.

### How encryption works in Kubernetes
Kubernetes does not encrypt the full disk or protect every part of the system by itself. Instead, the Kubernetes API server is configured with an `--encryption-provider-config` file that tells it how to encrypt data before writing to etcd.

According to the official docs:
- By default, Kubernetes stores resource data in etcd in plain text unless encryption is explicitly configured.
- The API server reads the `EncryptionConfiguration` file and uses the configured provider to encrypt selected resources.
- The first provider in the list is used for encryption of new writes.
- When reading existing data, Kubernetes tries each configured provider in order to decrypt it.

Example configuration:
```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
      - configmaps
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: <BASE64-ENCODED-SECRET>
      - identity: {}
```

This means:
- `aescbc` is the real encryption provider
- `identity` is a fallback that allows reading unencrypted data during a migration
- after migration, the `identity` provider is usually removed so Kubernetes cannot read plaintext accidentally

The Kubernetes docs also note that `Secret` objects are a common target for this feature because they often contain passwords, tokens, and private credentials.

**Important**: This is encryption at rest for API data in etcd, not filesystem encryption inside containers. For data inside mounted volumes, you still need volume-level or application-level encryption.

**Typical control-plane flow**:
1. A client creates or updates a `Secret` via the Kubernetes API.
2. The `kube-apiserver` receives the request.
3. The API server applies the configured encryption provider.
4. The encrypted result is stored in etcd.
5. When a client reads the object, the API server decrypts it before returning it.

This is why etcd is critical for Kubernetes state and why encryption configuration must be protected carefully.

---

### What is etcdctl?
`etcdctl` is the command-line client for etcd. It lets administrators inspect the key/value store that Kubernetes uses for its cluster state.

In Kubernetes, etcd stores the cluster state, including objects such as Pods, Services, and Secrets. The official Kubernetes doc shows an example of verifying encryption with `etcdctl` after creating a Secret:

```bash
kubectl create secret generic secret1 -n default --from-literal=mykey=mydata

ETCDCTL_API=3 etcdctl \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  get /registry/secrets/default/secret1 | hexdump -C
```

If encryption is enabled, the output will not show plain text. You will see a value that is prefixed with something like:
```text
k8s:enc:aescbc:v1:key1:
```

This tells you the stored Secret was encrypted using the `aescbc` provider and key `key1`.

**Why this matters**:
- Without encryption, etcd contains plain-text resource objects
- With encryption at rest, even if etcd files are read, the sensitive data is protected
- `etcdctl` is a troubleshooting and verification tool for checking whether the data was written in encrypted form

**Official etcd docs**: https://etcd.io/docs/latest/install/

---

### Install `etcdctl` on Linux
The `etcdctl` binary is usually provided by the etcd package. The installation package name varies depending on the Linux distribution.

#### Debian / Ubuntu
```bash
sudo apt-get update
sudo apt-get install etcd-client
```

#### Ubuntu / Debian (alternative if package name differs)
```bash
sudo apt-cache search etcd | grep client
sudo apt-get install etcd-client
```

#### RHEL / CentOS / Rocky / AlmaLinux
```bash
sudo yum install -y etcd
```

or
```bash
sudo dnf install -y etcd
```

#### Fedora
```bash
sudo dnf install -y etcd
```

#### OpenSUSE / SLES
```bash
sudo zypper install etcd
```

#### Arch Linux
```bash
sudo pacman -S etcd
```

#### Alpine Linux
```bash
sudo apk add etcd
```

**Check installation**:
```bash
etcdctl --version
```

**Note**: In some distributions the client may come as part of the `etcd` package, and in others it is packaged separately as `etcd-client`. The exact name depends on the distro and version repository.

---

### `kubectl get pods -n kube-system`
This command lists all Pods in the `kube-system` namespace:

```bash
kubectl get pods -n kube-system
```

It is used to inspect the Kubernetes control-plane and cluster add-ons that run inside that namespace, such as:
- `kube-apiserver`
- `etcd`
- `kube-scheduler`
- `kube-controller-manager`
- `coredns`
- networking and cluster add-ons depending on the environment

The Kubernetes components page explains that the control plane manages the overall state of the cluster and includes components like the API server and etcd. The `kube-system` namespace is where many of those system-critical components live.

Examples:
```bash
kubectl get pods -n kube-system -o wide
kubectl get pods -A
kubectl describe pod <pod-name> -n kube-system
kubectl logs <pod-name> -n kube-system
```

**Why this is useful**:
- Check whether the control plane is healthy
- Confirm that etcd and API server pods are running
- Troubleshoot cluster issues
- Investigate failed add-ons or networking components

**Important**: `kube-system` is not for normal application workloads; it is reserved for Kubernetes system components and cluster services.

---

## Docker Security
**Quick explanation**: Docker containers are not automatically isolated at the same level as a virtual machine. If a container is started with elevated privileges, it can abuse host-like capabilities. Security is improved when you run containers as a non-root user and remove unnecessary Linux capabilities.

### Running containers with elevated permissions
The biggest risk is starting a container in privileged mode:

```bash
docker run --privileged nginx
```

`--privileged` gives the container nearly all capabilities, including device access and strong host-like control. This is useful for debugging or special workloads, but it should be avoided for normal production containers.

Other dangerous patterns:
- Running as root inside the container
- Mounting the host filesystem or Docker socket
- Using host networking or host PID namespace
- Sharing the host network namespace, IPC, or group permissions unnecessarily

Examples:
```bash
docker run --privileged --pid host --network host nginx
```

This kind of setup can let a process escape the container boundary or interfere with the host.

### Linux capabilities and root user behavior
In Linux, capabilities are fine-grained privileges. The root user has many capabilities by default, but a non-root user normally does not. Docker containers inherit this model.

Common capabilities include:
- `NET_ADMIN` — modify network interfaces and firewall rules
- `NET_RAW` — create raw sockets
- `SYS_ADMIN` — broad system administration access
- `MKNOD` — create device nodes
- `SYS_PTRACE` — inspect other processes

When a container runs with extra privileges, it can do more than an application should normally be allowed to do.

**Important rule**:
- Do not grant capabilities unless the application really needs them.
- Prefer dropping all and adding only the required ones.

Examples:
```bash
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE nginx
```

This removes all capabilities and then adds only the minimal capability needed.

### `CAP_SYS_ADMIN` and similar capabilities
`CAP_SYS_ADMIN` is one of the most dangerous capabilities because it allows system administration actions that can affect the host or container runtime behavior.

Example:
```bash
docker run --cap-add=SYS_ADMIN nginx
```

This is risky because it can enable operations like:
- mount/umount filesystems
- modify kernel-level behavior
- manipulate networking and system settings
- interfere with the host environment when combined with other privileged settings

Other dangerous capabilities include:
- `CAP_SYS_MODULE` — load kernel modules
- `CAP_SYS_PTRACE` — debug processes
- `CAP_NET_ADMIN` — modify networking rules
- `CAP_MKNOD` — create device files

**Best practice**: never add broad capabilities to a container unless you fully understand the impact.

### Where capabilities are managed
Capabilities are not usually stored in a user profile file. They are Linux kernel attributes attached to processes or binary files.

For a running process, check capabilities with:
```bash
capsh --print
getpcaps 1234
```

For a file, set file capabilities with:
```bash
setcap cap_net_bind_service=+ep /usr/local/bin/mybinary
getcap /usr/local/bin/mybinary
```

This means:
- `capsh --print` shows the effective capabilities of the current process
- `getpcaps` shows a process capability set
- `setcap` and `getcap` are used for file-based capabilities on Linux

This is different from a user “capabilities file” in the sense that the real capability state is attached to the process or executable, not stored as a normal user config file.

### Best practices for Docker security
1. Run containers as a non-root user
```dockerfile
FROM nginx
RUN useradd -r -s /usr/sbin/nologin appuser
USER appuser
```

2. Drop all capabilities by default
```bash
docker run --cap-drop=ALL nginx
```

3. Add only the required capabilities
```bash
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE nginx
```

4. Avoid `--privileged` unless absolutely required
```bash
docker run --privileged nginx
```
This should be the exception, not the default.

5. Use read-only filesystem when possible
```bash
docker run --read-only nginx
```

6. Do not mount the Docker socket into containers
```bash
docker run -v /var/run/docker.sock:/var/run/docker.sock nginx
```
This is a common container breakout risk.

7. Use seccomp, AppArmor, or SELinux profiles
```bash
docker run --security-opt seccomp=default.json nginx
```

8. Prefer minimal base images
- Smaller images reduce attack surface
- Fewer installed packages means fewer vulnerable binaries

9. Use image scanning and patching
- Rebuild images regularly
- Remove unnecessary packages and tools
- Keep OS packages current

10. Use resource limits and network restrictions
```bash
docker run --memory=256m --cpus=0.5 --network=none nginx
```

11. Keep host and container namespaces separate
- Avoid `--pid host`
- Avoid `--network host`
- Avoid `--privileged`

12. Limit volumes and bind mounts
- Mount only the exact directories the app needs
- Do not expose `/etc`, `/var/run`, or `/proc` unnecessarily

### Practical secure example
```bash
docker run -d \
  --name webapp \
  --user 1000:1000 \
  --cap-drop=ALL \
  --cap-add=NET_BIND_SERVICE \
  --read-only \
  -p 8080:80 \
  nginx:stable
```

This design keeps the container from running as root and removes unnecessary privileges while allowing only the required capability.

**Summary**: A container should run with the least privilege possible. Root and `CAP_SYS_ADMIN`-style permissions should be avoided unless a real requirement exists. The safer pattern is: non-root user + `--cap-drop=ALL` + add only minimal capabilities + strong runtime isolation.

---

## Final Notes
**The lifecycle of a simple app**:
1. Create a pod definition.
2. Manage replicas with ReplicationController or ReplicaSet.
3. Use Deployment to manage updates and rollout safely.
4. Scale with `kubectl scale` or update YAML and apply it again.
5. Inspect with `kubectl get`, `kubectl describe`, and `kubectl logs`.

This structure keeps the same application example (`myapp`) throughout the notes, making it easier to understand how Kubernetes objects relate to each other.

**Recommended order to study**:
- Pods
- ReplicationController
- ReplicaSet
- Deployment
- kubectl troubleshooting commands

**Example workflow**
```bash
kubectl create -f pod.yaml
kubectl create -f rc.yaml
kubectl create -f rs.yaml
kubectl create -f deployment.yaml
kubectl get pods
kubectl describe deployment myapp-deploy
```

**Summary**: Kubernetes moves from basic pod creation to replicated workloads and finally to managed deployments for real-world application delivery.

---

## Kubernetes Pod Security Context
**Quick explanation**: A Pod security context defines the security settings for a pod or container. It controls things like which user should run the process, whether the container can run privileged, and which Linux capabilities are allowed. This is one of the main ways Kubernetes enforces least privilege.

### What is security context?
A `securityContext` is a Kubernetes field used to define security-related configuration for a Pod or an individual container.

It can be placed at:
- Pod level: applies to all containers in the pod
- Container level: applies to a specific container only

Example:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  containers:
    - name: app
      image: nginx
      securityContext:
        allowPrivilegeEscalation: false
        readOnlyRootFilesystem: true
```

This controls how the container process runs and what privileges it has at runtime.

**Why it matters**:
- runs containers as a non-root user
- reduces privilege escalation risk
- minimizes container attack surface
- helps secure workloads in production

---

### What is `runAsUser`?
`runAsUser` sets the user ID (UID) that the container process should run as.

Example:
```yaml
spec:
  securityContext:
    runAsUser: 1000
```

This tells Kubernetes to start the container process as UID 1000 instead of the default root user.

**Why it is important**:
- reduces the risk of running as root inside the container
- better matches least-privilege principles
- avoids direct root-level actions inside the app process

Example with Pod + container:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: non-root-pod
spec:
  securityContext:
    runAsUser: 1000
  containers:
    - name: app
      image: nginx
```

You can also set it at the container level:
```yaml
containers:
  - name: app
    image: nginx
    securityContext:
      runAsUser: 1000
```

**Good practice**: run application containers as a non-root UID whenever possible.

---

### What are capabilities?
Linux capabilities are fine-grained privileges that can be granted to a process instead of giving full root access. In Kubernetes, capabilities are controlled with the `securityContext.capabilities` field.

Example:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: capabilities-demo
spec:
  containers:
    - name: app
      image: nginx
      securityContext:
        capabilities:
          drop:
            - ALL
          add:
            - NET_BIND_SERVICE
```

This means:
- `drop: [ALL]` removes all capabilities
- `add: [NET_BIND_SERVICE]` grants only the needed capability

**Why this matters**:
- root is powerful, but Linux capabilities allow a narrower and more controlled set of permissions
- a container can be allowed a small subset of privileges without full root access
- it reduces the chance of abuse if the application is compromised

Common examples of capabilities:
- `NET_ADMIN` — modify network configuration
- `NET_RAW` — raw socket access
- `SYS_ADMIN` — broad administrative power
- `MKNOD` — create device files
- `SETUID` / `SETGID` — change process ownership or group

**Best practice**:
- drop all capabilities by default
- add only the capability the app truly needs
- avoid `privileged: true` unless absolutely required

**Example of locking down a container**:
```yaml
securityContext:
  runAsNonRoot: true
  allowPrivilegeEscalation: false
  capabilities:
    drop:
      - ALL
```

This is a recommended hardening pattern for many workloads.

**Summary**: `securityContext` gives you control over container security, `runAsUser` sets the Linux user ID, and `capabilities` let you grant or remove specific privileges in a granular way instead of giving full root power.

---

## Services
**Quick explanation**: A Service is a stable networking abstraction that exposes a set of pods to the cluster or to the outside world. Pods can be replaced, restarted, or scaled, but the Service keeps a consistent entry point.

**Definition**: A Service defines a logical set of pods and provides a stable IP, port, and DNS name so other components can communicate with them reliably.

**Why services matter**:
- Pods are ephemeral and their IP addresses can change.
- A Service gives you a stable endpoint for traffic.
- It enables internal communication between workloads.
- It can also expose apps externally depending on the Service type.

**Real-world analogy**: A Service is like a company front desk or receptionist. The backend team members change often, but the front desk always gives customers a single contact point.

**Example service**
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

```bash
# Create the service
kubectl apply -f service.yaml

# List services
kubectl get svc

# Check service details
kubectl describe svc myapp-service
```

**Use cases**:
- Reach an application from another pod in the cluster
- Expose a web app internally only
- Route traffic to multiple replicas behind one logical endpoint
- Provide stable DNS and access for other workloads

**Example internal communication**:
```yaml
apiVersion: v1
kind: Deployment
metadata:
  name: myapp
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

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 80
```

```bash
# From another pod in the same cluster
curl http://myapp
```

**Networking and external communication**:
Services are the primary networking object used to expose pods to other pods or to the outside world. They enable traffic routing without exposing individual pod IP addresses directly.

**Important**: Pods are not static. They can restart or move between nodes. A Service keeps the endpoint stable, even if the underlying pods change.

### Service Type: ClusterIP
**Quick explanation**: ClusterIP is the default Service type. It exposes the service only inside the Kubernetes cluster.

**Definition**: ClusterIP assigns a virtual IP address inside the cluster so other pods and internal workloads can communicate with it.

**Use cases**:
- Internal-only application communication
- Communication between frontend and backend services
- Services that should not be externally exposed

**Example**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  type: ClusterIP
  selector:
    app: backend
  ports:
    - port: 8080
      targetPort: 8080
```

**Limitations**:
- Not directly reachable from outside the cluster
- Not suitable for public internet access
- Not ideal for a user-facing application without an ingress or load balancer

### Service Type: NodePort
**Quick explanation**: NodePort exposes the Service on a static port on each node, making it accessible from outside the cluster.

**Definition**: NodePort opens a port on every cluster node, typically in the range 30000-32767, and forwards traffic to the Service.

**Use cases**:
- Simple external access for development or testing
- Learning Kubernetes networking
- Quick demos where a cloud load balancer is not available

**Example**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-nodeport
spec:
  type: NodePort
  selector:
    app: frontend
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```

```bash
# Access the app using any node IP and node port
http://<node-ip>:30080
```

**Limitations**:
- Exposes a high port on every node
- Not ideal for production workloads
- Less flexible than Ingress or cloud-managed load balancers
- Requires additional firewall and security planning

### Service Type: LoadBalancer
**Quick explanation**: LoadBalancer creates a cloud provider load balancer in front of the Service and exposes it externally.

**Definition**: LoadBalancer provisions a cloud-managed external load balancer, which routes traffic to your Service automatically.

**Use cases**:
- Public-facing web applications
- Production workloads requiring internet access
- Environments where a load balancer is provided by the cloud platform

**Example**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-lb
spec:
  type: LoadBalancer
  selector:
    app: frontend
  ports:
    - port: 80
      targetPort: 80
```

```bash
kubectl get svc
kubectl describe svc frontend-lb
```

**Limitations**:
- Depends on cloud provider support
- Can add cost depending on the environment
- More infrastructure overhead than ClusterIP
- Not always the best choice for multi-service internal routing

**Service type summary**
- **ClusterIP**: Internal access only; default and most common
- **NodePort**: External access via node IP and a port; simpler but not ideal for production
- **LoadBalancer**: Cloud-managed external access; best for public services

**Practical workflow**
```bash
# Create a deployment
kubectl create deployment web --image=nginx

# Expose it internally
kubectl expose deployment web --port=80 --target-port=80

# Expose it as a NodePort
kubectl expose deployment web --port=80 --target-port=80 --type=NodePort

# Expose it as a LoadBalancer
kubectl expose deployment web --port=80 --target-port=80 --type=LoadBalancer
```

**Summary**: Services are the networking layer that connects pods together and gives them stable access paths. Without Services, Kubernetes workloads would be hard to reach reliably because pods are dynamic and short-lived.

---

## kubectl Explain and API Resources
**Quick explanation**: `kubectl explain` and `kubectl api-resources` are used to inspect the Kubernetes API and learn how objects are structured. They are especially useful when you are unsure about fields, resource kinds, or how to write a valid manifest.

**Use cases**:
- Discover available Kubernetes resource types
- Check which fields exist in a resource definition
- Learn the required or optional attributes for an object
- Understand object structure before writing YAML files
- Troubleshoot invalid manifests or missing fields

**List available API resources**
```bash
kubectl api-resources
kubectl api-resources --namespaced
kubectl api-resources --verbs=list
```

**Explain a resource**
```bash
kubectl explain pod
kubectl explain deployment
kubectl explain service
kubectl explain deployment.spec
kubectl explain deployment.spec.template.spec.containers
```

**Explain with nested fields**
```bash
kubectl explain pod --recursive
kubectl explain deployment --recursive | head -50
```

**Example**
```bash
kubectl explain deployment.spec.strategy
kubectl explain service.spec.ports
kubectl explain namespace
```

**Why this is useful**:
- It helps you learn the API without memorizing every field.
- It reduces YAML mistakes.
- It is a fast way to inspect how Kubernetes expects a resource to be structured.

**Summary**: `kubectl api-resources` tells you what exists in the cluster API, and `kubectl explain` tells you how each resource is defined. Together, they are essential for writing correct Kubernetes manifests.

---

## kubectl Imperative Commands
**Quick explanation**: Imperative commands are direct kubectl commands used to create, update, or delete resources without writing YAML first. They are fast and convenient for quick tasks, testing, and troubleshooting.

**Use cases**:
- Quick one-off deployments
- Learning Kubernetes workflows
- Creating simple resources without a manifest
- Scaling and updating resources quickly
- Troubleshooting in live clusters

**Common imperative examples**
```bash
# Create a deployment directly from the command line
kubectl create deployment nginx --image=nginx

# Create a pod directly
kubectl run busybox --image=busybox --restart=Never -- sleep 3600

# Expose a deployment as a service
kubectl expose deployment nginx --port=80 --target-port=80

# Scale a deployment
kubectl scale deployment nginx --replicas=3

# Update an image in a deployment
kubectl set image deployment/nginx nginx=nginx:1.25

# Delete resources
kubectl delete deployment nginx
kubectl delete service nginx
kubectl delete pod busybox
```

**Create a namespace imperatively**
```bash
kubectl create namespace dev
kubectl get ns
```

**Create a configmap imperatively**
```bash
kubectl create configmap app-config --from-literal=MODE=prod
kubectl get configmap
```

**Create a secret imperatively**
```bash
kubectl create secret generic app-secret --from-literal=DB_PASSWORD=secret123
kubectl get secret
```

**Imperative commands vs YAML**:
- **Imperative**: fast, simple, good for learning and quick tasks
- **Declarative**: preferred for production and version-controlled infrastructure

**Example of declarative workflow**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp
          image: nginx
```

```bash
kubectl apply -f deployment.yaml
```

**Summary**: Imperative commands are great for quick tasks and learning, while declarative YAML is the standard approach for repeatable, managed, and versioned Kubernetes configuration.

---