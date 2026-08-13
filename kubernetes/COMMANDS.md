# Kubernetes kubectl Commands

## Table of Contents

- [Check cluster health](#check-cluster-health)
- [List resources](#list-resources)
- [Create and run workloads](#create-and-run-workloads)
- [Deploy from YAML](#deploy-from-yaml)
- [Inspect resources](#inspect-resources)
- [Read logs and execute commands](#read-logs-and-execute-commands)
- [Expose services locally](#expose-services-locally)
- [Scale and manage deployments](#scale-and-manage-deployments)
- [Check rollout and status](#check-rollout-and-status)
- [Manage context and namespace](#manage-context-and-namespace)
- [Delete resources](#delete-resources)

**Check cluster health**
```bash
kubectl cluster-info
kubectl get nodes
kubectl get pods -A
```

**List resources**
```bash
kubectl get pods
kubectl get deployments
kubectl get services
kubectl get svc -n kube-system
kubectl get all
```

#### Note: Use -A or --all-namespaces to inspect resources across all namespaces.

**Create and run workloads**
```bash
kubectl run nginx --image=nginx
kubectl run busybox --image=busybox --restart=Never -- sleep 3600
```

**Deploy from YAML**
```bash
kubectl apply -f deployment.yaml
kubectl create -f pod.yaml
kubectl apply -f service.yaml
```

#### Note: Prefer `kubectl apply` for managing Kubernetes objects because it updates existing resources instead of failing when they already exist.

**Inspect resources**
```bash
kubectl describe pod nginx
kubectl describe deployment nginx-deployment
kubectl get pod nginx -o wide
kubectl get pod nginx -o yaml
```

**Read logs and execute commands**
```bash
kubectl logs nginx
kubectl logs -f nginx
kubectl logs deployment/nginx
kubectl exec -it nginx -- /bin/sh
```

#### Note: `kubectl exec` lets you enter a running container for troubleshooting and quick checks.

**Expose services locally**
```bash
kubectl port-forward service/nginx 8080:80
kubectl port-forward pod/nginx 8080:80
kubectl get endpoints
```

**Scale and manage deployments**
```bash
kubectl scale deployment nginx --replicas=3
kubectl scale --replicas=5 deployment/nginx
kubectl rollout status deployment/nginx
```

**Check rollout and status**
```bash
kubectl rollout history deployment/nginx
kubectl rollout undo deployment/nginx
kubectl rollout pause deployment/nginx
kubectl rollout resume deployment/nginx
```

#### Note: Rollbacks are useful when a deployment introduces an issue and you need to restore the previous stable version.

**Manage context and namespace**
```bash
kubectl config get-contexts
kubectl config use-context my-cluster
kubectl config set-context --current --namespace=dev
kubectl get pods -n dev
```

**Delete resources**
```bash
kubectl delete pod nginx
kubectl delete deployment nginx
kubectl delete service nginx
kubectl delete -f deployment.yaml
```

#### Note: Use `kubectl delete` carefully in production, especially when removing deployments or services that are serving live traffic.

**View resource labels and selectors**
```bash
kubectl get pods --show-labels
kubectl get deployment -o wide --show-labels
kubectl get pods -l app=nginx
```

**Filter output with `-o` formats**
```bash
kubectl get pods -o wide
kubectl get pods -o json
kubectl get pods -o yaml
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase
```

#### Note: The `-o` option is useful when you want data in a compact or machine-readable format.

**Export a pod definition to YAML**
```bash
kubectl get pod nginx -o yaml > pod-definition.yaml
```

#### Note: This command saves the current live pod configuration to a YAML file, which is useful for backup, review, or creating a template for future deployments.

**Basic troubleshooting workflow**
```bash
kubectl get pods
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl exec -it <pod-name> -- /bin/sh
```

This is a common troubleshooting sequence: check if the pod is running, inspect its status and events, read logs, and open a shell into the container if needed.

**Example: create a deployment with a single command**
```bash
kubectl create deployment web --image=nginx --replicas=2
kubectl expose deployment web --port=80 --target-port=80 --type=LoadBalancer
kubectl get pods,svc
```

**Example: apply a YAML manifest**
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
        ports:
        - containerPort: 80
```

```bash
kubectl apply -f myapp.yaml
kubectl get deployment myapp
kubectl get pods -l app=myapp
```

**Example: check current namespace and switch to another one**
```bash
kubectl config view --minify | grep namespace
kubectl config set-context --current --namespace=default
kubectl get pods
```

#### Note: Namespaces help organize and isolate workloads in the same cluster.
