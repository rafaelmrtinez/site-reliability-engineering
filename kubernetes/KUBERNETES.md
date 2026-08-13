Certified Kubernetes Application Developer: https://www.cncf.io/certification/ckad/
Candidate Handbook: https://www.cncf.io/certification/candidate-handbook
Exam Tips: https://docs.linuxfoundation.org/tc-docs/certification/tips-cka-and-ckad

Node - A machine which is a physical or virtual. Also called minions in the past.
Cluster - Set of nodes grouped together.
Master Node - Responsible for the orchestration of nodes and containers
Api Server - Frontend for Kubernetes.
Etcd - A distrubted key-value store to manage the cluster.
Scheduler - Distributing work of contaienrs and assigns them to nodes. Reponsible for noticing and responding when containers goes down.
Container Runtime - Underlying software that runs containers.
Kubelet - Agent that runs in the nodes to manage containers.

Worker nodes requires container runtime. Master runs the 4 components (etcd, scheduler, controller, and api server).

Kubectl (Kube command line tool or Kube control) - Used to manage nodes/pods in a cluster.

```bash
# Examples:
kubectl run hello-minkube
kubectl cluster-info
kubectl get nods
```

## Containerd vs Docker
Docker - Container management tool which also contains docker engine as its runtime.
Containerd - Runtime that is being used by Docker.

Docker --> Docker Enginer --> Dockerd --> Containerd --> runc

Contaienrd is part of Docker but also a separate runtime for containers. Ideally, you can install containerd without docker and still be able to run containers.

```bash
Examples:
ctr
ctr images pull docker.io/<url>
```

## Nerdctl
Used to have almost the same commands with docker cli.

```bash
# Examples:
nerdctl
nerdctl run --name <image_name>
```

## Crictl
Used to interact with the cri compatible runtime. Maintained by the kubernetes community. This however is not ideal to create containers. Only to be used for debugging purposes.

```bash
crictl
crictl pull busybox
crictl images
crictl ps -a
crictl exec -it -t <container_id> <command_to_be_run>
crictl logs <container_id>
crictl pods
```
## Docker deprecation

- Containerd itself is sufficient to run containers so docker is no longer needed but NOT ENTIRELY GONE. Only for Kubernetes, docker is no longer used.

## Pods
- Containers are encapsulter in a object called Pods.
- Smalles object in Kubernetes.
- Ideally only one container must run inside a single pod. 
- Can be deployed in different nodes
- Has one-to-one relationship with the container.
- Do not add additional containers in a pod to scale.

#### Note: We can have multiple containers in a pod ONLY IF they are not the same applications.

```yaml
Example:
- Pod
  - Python App
  - Dotnet Runtime (Helper container)
```

Helper containers only exist if another container requires dependency on it! That means, if the main container dies, the helper also dies.

However, this type of architecture is only **rarely used**.

## Commands
``` bash
kubectl run nginx --image nginx
kubectl get pods
```

## YAML Base Configuration in Kubernetes

```yaml
apiVersion: v1
# Kind can be Pod, Service, ReplicaSet, or Deployment
kind: Pod 
# Metabdata is a data about the object
metadata:
    name: myapp-pod
    labels:
        app: myapp
        type: frontend

# Specification contents can differ across different services/configurations
spec:
    containers:
        - name: nginx-container
          image: nginx
```

Once the configuration is created we can run the command.

```bash
kubectl create -f <file_name>.yml

# Get pods
kubectl get pods

# Details about the pod
kubectl describe pod myapp-pod
```

## Replication Controller
- High availability
- Creates multiple pod of the same instance
- Can replace dead or unhealthy pods automatically
- Can be used to share a load between them by re-routing traffic
- Older technology that was replaced by ReplicaSet

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
    spec:
      containers:
        - name: myapp-container
          image: nginx
          ports:
            - containerPort: 80
```

```bash
kubectl create -f rc.yaml
kubectl get rc
kubectl get pods
kubectl describe rc myapp-rc
```

**Create, add, update, and delete ReplicaController**
```bash
# Create a replica controller from a YAML file
kubectl create -f rc.yaml

# Apply or update the controller definition
kubectl apply -f rc.yaml

# Increase or decrease replica count
kubectl scale rc myapp-rc --replicas=5

# Delete the controller and all managed pods
kubectl delete rc myapp-rc
```

#### Note: A ReplicationController ensures the cluster always maintains the desired number of pod replicas. It does not explicitly declare a selector in the manifest because the pod template labels are used for matching.

#### Note: `kubectl scale` is commonly used to change the replica count, while `kubectl delete rc <name>` removes the controller and all the pods it manages.

## ReplicaSet
- Modern replacement for ReplicationController
- Ensures a set number of identical pods are running
- Requires a selector to identify which pods it manages
- Uses label selectors and supports more flexible matching
- Commonly used as a lower-level building block for Deployment

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
kubectl create -f rs.yaml
kubectl get rs
kubectl get pods
kubectl describe rs myapp-rs
```

**Create, scale, update, and delete ReplicaSet**
```bash
# Create a ReplicaSet from YAML
kubectl create -f rs.yaml

# Apply updates to the ReplicaSet definition
kubectl apply -f rs.yaml

# Scale to a specific replica count
kubectl scale rs myapp-rs --replicas=5

# Scale up from current value to a higher count
kubectl scale rs myapp-rs --replicas=10

# Scale down to a smaller count
kubectl scale rs myapp-rs --replicas=2

# Update the replica count directly in the YAML and re-apply
kubectl apply -f rs.yaml

# Delete the ReplicaSet and its managed pods
kubectl delete rs myapp-rs
```

#### Note: ReplicaSet must define a selector. This selector tells Kubernetes which pods belong to the ReplicaSet and must match the labels in the pod template.

#### Note: You can scale a ReplicaSet either with the `kubectl scale` command or by modifying the `replicas` field in the YAML and applying it again.

## ReplicaController vs ReplicaSet

**ReplicaController**
- Older Kubernetes object
- Does not explicitly declare a selector in the manifest
- Relies on the template labels for pod matching
- Replaced by ReplicaSet in modern workloads
- Good for basic pod replication

**ReplicaSet**
- Newer and preferred for pod replication
- Must define a selector using `matchLabels` or a more advanced selector rule
- Uses a more robust selector model
- Acts as the underlying primitive for Deployments
- More commonly used in modern Kubernetes environments

**Example comparison**
```bash
# ReplicationController
kubectl create -f rc.yaml
kubectl scale rc myapp-rc --replicas=3
kubectl delete rc myapp-rc

# ReplicaSet
kubectl create -f rs.yaml
kubectl scale rs myapp-rs --replicas=3
kubectl delete rs myapp-rs
```

#### Note: In modern Kubernetes, Deployment is usually preferred over both ReplicationController and ReplicaSet because it adds rolling updates and rollback support.

#### Note: In real-world Kubernetes usage, Deployment is usually preferred over both ReplicationController and ReplicaSet because it adds rolling updates and rollback support.