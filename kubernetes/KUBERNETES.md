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
  - [Node Selectors Logging](#node-selectors-logging)
    - [What is node selector?](#what-is-node-selector)
    - [Node selector in declarative configuration](#node-selector-in-declarative-configuration)
    - [What are labels?](#what-are-labels)
    - [How to label a node](#how-to-label-a-node)
  - [Node Affinity](#node-affinity)
    - [What is Node Affinity?](#what-is-node-affinity)
    - [What is the limitation of node selectors?](#what-is-the-limitation-of-node-selectors)
    - [Node affinity types](#node-affinity-types)
    - [Syntax explanation (Affinity only)](#syntax-explanation-affinity-only)
    - [Examples of Node Affinity in declarative Pod configuration](#examples-of-node-affinity-in-declarative-pod-configuration)
  - [Taints and Tolerations](#taints-and-tolerations)
    - [What are taints and tolerations?](#what-are-taints-and-tolerations)
    - [Node and Pod relationship](#node-and-pod-relationship)
    - [Commands to taint a node](#commands-to-taint-a-node)
    - [Types of taint effects](#types-of-taint-effects)
    - [Tolerations in declarative Pod config (and quoting)](#tolerations-in-declarative-pod-config-and-quoting)
    - [What does `kubectl describe node kubemaster` do?](#what-does-kubectl-describe-node-kubemaster-do)
    - [Does the master/control-plane node schedule normal pods?](#does-the-mastercontrol-plane-node-schedule-normal-pods)
  - [Node Affinity vs Taints](#node-affinity-vs-taints)
    - [Differences](#differences)
    - [Use cases](#use-cases)
    - [Can we combine them?](#can-we-combine-them)
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
  - [Resource Requirements](#resource-requirements)
    - [What is a resource in Kubernetes?](#what-is-a-resource-in-kubernetes)
    - [Problems resources face](#problems-resources-face)
    - [Specifying resources in a pod](#specifying-resources-in-a-pod)
    - [CPU counts and memory metrics](#cpu-counts-and-memory-metrics)
      - [CPU](#cpu)
      - [Memory](#memory)
    - [What happens when pods exceed CPU?](#what-happens-when-pods-exceed-cpu)
    - [What happens when pods exceed Memory?](#what-happens-when-pods-exceed-memory)
    - [Explain throttling](#explain-throttling)
    - [Explain OOM](#explain-oom)
    - [Pod default configuration](#pod-default-configuration)
    - [What happens without limits?](#what-happens-without-limits)
    - [What happens with limits?](#what-happens-with-limits)
    - [Can we throttle memory like CPU?](#can-we-throttle-memory-like-cpu)
  - [LimitRange](#limitrange)
    - [What is LimitRange?](#what-is-limitrange)
    - [When to use LimitRange](#when-to-use-limitrange)
    - [LimitRange example: CPU default and minimum (Declarative)](#limitrange-example-cpu-default-and-minimum-declarative)
    - [LimitRange example: Memory default and maximum (Declarative)](#limitrange-example-memory-default-and-maximum-declarative)
    - [Imperative example: LimitRange with CPU](#imperative-example-limitrange-with-cpu)
    - [Imperative example: LimitRange with Memory](#imperative-example-limitrange-with-memory)
  - [Resource Quotas](#resource-quotas)
    - [What are resource quotas?](#what-are-resource-quotas)
    - [Why are Resource Quotas configured at the Namespace level?](#why-are-resource-quotas-configured-at-the-namespace-level)
  - [Service Account](#service-account)
    - [What is a Service Account?](#what-is-a-service-account)
    - [Difference between User and Service Account](#difference-between-user-and-service-account)
    - [What are tokens?](#what-are-tokens)
    - [How do they work?](#how-do-they-work)
    - [kubectl commands for ServiceAccount](#kubectl-commands-for-serviceaccount)
    - [Where is it mounted in a pod?](#where-is-it-mounted-in-a-pod)
    - [How to create a ServiceAccount](#how-to-create-a-serviceaccount)
      - [Imperative way](#imperative-way-1)
      - [Declarative way](#declarative-way-1)
    - [How to associate a ServiceAccount to a pod](#how-to-associate-a-serviceaccount-to-a-pod)
    - [ServiceAccount per namespace](#serviceaccount-per-namespace)
    - [Does Kubernetes automatically create a token?](#does-kubernetes-automatically-create-a-token)
  - [Talking to the cluster outside the cluster (CI/CD tools, Monitoring, etc.)](#talking-to-the-cluster-outside-the-cluster-cicd-tools-monitoring-etc)
    - [How to create a token](#how-to-create-a-token)
    - [How to decode a token](#how-to-decode-a-token)
      - [Base decode with Python](#base-decode-with-python)
      - [Example token decode output](#example-token-decode-output)
    - [Check RBAC configuration files](#check-rbac-configuration-files)
  - [Services](#services)
    - [Service Type: ClusterIP](#service-type-clusterip)
    - [Service Type: NodePort](#service-type-nodeport)
    - [Service Type: LoadBalancer](#service-type-loadbalancer)
  - [Network Policies](#network-policies)
    - [How it works with other Kubernetes components](#how-it-works-with-other-kubernetes-components)
    - [Example: frontend, api, and db pods](#example-frontend-api-and-db-pods)
      - [1. Allow frontend to reach the api](#1-allow-frontend-to-reach-the-api)
      - [2. Allow api to reach the db](#2-allow-api-to-reach-the-db)
      - [3. Allow API to egress to DB](#3-allow-api-to-egress-to-db)
    - [Ingress and Egress](#ingress-and-egress)
    - [Example: a restricted policy with DNS egress and specific app access](#example-a-restricted-policy-with-dns-egress-and-specific-app-access)
    - [Pod selector and namespace selector](#pod-selector-and-namespace-selector)
      - [Pod selector](#pod-selector)
      - [Namespace selector](#namespace-selector)
    - [Example with another server outside the cluster](#example-with-another-server-outside-the-cluster)
    - [Summary](#summary)
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

## Node Selectors Logging
**Quick explanation**: A node selector is the simplest way to constrain a pod so it runs only on nodes with a specific label.

### What is node selector?
A `nodeSelector` is a key-value match in a Pod spec.

How it works:
- You label a node, for example `disk=ssd`.
- You add `nodeSelector: { disk: ssd }` to the pod.
- The scheduler places the pod only on nodes that have that exact label.

If no node matches, the pod remains `Pending`.

---

### Node selector in declarative configuration
Example pod snippet:
```yaml
spec:
  nodeSelector:
    disktype: ssd
```

Minimal full example:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: selector-demo
spec:
  nodeSelector:
    disktype: ssd
  containers:
    - name: app
      image: nginx
```

---

### What are labels?
Labels are key-value metadata attached to Kubernetes objects such as nodes, pods, and namespaces.

Examples:
- `disktype=ssd`
- `zone=us-east-1a`
- `environment=prod`

Why labels matter:
- selection and grouping of resources
- scheduling decisions (nodeSelector / affinity)
- operational filtering with `kubectl`

---

### How to label a node
Add or update a label:
```bash
kubectl label nodes <node-name> disktype=ssd
```

Overwrite an existing label value:
```bash
kubectl label nodes <node-name> disktype=hdd --overwrite
```

Show node labels:
```bash
kubectl get nodes --show-labels
kubectl describe node <node-name>
```

Remove a label (trailing `-`):
```bash
kubectl label nodes <node-name> disktype-
```

---

## Node Affinity
**Quick explanation**: Node Affinity is an advanced scheduling rule that lets pods match nodes using richer expressions than node selectors.

### What is Node Affinity?
Node Affinity is configured in `spec.affinity.nodeAffinity` and supports operators such as:
- `In`
- `NotIn`
- `Exists`
- `DoesNotExist`
- `Gt`
- `Lt`

It allows more flexible and readable placement logic than plain `nodeSelector`.

Operator explanations:
- `In`: matches when the node label value is in a provided list of values.
  - Example: key `disktype`, values `[ssd, nvme]` -> node matches if label is `ssd` or `nvme`.
- `NotIn`: matches when the node label value is not in the provided list.
  - Example: key `environment`, values `[dev]` -> node matches if label is not `dev`.
- `Exists`: matches when the label key exists on the node, regardless of value.
  - Example: key `gpu` -> node matches if it has label `gpu=<anything>`.
- `DoesNotExist`: matches when the label key is absent from the node.
  - Example: key `spot` -> node matches only if there is no `spot` label.
- `Gt`: matches when the node label value is an integer greater than the given value.
  - Example: key `cpu-count`, values `[8]` -> node matches if `cpu-count` label is `9` or higher.
- `Lt`: matches when the node label value is an integer less than the given value.
  - Example: key `cpu-count`, values `[16]` -> node matches if `cpu-count` is below `16`.

Notes:
- `Gt` and `Lt` work with integer-like label values.
- `Exists` and `DoesNotExist` do not use the `values` field.

---

### What is the limitation of node selectors?
`nodeSelector` is limited because it only supports exact key-value matching.

Limitations:
- no OR logic across values
- no soft preference (only hard matching)
- no operators like `NotIn`, `Exists`, `Gt`, `Lt`
- difficult to express complex placement policies

Node Affinity addresses these limitations.

---

### Node affinity types
1. `requiredDuringSchedulingIgnoredDuringExecution`
- Hard rule at scheduling time.
- Pod is scheduled only if rule matches.
- If node labels change later, pod is not evicted because of `IgnoredDuringExecution`.

2. `preferredDuringSchedulingIgnoredDuringExecution`
- Soft preference.
- Scheduler tries to honor it but may place pod elsewhere.
- Also not re-evaluated for eviction during execution.

---

### Syntax explanation (Affinity only)
Required (hard) syntax:
```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: disktype
              operator: In
              values:
                - ssd
```

Preferred (soft) syntax:
```yaml
affinity:
  nodeAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
            - key: zone
              operator: In
              values:
                - us-east-1a
```

Field notes:
- `nodeSelectorTerms` are ORed with each other.
- `matchExpressions` inside one term are ANDed.
- `weight` range is 1-100 for preference scoring.

---

### Examples of Node Affinity in declarative Pod configuration
Hard requirement example:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: affinity-required-demo
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: disktype
                operator: In
                values:
                  - ssd
  containers:
    - name: app
      image: nginx
```

Soft preference example:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: affinity-preferred-demo
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 80
          preference:
            matchExpressions:
              - key: zone
                operator: In
                values:
                  - us-east-1a
  containers:
    - name: app
      image: nginx
```

Combined required + preferred example:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: affinity-combined-demo
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: kubernetes.io/os
                operator: In
                values:
                  - linux
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 50
          preference:
            matchExpressions:
              - key: disktype
                operator: In
                values:
                  - ssd
  containers:
    - name: app
      image: nginx
```

---

## Taints and Tolerations
**Quick explanation**: Taints are applied to nodes to repel pods, while tolerations are added to pods to allow them to be scheduled onto tainted nodes.

### What are taints and tolerations?
A **taint** is a rule on a node that says, "do not place pods here unless they explicitly tolerate this condition."

A **toleration** is a rule on a pod that says, "this pod can run on nodes with a matching taint."

Important behavior:
- taints do not force scheduling by themselves
- tolerations only allow scheduling; they do not guarantee a pod will land on that node
- to target a specific node pool, combine tolerations with labels and node selectors/affinity

---

### Node and Pod relationship
The scheduler decides where a pod runs by checking:
- resource availability
- node selectors/affinity rules
- taints on nodes and tolerations on pods

Relationship summary:
- nodes advertise constraints (including taints)
- pods declare what they can tolerate
- a pod without matching tolerations is blocked from tainted nodes

---

### Commands to taint a node
Apply a taint:
```bash
kubectl taint nodes <node-name> dedicated=infra:NoSchedule
```

Check node taints:
```bash
kubectl describe node <node-name>
kubectl get nodes -o custom-columns=NAME:.metadata.name,TAINTS:.spec.taints
```

Remove a taint (note the trailing `-`):
```bash
kubectl taint nodes <node-name> dedicated=infra:NoSchedule-
```

Control-plane taint examples (name varies by distro/version):
```bash
kubectl taint nodes <control-plane-node> node-role.kubernetes.io/control-plane=:NoSchedule
kubectl taint nodes <control-plane-node> node-role.kubernetes.io/master=:NoSchedule
```

---

### Types of taint effects
Kubernetes supports three taint effects:

1. `NoSchedule`
- Pods without a matching toleration will not be scheduled onto the node.

2. `PreferNoSchedule`
- Soft rule. Scheduler tries to avoid placing non-tolerating pods there, but may still place them if needed.

3. `NoExecute`
- New non-tolerating pods are not scheduled.
- Existing non-tolerating pods are evicted from the node.

---

### Tolerations in declarative Pod config (and quoting)
Pod example with tolerations:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: toleration-demo
spec:
  tolerations:
    - key: "dedicated"
      operator: "Equal"
      value: "infra"
      effect: "NoSchedule"
  containers:
    - name: app
      image: nginx
```

Do values need double quotes?
- Not strictly required for simple strings in YAML.
- Recommended for consistency and to avoid parsing surprises.
- Especially useful when values may look like booleans, numbers, dates, or include special characters.

Equivalent unquoted example (valid YAML):
```yaml
tolerations:
  - key: dedicated
    operator: Equal
    value: infra
    effect: NoSchedule
```

---

### What does `kubectl describe node kubemaster` do?
This command shows detailed information about the node named `kubemaster`.

It includes:
- node labels and annotations
- taints
- capacity and allocatable CPU/memory
- conditions (Ready, MemoryPressure, DiskPressure, etc.)
- running pods and resource usage summary
- recent node-related events

It is commonly used to troubleshoot scheduling issues and verify whether taints are preventing pod placement.

---

### Does the master/control-plane node schedule normal pods?
By default in many kubeadm-style clusters, control-plane nodes are tainted with `NoSchedule` for general workloads.

What this means:
- normal application pods usually do not run there
- control-plane/system components still run there
- if you add a matching toleration (or remove the taint), regular pods can be scheduled there

Check current taints on control-plane nodes:
```bash
kubectl get nodes -o custom-columns=NAME:.metadata.name,TAINTS:.spec.taints
```

In single-node labs, people sometimes remove the control-plane taint to allow workloads:
```bash
kubectl taint nodes <control-plane-node> node-role.kubernetes.io/control-plane:NoSchedule-
kubectl taint nodes <control-plane-node> node-role.kubernetes.io/master:NoSchedule-
```

Use this carefully in production environments.

---

## Node Affinity vs Taints
**Quick explanation**: Both affect pod placement, but they solve different scheduling problems and are often strongest when used together.

### Differences
1. Direction of control
- Node Affinity is pod-driven (the pod asks for matching nodes).
- Taints are node-driven (the node repels pods unless tolerated).

2. Primary purpose
- Node Affinity selects where a pod should run.
- Taints protect nodes from unwanted pods.

3. Scheduling behavior
- Affinity can be hard (`required`) or soft (`preferred`).
- Taints can block (`NoSchedule`), discourage (`PreferNoSchedule`), or evict (`NoExecute`).

4. Matching model
- Affinity uses label expressions (`In`, `NotIn`, `Exists`, `Gt`, etc.).
- Taints require matching tolerations in the pod.

5. Typical ownership
- Application/platform teams define affinity in workload manifests.
- Cluster operators define taints on nodes.

---

### Use cases
Node Affinity common use cases:
- place pods on SSD or GPU nodes
- keep workloads in specific zones or hardware classes
- prefer lower-cost nodes while allowing fallback

Taints common use cases:
- reserve dedicated nodes for special workloads
- protect control-plane nodes from normal applications
- isolate sensitive or high-priority capacity
- evict pods on unhealthy/special-condition nodes (`NoExecute`)

Rule of thumb:
- use Affinity when pods need certain node characteristics
- use Taints when nodes must reject general workloads by default

---

### Can we combine them?
Yes. Combining them is a best-practice pattern for strict workload isolation.

Example strategy:
1. Label and taint a node pool for `payments`.
2. Add Node Affinity in payment pods to require that label.
3. Add matching toleration so those pods are allowed onto tainted nodes.

Practical result:
- Affinity ensures payment pods target the correct nodes.
- Taints ensure non-payment pods are kept out.

Minimal combined pod snippet:
```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: workload
                operator: In
                values:
                  - payments
  tolerations:
    - key: "workload"
      operator: "Equal"
      value: "payments"
      effect: "NoSchedule"
```

Related node commands:
```bash
kubectl label node <node-name> workload=payments
kubectl taint node <node-name> workload=payments:NoSchedule
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
**Quick explanation**: A Pod security context defines the security settings for a pod. It controls things like which user should run the process, whether the container can escalate privileges, and which Linux capabilities are allowed. This is one of the main ways Kubernetes enforces least privilege.

### What is security context?
A `securityContext` is a Kubernetes field used to define security-related configuration for a Pod.

It is defined at the Pod `spec` level:
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
    runAsNonRoot: true
    allowPrivilegeEscalation: false
    capabilities:
      drop:
        - ALL
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
    runAsNonRoot: true
  containers:
    - name: app
      image: nginx
```

**Good practice**: run application containers as a non-root UID whenever possible.

---

### What are capabilities?
Linux capabilities are fine-grained privileges that can be granted to a process instead of giving full root access. In Kubernetes, capabilities are controlled with the Pod `securityContext.capabilities` field.

Example:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: capabilities-demo
spec:
  securityContext:
    capabilities:
      drop:
        - ALL
      add:
        - NET_BIND_SERVICE
  containers:
    - name: app
      image: nginx
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

**Example of locking down a pod**:
```yaml
spec:
  securityContext:
    runAsNonRoot: true
    allowPrivilegeEscalation: false
    capabilities:
      drop:
        - ALL
```

This is a recommended hardening pattern for many workloads.

**Summary**: `securityContext` gives you control over pod security, `runAsUser` sets the Linux user ID, and `capabilities` let you grant or remove specific privileges in a granular way instead of giving full root power.

---

## Resource Requirements
**Quick explanation**: In Kubernetes, resources are the CPU and memory a pod or container is allowed to use. They are important because the scheduler and kubelet need to know how much compute a workload needs in order to place it on a node and keep the cluster stable.

### What is a resource in Kubernetes?
Resources are the compute limits and requests assigned to containers. The two main ones are:
- **CPU**
- **Memory**

Kubernetes uses these values to decide:
- where to place a pod
- how much of the node's capacity can be consumed
- whether a pod can run smoothly or be throttled or killed

Example:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
    - name: app
      image: nginx
      resources:
        requests:
          cpu: "250m"
          memory: "128Mi"
        limits:
          cpu: "500m"
          memory: "256Mi"
```

**requests**: the minimum guaranteed amount of resource the container can rely on.
**limits**: the maximum amount the container is allowed to use.

---

### Problems resources face
When resources are not managed correctly, problems can happen such as:
- a pod gets scheduled onto a node that is already overloaded
- one workload consumes too much CPU and slows down other workloads
- a container tries to use more memory than the node can provide
- the node becomes unstable or unresponsive
- containers are throttled or evicted

This is why Kubernetes has requests, limits, and scheduling logic.

---

### Specifying resources in a pod
You define resources under each container's `resources` block.

Example:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web
spec:
  containers:
    - name: web
      image: nginx
      resources:
        requests:
          cpu: "100m"
          memory: "64Mi"
        limits:
          cpu: "200m"
          memory: "128Mi"
```

This tells Kubernetes:
- minimum requested: 100m CPU and 64Mi memory
- upper bound: 200m CPU and 128Mi memory

**Important**: if you do not specify requests, scheduling may still work, but the pod has no guaranteed minimum.

---

### CPU counts and memory metrics
#### CPU
CPU is measured in cores or fractional cores.

Common CPU units:
- `100m` = 0.1 CPU core
- `500m` = 0.5 CPU core
- `1` = 1 CPU core

Example:
```yaml
resources:
  requests:
    cpu: "500m"
  limits:
    cpu: "1"
```

This means the pod requests half a CPU and is limited to one full CPU.

#### Memory
Memory is measured in bytes and often expressed with binary units such as:
- `Mi` = mebibytes
- `Gi` = gibibytes
- `M` = megabytes
- `G` = gigabytes

Common examples:
- `128Mi`
- `512Mi`
- `1Gi`

Example:
```yaml
resources:
  requests:
    memory: "256Mi"
  limits:
    memory: "512Mi"
```

This means the pod needs at least 256Mi and can use up to 512Mi before it is killed or restarted depending on the policy.

---

### What happens when pods exceed CPU?
When a pod exceeds its CPU limit, Kubernetes does not instantly kill it. Instead, the CPU is throttled.

**CPU throttling**:
- if the pod tries to use more CPU than its limit, the kernel reduces the amount of CPU time given to the process
- the application may become slower, respond more slowly, or appear laggy
- CPU throttling is usually a performance issue, not an immediate crash

Example:
```yaml
resources:
  limits:
    cpu: "500m"
```

If the container tries to use 1 CPU but is limited to 500m, it will be restricted and may run slower.

---

### What happens when pods exceed Memory?
Memory is different from CPU. If a pod exceeds its memory limit, the container may be terminated.

**OOM (Out Of Memory)**:
- the kernel or runtime detects the process has used too much memory
- if it exceeds available memory or limit, the process can be killed
- Kubernetes may restart the container depending on the restart policy

Typical symptoms:
- container restarts repeatedly
- pod enters `CrashLoopBackOff`
- application logs show OOMKilled events

Example:
```bash
kubectl get pods
kubectl describe pod myapp
```

You may see events such as:
```text
OOMKilled
```

---

### Explain throttling
**CPU throttling** is the process of limiting how much CPU time a process receives.

- It is a built-in Linux scheduler behavior
- It is usually not a crash; it is a slowdown
- It happens when CPU requests/limits are not enough for workload demand

Common effect:
- slow database queries
- delayed API responses
- not enough processing power for background jobs

**Key point**: CPU can be throttled. Memory usually cannot be gracefully throttled in the same way; it often leads to OOM or eviction.

---

### Explain OOM
OOM stands for Out Of Memory.

This happens when a process tries to use more memory than the node or container limit allows. In Kubernetes, the container runtime or Linux kernel may kill the process to prevent full node instability.

Examples:
- Java app with a memory leak
- Node app processing large payloads
- database using too much memory

When OOM occurs, Kubernetes often shows:
```bash
kubectl describe pod myapp
```

and the pod may restart with status like:
- `CrashLoopBackOff`
- `Error`
- `OOMKilled`

---

### Pod default configuration
If you do not set resource requests or limits, the pod still runs, but it has no explicit CPU/memory policy.

This means:
- no guaranteed minimum resources
- no upper bound
- it may consume as much as the node allows
- other workloads can be affected

Example of a pod without resource settings:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: no-resource-pod
spec:
  containers:
    - name: app
      image: nginx
```

This is allowed, but it is not ideal for production workloads.

---

### What happens without limits?
If you set only requests and no limits:
- the pod gets guaranteed minimum resources
- but it can use more if the node has free capacity
- the system cannot strictly stop it from using extra memory or CPU

Example:
```yaml
resources:
  requests:
    cpu: "200m"
    memory: "128Mi"
```

This gives a guaranteed baseline but not a hard maximum.

---

### What happens with limits?
If you set limits:
- the pod cannot exceed the configured maximum
- CPU can be throttled
- memory can trigger OOM or termination if exceeded
- scheduling is based on requests, while enforcement is done by the kubelet

Example:
```yaml
resources:
  requests:
    cpu: "250m"
    memory: "128Mi"
  limits:
    cpu: "500m"
    memory: "256Mi"
```

**Summary**: requests are for scheduling and minimum guarantees; limits are for protection and control.

---

### Can we throttle memory like CPU?
Not in the same way as CPU.

- **CPU** can be throttled by the CFS scheduler.
- **Memory** cannot normally be throttled in a clean, graceful way.
- If memory exceeds limit, the process may be OOM-killed.

So memory is enforced by a hard limit and failure condition, while CPU is more often controlled by quota and throttling.

**Best practice**:
- set sane CPU limits to avoid throttling
- set realistic memory limits to avoid OOM
- monitor application behavior with metrics

---

## LimitRange
**Quick explanation**: A `LimitRange` is a Kubernetes object that defines default resource requests and limits for Pods or containers in a namespace. It helps enforce a minimum and maximum size for CPU and memory so workloads do not consume too much cluster capacity.

### What is LimitRange?
A `LimitRange` sets default and minimum/maximum values for resources in a namespace.

It is often used to:
- enforce a standard resource policy
- prevent misconfigured pods from consuming too much memory or CPU
- ensure developers get sensible defaults
- reduce cluster instability

---

### When to use LimitRange
Use a `LimitRange` when:
- multiple teams share a cluster
- you want default resource values for all pods in a namespace
- you want to avoid out-of-control workloads
- you want to enforce allowed CPU and memory ranges per namespace

This is especially useful for shared dev/test/prod namespaces.

---

### LimitRange example: CPU default and minimum (Declarative)
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-limitrange
  namespace: dev
spec:
  limits:
    - type: Container
      min:
        cpu: "100m"
      max:
        cpu: "500m"
      default:
        cpu: "200m"
      defaultRequest:
        cpu: "100m"
```

This means in `dev` namespace:
- minimum CPU per container is 100m
- maximum CPU per container is 500m
- default CPU limit is 200m
- default CPU request is 100m

---

### LimitRange example: Memory default and maximum (Declarative)
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: memory-limitrange
  namespace: dev
spec:
  limits:
    - type: Container
      min:
        memory: "64Mi"
      max:
        memory: "512Mi"
      default:
        memory: "128Mi"
      defaultRequest:
        memory: "64Mi"
```

This means in `dev` namespace:
- minimum memory per container is 64Mi
- maximum memory per container is 512Mi
- default memory limit is 128Mi
- default memory request is 64Mi

---

### Imperative example: LimitRange with CPU
```bash
kubectl create namespace dev
kubectl create configmap cpu-limitrange --from-literal=... 
```

The better and more common imperative style for LimitRange is to create a YAML file and then apply it:

```bash
cat <<EOF > limitrange-cpu.yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-limitrange
  namespace: dev
spec:
  limits:
    - type: Container
      min:
        cpu: "100m"
      max:
        cpu: "500m"
      default:
        cpu: "200m"
      defaultRequest:
        cpu: "100m"
EOF

kubectl apply -f limitrange-cpu.yaml
```

**Check it**:
```bash
kubectl get limitrange -n dev
kubectl describe limitrange cpu-limitrange -n dev
```

---

### Imperative example: LimitRange with Memory
```bash
cat <<EOF > limitrange-memory.yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: memory-limitrange
  namespace: dev
spec:
  limits:
    - type: Container
      min:
        memory: "64Mi"
      max:
        memory: "512Mi"
      default:
        memory: "128Mi"
      defaultRequest:
        memory: "64Mi"
EOF

kubectl apply -f limitrange-memory.yaml
```

**Check it**:
```bash
kubectl get limitrange -n dev
kubectl describe limitrange memory-limitrange -n dev
```

**Key point**: CPU and memory controls are usually defined in separate LimitRange resources or in separate `limits` blocks for clarity.

---

## Resource Quotas
**Quick explanation**: A `ResourceQuota` restricts the total amount of compute resources a namespace can consume. It is used to stop one team or application from using the entire cluster capacity.

### What are resource quotas?
A ResourceQuota is a namespace-level policy that limits the total resources used across all objects in that namespace.

Example:
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: dev
spec:
  hard:
    requests.cpu: "4"
    requests.memory: "8Gi"
    limits.cpu: "8"
    limits.memory: "16Gi"
    pods: "10"
```

This means the `dev` namespace can have total resource consumption up to:
- 4 CPU requested
- 8Gi memory requested
- 8 CPU limit
- 16Gi memory limit
- 10 pods max

---

### Why are Resource Quotas configured at the Namespace level?
A namespace is the boundary where related workloads are grouped together. Resource Quotas are applied at namespace level because:
- teams often share a cluster but need isolated resource usage
- each namespace can be assigned a fair share of cluster capacity
- this prevents one namespace from starving others
- it helps support multi-tenant or team-based environments

In other words, ResourceQuota is used to control the total sum of resources in a namespace, while LimitRange is used to control per-pod or per-container defaults and boundaries.

**Example comparison**:
- **LimitRange**: applies to individual containers or pods
- **ResourceQuota**: applies to all workloads in a namespace combined

**Summary**: LimitRange controls the minimum, maximum, and default values per object, while ResourceQuota controls the total usage allowed across all objects in a namespace.

---

## Service Account
**Quick explanation**: A ServiceAccount is an identity used by workloads inside Kubernetes to authenticate to the Kubernetes API. It is the Kubernetes equivalent of a non-human account used by pods, controllers, and automation tools.

### What is a Service Account?
A ServiceAccount is a Kubernetes object that provides an identity for processes running inside a pod.

It is commonly used for:
- pods that need to talk to the Kubernetes API
- CI/CD systems that deploy resources
- monitoring agents that read cluster data
- controllers and automation jobs

**Example**:
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-sa
  namespace: dev
```

---

### Difference between User and Service Account
**User**:
- represents a human or external identity
- usually managed by an external auth system such as OIDC, LDAP, or an enterprise identity provider
- not created by Kubernetes by default for normal app workloads

**ServiceAccount**:
- represents an application or workload identity inside the cluster
- created inside Kubernetes
- used by pods, jobs, controllers, and automation
- usually scoped to a namespace

**Key difference**:
- a user authenticates to the cluster as a person or external system
- a service account authenticates as a workload running in the cluster

---

### What are tokens?
Tokens are credentials used by Kubernetes to authenticate a ServiceAccount to the API server.

In practice, Kubernetes ServiceAccount tokens are usually JWTs (JSON Web Tokens).

A token usually contains:
- issuer information
- subject (the service account)
- audience
- expiration time
- namespace and name claims

**Example structure**:
```text
eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9...
```

This token is used when a pod or automation tool calls the Kubernetes API.

---

### How do they work?
When a pod is assigned a ServiceAccount, Kubernetes mounts a token into the pod filesystem.

Default mount path:
```bash
/var/run/secrets/kubernetes.io/serviceaccount/
```

Inside that directory you usually see:
- `token`
- `namespace`
- `ca.crt`

Example:
```bash
ls -la /var/run/secrets/kubernetes.io/serviceaccount/
```

Typical output:
```bash
ca.crt  namespace  token
```

The pod can then use that token to authenticate to the Kubernetes API with a certificate authority and API server URL.

**How it works conceptually**:
1. Pod gets a ServiceAccount
2. Kubernetes creates or mounts a token
3. The token is mounted in the container filesystem
4. App uses the token to authenticate to the API server
5. API server checks the token and authorizes the request based on RBAC

---

### kubectl commands for ServiceAccount
Common commands:
```bash
kubectl get serviceaccount
kubectl get sa
kubectl get sa -A

kubectl describe sa default
kubectl describe sa app-sa -n dev

kubectl create sa app-sa -n dev
kubectl delete sa app-sa -n dev
```

To view tokens associated with a service account:
```bash
kubectl get secret -n dev
kubectl describe secret <secret-name> -n dev
```

**Note**: In newer Kubernetes versions, tokens may be projected as projected volume tokens instead of long-lived secret objects, depending on the cluster configuration.

---

### Where is it mounted in a pod?
Kubernetes mounts the ServiceAccount token and related files here:
```bash
/var/run/secrets/kubernetes.io/serviceaccount/
```

It includes:
- `token`: bearer token used for API authentication
- `namespace`: current namespace name
- `ca.crt`: certificate authority for verifying the API server certificate

This is the standard in-cluster authentication path for a pod.

---

### How to create a ServiceAccount
#### Imperative way
```bash
kubectl create serviceaccount app-sa -n dev
```

or
```bash
kubectl create sa app-sa -n dev
```

#### Declarative way
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-sa
  namespace: dev
```

```bash
kubectl apply -f serviceaccount.yaml
```

---

### How to associate a ServiceAccount to a pod
You specify `serviceAccountName` in the Pod spec.

Example:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
  namespace: dev
spec:
  serviceAccountName: app-sa
  containers:
    - name: app
      image: nginx
```

This tells Kubernetes to use `app-sa` for the pod identity.

---

### ServiceAccount per namespace
A ServiceAccount belongs to a namespace.

Example:
```bash
kubectl get sa -n dev
kubectl get sa -n prod
```

This means:
- the same name can exist in different namespaces
- the account is isolated by namespace
- RBAC permissions are usually assigned per namespace

---

### Does Kubernetes automatically create a token?
**Yes, in most cases Kubernetes will automatically provide a token for a ServiceAccount used by a pod.**

Historically, Kubernetes created a Secret containing a token for each default ServiceAccount, but modern clusters often use projected service account tokens mounted via the token controller. The important point is:
- a pod can authenticate to the API using the ServiceAccount token automatically
- the token is mounted into the pod without you manually creating it in the container

For the default ServiceAccount, a pod can often use it automatically if no custom `serviceAccountName` is specified.

Example:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: default-sa-pod
spec:
  containers:
    - name: app
      image: nginx
```

This pod will use the default ServiceAccount of its namespace unless configured otherwise.

---

## Talking to the cluster outside the cluster (CI/CD tools, Monitoring, etc.)
**Quick explanation**: External tools such as CI/CD systems, monitoring agents, and automation scripts also need a way to authenticate to the Kubernetes API. This is typically done with a ServiceAccount and a token.

### How to create a token
Create a ServiceAccount first:
```bash
kubectl create namespace ci
kubectl create serviceaccount ci-bot -n ci
```

Then create a token for that service account.

For modern Kubernetes:
```bash
kubectl create token ci-bot -n ci
```

This prints a JWT token you can use as a bearer token in external tools.

Example:
```bash
export TOKEN=$(kubectl create token ci-bot -n ci)
```

Then use it with `kubectl`:
```bash
kubectl get pods -n ci --token="$TOKEN"
```

or with an external client:
```bash
curl -H "Authorization: Bearer $TOKEN" \
  https://<kubernetes-api-server>:6443/api/v1/namespaces/default/pods
```

---

### How to decode a token
ServiceAccount tokens are usually JWTs, so they can be decoded.

#### Base decode with Python
```bash
python - <<'PY'
import base64, json, os

token = os.environ['TOKEN']
header, payload, sig = token.split('.')

def b64url_decode(s):
    s += '=' * (-len(s) % 4)
    return base64.urlsafe_b64decode(s.encode())

print(json.dumps(json.loads(b64url_decode(payload)), indent=2))
PY
```

#### Example token decode output
```json
{
  "aud": [
    "https://kubernetes.default.svc"
  ],
  "exp": 1750000000,
  "iat": 1740000000,
  "iss": "https://kubernetes.default.svc.cluster.local",
  "kubernetes.io": {
    "namespace": "ci",
    "serviceaccount": {
      "name": "ci-bot",
      "uid": "..."
    }
  },
  "nbf": 1740000000,
  "sub": "system:serviceaccount:ci:ci-bot"
}
```

This shows which namespace and service account the token belongs to.

**Important**: a token is not a password; it is a signed bearer credential that Kubernetes validates against its auth system.

**Best practice**: keep tokens short-lived and rotate them regularly. Use RBAC to limit the permissions of the ServiceAccount.

### Check RBAC configuration files
If your environment stores RBAC policy/configuration files on disk, check this directory:

```bash
/var/rbac
```

Where it can be found:
- on the control-plane node filesystem
- at `/var/rbac`

Useful checks:
```bash
ls -la /var/rbac
find /var/rbac -maxdepth 2 -type f
```

If the directory does not exist, RBAC is likely managed directly by Kubernetes API objects (Roles, ClusterRoles, RoleBindings, ClusterRoleBindings) rather than local files.

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

## Network Policies
**Quick explanation**: A `NetworkPolicy` is a Kubernetes resource that controls which pods are allowed to send or receive traffic. It adds a security layer at the network level so you can restrict communication between workloads instead of allowing all traffic by default.

**What is a NetworkPolicy?**
A `NetworkPolicy` is a rule set applied to a group of pods. It decides:
- which incoming traffic is allowed to reach them
- which outgoing traffic is allowed from them
- what namespaces or pods can communicate with them

It is commonly used to enforce the principle of least privilege inside a cluster.

**Why it matters**:
- restrict traffic between microservices
- protect databases from unexpected access
- isolate dev, test, and prod workloads
- reduce lateral movement inside the cluster
- work well with labels, namespaces, and services

**Important default behavior**:
- If a pod is not selected by any `NetworkPolicy`, Kubernetes usually allows all traffic to and from it.
- Once a pod is selected by a `NetworkPolicy`, the rules in that policy apply.
- If you define an ingress rule, then only matching incoming traffic is allowed; other ingress is denied.
- If you define an egress rule, then only matching outgoing traffic is allowed; other egress is denied.

This means network policy is a way to create allow-lists instead of open networking.

---

### How it works with other Kubernetes components
`NetworkPolicy` works best when combined with labels, namespaces, and services:

- **Labels** identify which pods belong to a service or application
- **Namespaces** let you allow traffic only from a specific team or environment
- **Services** route traffic to selected pods
- **Ingress controllers** can be allowed explicitly using namespace and pod selectors
- **External IPs** can be allowed with `ipBlock` rules

**Real-world pattern**:
1. `frontend` pods receive traffic from users or ingress controller
2. `api` pods receive traffic only from `frontend`
3. `db` pods allow traffic only from `api`
4. `db` blocks all other sources

This gives a clean request path:
- Users -> Frontend
- Frontend -> API
- API -> Database
- Database does not allow random pod access

---

### Example: frontend, api, and db pods
Assume these pod labels:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
  labels:
    app: frontend
spec:
  containers:
    - name: frontend
      image: nginx
---
apiVersion: v1
kind: Pod
metadata:
  name: api
  labels:
    app: api
spec:
  containers:
    - name: api
      image: nginx
---
apiVersion: v1
kind: Pod
metadata:
  name: db
  labels:
    app: db
spec:
  containers:
    - name: db
      image: postgres
```

Now we apply policies to restrict communication.

#### 1. Allow frontend to reach the api
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-api
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend
      ports:
        - protocol: TCP
          port: 8080
```

This means:
- only pods labeled `app=frontend` can connect to `app=api`
- only port `8080` is allowed
- other sources are denied

#### 2. Allow api to reach the db
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-api-to-db
spec:
  podSelector:
    matchLabels:
      app: db
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: api
      ports:
        - protocol: TCP
          port: 5432
```

This means:
- the database accepts traffic only from `api` pods
- `db` does not accept direct traffic from `frontend`
- the database is protected by a narrow allow rule

#### 3. Allow API to egress to DB
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-egress-to-db
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
    - Egress
  egress:
    - to:
        - podSelector:
            matchLabels:
              app: db
      ports:
        - protocol: TCP
          port: 5432
```

This tells the `api` pod:
- it may send traffic to the `db` pod
- other outbound traffic is blocked

**Example flow**:
- `frontend` -> `api` allowed
- `api` -> `db` allowed
- `frontend` -> `db` denied
- any other pod -> `db` denied

---

### Ingress and Egress
**Ingress** is traffic coming into a pod or a set of pods.

```yaml
ingress:
  - from:
      - podSelector:
          matchLabels:
            app: frontend
    ports:
      - protocol: TCP
        port: 80
```

This means inbound connections to the selected pod are allowed only when they come from pods labeled `app=frontend` on TCP port `80`.

**Egress** is traffic leaving a pod.

```yaml
egress:
  - to:
      - podSelector:
          matchLabels:
            app: db
    ports:
      - protocol: TCP
        port: 5432
```

This means the selected pod may send traffic only to `app=db` on TCP port `5432`.

**How ingress and egress work together**:
- Ingress controls who can connect to a pod.
- Egress controls where a pod is allowed to connect.
- A pod can have only ingress rules, only egress rules, or both.
- The intent is to control both directions of communication for security.

**Simple rule**:
- Ingress = traffic entering the pod
- Egress = traffic leaving the pod

---

### Example: a restricted policy with DNS egress and specific app access
This example is commonly used when a pod must talk to a small set of internal services while still being allowed to resolve names over DNS.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: internal-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      name: internal
  policyTypes:
  - Egress
  - Ingress
  ingress:
    - {}
  egress:
    - to:
      - podSelector:
          matchLabels:
            name: mysql
      ports:
        - protocol: TCP
          port: 3306
    - to:
      - podSelector:
          matchLabels:
            name: payroll
      ports:
        - protocol: TCP
          port: 8080
    - ports:
      - protocol: TCP
        port: 53
      - protocol: UDP
        port: 53
```

**What each part means**:

- `metadata.name: internal-policy`
  - This is the name of the NetworkPolicy object.

- `metadata.namespace: default`
  - The policy is applied inside the `default` namespace.

- `spec.podSelector.matchLabels.name: internal`
  - This selects the pods that this policy applies to.
  - Only pods labeled `name=internal` are affected.
  - If a pod is not labeled `internal`, this policy does not apply to it.

- `spec.policyTypes:`
  - `Egress` means outbound traffic from the selected pods is controlled.
  - `Ingress` means inbound traffic to the selected pods is controlled.
  - Because both are listed, this policy governs both directions.

- `ingress:
    - {}`
  - This is a special allow-all ingress rule.
  - The empty object `{}` means: allow inbound traffic from any source, on any port, to the selected pods.
  - In other words, the selected pods accept all incoming traffic.
  - This is intentionally broad and is usually only used for trusted internal apps or for a short-lived testing setup.
  - If you wanted to restrict ingress, you would replace `{}` with explicit `from:` rules such as podSelector, namespaceSelector, or ipBlock.

- `egress:`
  - This is a list of allowed outbound rules. Any outbound traffic not matching one of these rules is denied.

- Rule 1: MySQL access
  ```yaml
  - to:
    - podSelector:
        matchLabels:
          name: mysql
    ports:
      - protocol: TCP
        port: 3306
  ```
  - This allows the selected pods to connect only to pods labeled `name=mysql`.
  - The protocol is TCP and the allowed port is `3306`.
  - This is a narrow, least-privilege rule for the database port.

- Rule 2: Payroll service access
  ```yaml
  - to:
    - podSelector:
        matchLabels:
          name: payroll
    ports:
      - protocol: TCP
        port: 8080
  ```
  - This allows traffic only to pods labeled `name=payroll`.
  - Only TCP port `8080` is allowed.
  - This keeps the app from reaching unrelated services on arbitrary ports.

- Rule 3: DNS access by port only
  ```yaml
  - ports:
      - protocol: TCP
        port: 53
      - protocol: UDP
        port: 53
  ```
  - This is the important part: there is no `to:` field here.
  - That means this rule applies to all destinations, not just a podSelector, namespaceSelector, or ipBlock.
  - It allows DNS traffic on TCP/53 and UDP/53 to any destination that resolves the query.
  - This is commonly used for cluster DNS lookups, because applications need to resolve service names such as `mysql`, `payroll`, or external hostnames.

**Why the final DNS rule is different**:
- `podSelector`, `namespaceSelector`, and `ipBlock` are used to restrict who or where traffic is allowed to go.
- In the DNS rule, there is no selector or CIDR block, so the rule is not limited to a specific pod, namespace, or IP range.
- It is instead restricted only by `port` and `protocol`.
- This makes it a port-based exception: the pod can send DNS traffic anywhere, but only on port `53`.
- This is useful because DNS requests are usually sent to the cluster DNS service or a configured upstream resolver, and the actual IP target may be dynamic or managed by Kubernetes.

**Important note about `ingress: - {}`**:
- `ingress: - {}` is equivalent to saying: allow all inbound traffic.
- It does not use a `from:` selector and therefore does not restrict source pods, source namespaces, or external CIDR ranges.
- This is broader than the egress rules and is often the opposite of a locked-down network policy.

**What this pattern gives you**:
- `internal` pods are allowed to reach MySQL on port `3306`.
- `internal` pods are allowed to reach payroll on port `8080`.
- `internal` pods are allowed to do DNS lookups on port `53`.
- Inbound traffic is open to all sources.

**When to use this pattern**:
- During development or testing when you need a quick allowlist for specific services and DNS.
- In environments where only the outbound connection paths matter and ingress is intentionally left open.
- As a starting point before tightening ingress with more specific `from:` rules.

**Security warning**:
- The rule `ingress: - {}` is very permissive and should not be used for production workloads unless you truly want unrestricted inbound access.
- A stronger production pattern is to restrict ingress using `podSelector`, `namespaceSelector`, and `ipBlock` so only trusted sources can connect.

This is the clearest example of how a NetworkPolicy can combine:
- a pod label selector for the target workload
- port-specific access for service traffic
- a port-only egress exception for DNS
- and a broad ingress rule when intentionally open

---

### Pod selector and namespace selector
A `NetworkPolicy` usually selects pods using a `podSelector`.

#### Pod selector
```yaml
podSelector:
  matchLabels:
    app: api
```

This means the policy applies to the pods labeled `app=api`.

#### Namespace selector
A namespace selector lets you match by namespace labels instead of just pod labels.

```yaml
from:
  - namespaceSelector:
      matchLabels:
        team: platform
    podSelector:
      matchLabels:
        app: ingress
```

This means:
- allow traffic only from pods in namespaces labeled `team=platform`
- and only from pods labeled `app=ingress`

This is useful when:
- a frontend in one namespace talks to a backend in another namespace
- an ingress controller runs in a dedicated namespace
- you want to allow traffic from an isolated platform namespace only

**Common pattern**: combine `namespaceSelector` and `podSelector` to allow only a specific service in a trusted namespace.

---

### Example with another server outside the cluster
Sometimes an external system such as a VPN server, bastion host, or on-prem app needs to reach a pod directly. You can allow it using `ipBlock`.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-external-admin
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
    - Ingress
  ingress:
    - from:
        - ipBlock:
            cidr: 203.0.113.40/32
      ports:
        - protocol: TCP
          port: 8080
```

This allows only the external IP `203.0.113.40/32` to reach the `api` pod on port `8080`.

**Example with an ingress controller namespace**:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-ingress-controller
spec:
  podSelector:
    matchLabels:
      app: frontend
  policyTypes:
    - Ingress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: ingress-nginx
          podSelector:
            matchLabels:
              app.kubernetes.io/name: ingress-nginx
      ports:
        - protocol: TCP
          port: 80
```

This allows traffic from the ingress controller namespace to the frontend pod, which is the normal pattern when an ingress controller sits in front of application services.

**Important note**:
- `ipBlock` is used for external IP ranges, not Kubernetes label matching.
- `namespaceSelector` and `podSelector` are used for in-cluster traffic.
- If you expose a service using `LoadBalancer`, `NodePort`, or `Ingress`, the source IP seen by the pod may be the load balancer or ingress controller, not the original client.

---

### Summary
A `NetworkPolicy` is the Kubernetes way to control network traffic between workloads. It works by selecting pods with labels and then defining allow rules for ingress and egress. When combined with services, namespaces, and selectors, it creates a clear trust model like:
- users and ingress reach frontend
- frontend reaches api
- api reaches db
- db is not reachable from other unauthorized pods

This is a key building block for secure multi-tier applications in Kubernetes.

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