# Lesson 3 Deploying Kubernetes Applications

- [Lesson 3 Deploying Kubernetes Applications](#lesson-3-deploying-kubernetes-applications)
    - [3.1 Using Deployments](#31-using-deployments)
      - [Using Deployments](#using-deployments)
    - [3.2 Running Agents with DaemonSets](#32-running-agents-with-daemonsets)
      - [Understanding DaemonSets](#understanding-daemonsets)
    - [3.3 Using StatefulSets](#33-using-statefulsets)
      - [Understanding Stateful and Stateless Applications](#understanding-stateful-and-stateless-applications)
      - [Understanding StatefulSet](#understanding-statefulset)
      - [When to Use a StatefulSet](#when-to-use-a-statefulset)
      - [StatefulSet Considerations](#statefulset-considerations)
    - [3.4 The Case for Running Individual Pods](#34-the-case-for-running-individual-pods)
      - [Running Individual Pods](#running-individual-pods)
    - [3.5 Managing Pod Initialization](#35-managing-pod-initialization)
      - [Using Init Containers](#using-init-containers)
    - [3.6 Scaling Applications](#36-scaling-applications)
      - [Scaling Applications](#scaling-applications)
    - [3.7 Using Sidecar Containers for Application Logging](#37-using-sidecar-containers-for-application-logging)
      - [Understanding Multi-container Pods](#understanding-multi-container-pods)
      - [Understanding Mulit-container Storage](#understanding-mulit-container-storage)
    - [Lesson 3 Lab Running a DaemonSet](#lesson-3-lab-running-a-daemonset)
    - [Lesson 3 Lab Solution Running a DaemonSet](#lesson-3-lab-solution-running-a-daemonset)

### 3.1 Using Deployments

#### Using Deployments

- The Deployment is the standard way for running containers in Kubernetes.
- Deployments are responsible for starting Pods in a scalable way.
- The Deployment resource uses a ReplicaSet to manage scalability.
- Also, the Deployment offers the RollingUpdate feature to allow for zero-downtime application updates.
- To start a Deployment the imperative way, use **kubectl create deploy ...**

```bash
kubectl create deploy -h | less
kubectl create deployment firstnginx --image=nginx --replicas=3
kubectl get all
```

### 3.2 Running Agents with DaemonSets

#### Understanding DaemonSets

- A DaemonSet is a resource that starts one application instance on each cluster node.
- It is commonly used to start agents like the kube-proxy that need to be running on all cluster nodes.
- It can also be used for user workloads.
- If the DaemonSet needs to run on control-plane nodes, a teleration must be configured to allow the node to run regardless of the control-plane taints.

```bash
kubectl get ds
kubectl get ds -A
kubectl get ds -n kube-system calico-node -o yaml | less
```

- Open - **https://kubernetes.io/docs/**
- Search for **DaemonSet**
- Click **DaemonSet**
- Copy the sample code of **Create a DaemonSet**
- Create a daemonset.yaml and add the below YAML content into it.

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      tolerations:
      # these tolerations are to have the daemonset runnable on control plane nodes
      # remove them if your control plane nodes should not run pods
      - key: node-role.kubernetes.io/control-plane
        operator: Exists
        effect: NoSchedule
      - key: node-role.kubernetes.io/master
        operator: Exists
        effect: NoSchedule
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
```
```bash
kubectl create deployment mydaemon --image=nginx --dry-run=client -o yaml > mydaemon.yaml
vi mydaemon.yaml
# 1. Change kind: Deployment to DaemonSet
# 2. Delete replicas: 1
# 3. Delete stratege: {}
# 4. Save and close the file.

# Create a DaemonSet
kubectl apply -f mydaemon.yaml
kubectl get ds
kubectl get pods -o wide
```

### 3.3 Using StatefulSets

#### Understanding Stateful and Stateless Applications

- A stateless application is an application that doesn't store any session data.
- Redirecting traffic in a stateless application is easy, the traffic can just be directed to another Pod instance.
- A stateful application saves session data to persistent storage.
- Databases are an example of stateful applications.
- Even if stateful applications can be started by a Deployment, it's better to start it in a StatefulSet.

#### Understanding StatefulSet

A StatefulSet offers features that are needed by stateful applications.
- It provides guarantees about ordering and uniqueness of Pods.
- It maintains a sticky identifier for each of the Pods it creates.
- Pods in a StatefulSet are not interchangeable: each Pod has a persistent identifier that it maintains while being rescheduled.
- The unique Pod identifiers make it easier to match existing volumes to replaced Pods.

#### When to Use a StatefulSet

- StatefulSet is used for applications that require one or more of the following.
  - Stable and unique network identifiers.
  - Stable persistent storage.
  - Ordered, graceful deployment and scaling.
  - Ordered and automated rolling update.
- If none of these is needed, Deployment should be used.

#### StatefulSet Considerations

- Storage must be automatically provisioned by a persistent volume provisioner. Pre-provisioning is challenging, as volumes need to be dynamically added when new Pods are scheduled.
- When a StatefulSet is deleted, associated volumes will not be deleted.
- A headless Service resource must be created in order to manage the network identity of Pods.
- Pods are not guaranteed to be stopped while deleting a StatefulSet, and it recommended to scale down to zero Pods before deleting the StatefulSet.
```bash
# Do it in the Minikube
cd labs/
vi statefuldemo.yaml
kubectl apply -f statefuldemo.yaml
kbuectl get statefulset
kubectl get pods
kubectl get pvc
```

### 3.4 The Case for Running Individual Pods

#### Running Individual Pods

- Running individul Pods has disadvantages.
  - No workload protection.
  - No load balancing.
  - No zero-downtime application update.
- Use individual Pods only for testing, troubleshooting, and analyzing.
- In all other cases, use Deployment, DaemonSet or StatefulSet.

```bash
kubectl run -h | less
kubectl run sleepy --image=busybox -- sleep 3600
kubectl get pods
```

### 3.5 Managing Pod Initialization

#### Using Init Containers

- If preparation is required before running the main container, use an init container.
- Init containers run to completion, and once completed the main container can be started.
- Use init containers in any case where preliminary set up is required.

```bash
# Open - **https://kubernetes.io/docs/**
# Search for **init container**
# Click **Configure Pod Initialization**
# Copy the sample code of **Create a Pod that has an Init Container** and paste into init-container file.
cd labs/tasks
vi init-container.yaml
kubectl apply -f init-container.yaml
kubectl get all
```
- init-container.yaml file.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-container-demo
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
  # These containers are run during pod initialization
  initContainers:
  - name: install
    image: busybox
    command:
    - sleep
    - "30"
```

### 3.6 Scaling Applications

#### Scaling Applications

- **kubectl scale** is used to manually scale Deployment, ReplicaSet or StatefulSet.
  - **kubectl scale deployment myapp --replicas=3**
- Alternatively, HorizontalPodAutoscaler can be used.
```bash
kubectl get deploy
kubectl scale deployment firstnginx --replicas=5
kubectl get deploy
kubectl get all --selector app=firstnginx
```

### 3.7 Using Sidecar Containers for Application Logging

#### Understanding Multi-container Pods

- As a Pod should be created for each specific task, running single-container Pods is the standard.
- In some cases, an additional container is needed to modify or present data generated by the main container.
- Specific use cases are defined.
  - **Sidecar**: provides additional functionality to the main container.
  - **Ambassador**: is used as a proxy to connect containers externally.
  - **Adapter**: is used to standardize or normalize main container output.

#### Understanding Mulit-container Storage

- In a multi-container Pod, Pod volumes are often used as shared storage.
- The Pod Volume may use PersistentVolumeClaim (PVC) to refer to a PersistentVolume, but may also directly refer to the required storage.
- By using shared storage, the main container can write to it, and the helper Pod will pick up information written to the shared storage.

```bash
vi labs/sidecarlog.yaml
kubectl apply -f labs/sidecarlog.yaml
kubectl exec -it two-containers -c nginx-container -- cat /usr/share/nginx/html/index.html
```

### Lesson 3 Lab Running a DaemonSet

- Create a DaemonSet with the name nginxdaemon.
- Ensure it runs an Nginx Pod on every worker node.

### Lesson 3 Lab Solution Running a DaemonSet

```bash
kubectl create deployment deploydaemon --image=nginx --dry-run=client -o yaml > deploydaemon.yaml
vim deploydaemon.yaml
# 1. Change kind: Deployment to DaemonSet
# 2. Delete replicas: 1
# 3. Delete stratege: {}
# 4. Save and close the file.
kubectl apply -f deploydaemon.yaml
kubectl get all --selector app=deploydaemon
```
