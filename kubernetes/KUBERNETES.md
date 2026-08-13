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
  - [YAML Base Configuration in Kubernetes](#yaml-base-configuration-in-kubernetes)
  - [Replication Controller](#replication-controller)
  - [ReplicaSet](#replicaset)
  - [ReplicaController vs ReplicaSet](#replicacontroller-vs-replicaset)
    - [ReplicaController](#replicacontroller)
    - [ReplicaSet](#replicaset-1)
  - [Rolling Updates](#rolling-updates)
  - [Deployment (Modern Recommendation)](#deployment-modern-recommendation)
  - [Final Notes](#final-notes)

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