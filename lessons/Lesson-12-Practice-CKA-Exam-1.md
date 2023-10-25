# Lesson 12 Practice CKA Exam 1

- [Lesson 12 Practice CKA Exam 1](#lesson-12-practice-cka-exam-1)
    - [12.1 Question Overview](#121-question-overview)
      - [Question 1: Creating a Kubernetes Cluster](#question-1-creating-a-kubernetes-cluster)
      - [Question 2: Scheduling a Pod](#question-2-scheduling-a-pod)
      - [Question 3: Managing Application Initialization](#question-3-managing-application-initialization)
      - [Question 4: Setting up Persistent Storage](#question-4-setting-up-persistent-storage)
      - [Question 5: Configuring Application Access](#question-5-configuring-application-access)
      - [Question 6: Securing Network Traffic](#question-6-securing-network-traffic)
      - [Question 7: Setting up Quota](#question-7-setting-up-quota)
      - [Question 8: Creating a Static Pod](#question-8-creating-a-static-pod)
      - [Question 9: Troubleshooting Node Services](#question-9-troubleshooting-node-services)
      - [Question 10: Configuring Cluster Access](#question-10-configuring-cluster-access)
      - [Question 11: Configuring Taints and Tolerations](#question-11-configuring-taints-and-tolerations)
    - [12.2 Creating a Kubernetes Cluster](#122-creating-a-kubernetes-cluster)
      - [Nodes](#nodes)
      - [kubeadm init Setup Procudure](#kubeadm-init-setup-procudure)
    - [12.3 Scheduling a Pod](#123-scheduling-a-pod)
    - [12.4 Managing Application Initialization](#124-managing-application-initialization)
    - [12.5 Setting up Persistent Storage](#125-setting-up-persistent-storage)
    - [12.6 Configuring Application Access](#126-configuring-application-access)
    - [12.7 Securing Network Traffic](#127-securing-network-traffic)
    - [12.8 Setting up Quota](#128-setting-up-quota)
    - [12.9 Creating a Static Pod](#129-creating-a-static-pod)
    - [12.10 Troubleshooting Node Services](#1210-troubleshooting-node-services)
    - [12.11 Configuring Cluster Access](#1211-configuring-cluster-access)
    - [12.12 Configuring Taints and Tolerations](#1212-configuring-taints-and-tolerations)

### 12.1 Question Overview

#### Question 1: Creating a Kubernetes Cluster

- Create a 3-node Kubernetes cluster, using one control plane node and 2 worker nodes.
- Use the scripts provided in the course Git repository.

#### Question 2: Scheduling a Pod

- Schedule a Pod with the name lab123 that runs the Nginx and redis applications.

#### Question 3: Managing Application Initialization

- Create a deployment with the name lab1234deploy which runs the Nginx image, but waits 30 seconds before starting the actual Pods.

#### Question 4: Setting up Persistent Storage

- Create a Persistent Volume with the name lab123 that uses HostPath on the directory /lab125.

#### Question 5: Configuring Application Access

- Create a Deployment with the name lab126deploy, running 3 instances of the Nginx image.
- Configure it such that it can be accessed by external users on port 32567 on each cluster node.

#### Question 6: Securing Network Traffic

- Create a Namespace with the name restricted, and configure it such that it only allows access to Pods exposing port 80 for Pods coming from the Namespaces access.

#### Question 7: Setting up Quota

- Create a Namespace with the name limited and configure it such that only 5 Pods can be started and the total amount of available memory for applications running in that Namespace is limited to 2 GiB.
- Run a webserver Deployment with the name lab128deploy and using 3 Pods in this Namespace.
- Each of the Pods should request 128MiB memory and be limited to 256MiB.

#### Question 8: Creating a Static Pod

- Configure a Pod with the name lab129pod that will be started by the kubelet on node worker2 as a static Pod.

#### Question 9: Troubleshooting Node Services

- Assume that node worker2 is not currently available. Ensure that the appropriate service is started on that node which will show the node as running.

#### Question 10: Configuring Cluster Access

- Create a ServiceAccount that has permissions to create Pods, Deployments, DaemonSets and StatefulSets in the Namespace "access". 

#### Question 11: Configuring Taints and Tolerations

- Configure node worker2 such that it will noly allow Pods to run that have been configured with the setting type:db
- After verifying this works, remove the node restriction to return to normal operation.

### 12.2 Creating a Kubernetes Cluster

- Create a 3-node Kubernetes cluster, using one control plane node and 2 worker nodes.
- Use the scripts provided in the course Git repository.

#### Nodes

|ID | HOSTNAME              | IP ADDRESS    |
|---| --------------------- | ------------- |
| 1 | control-plane.lab.com | 192.168.1.200 |
| 2 | worker-node-1.lab.com | 192.168.1.201 |
| 3 | worker-node-2.lab.com | 192.168.1.202 |
|

```bash
#### Setup static IP address and hostname

- Set up the static IP address and hostname in all nodes as per the table above.

#### Setup hostname resolution

```bash
sudo vi /etc/hosts # Add below content

192.168.1.200 control-plane.lab.com control-plane
192.168.1.201 worker-node-1.lab.com worker-node-1
192.168.1.202 worker-node-2.lab.com worker-node-2
```

#### kubeadm init Setup Procudure

```bash
cd labs/

# Install CRI
sudo ./setup-container.sh # Execute it on all servers.

# Install kubetools
sudo ./setup-kubetools.sh # Execute it on all servers.

# Install the Cluster
sudo kubeadm init # Execute it only on the control-plane

# Copy the following three lines on the terminal and paste them.
mkdir ~/.kube
sudo cp -i /etc/kubernetes/admin.conf ~/.kube/config
sudo chown $(id -u):$(id -g) ~/.kube/config

# Verify the cluster setup.
kubectl get all

# Set up calico network.
# kubectl apply -f labs/calico.yaml
# kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.0/manifests/calico.yaml
sh labs/setup-calico.sh
kubectl get all
kubectl get pods -n kube-system

# Join other nodes (execute on the worker nodes)
sudo kubeadm join <join-token> # Copy the kubeadm join complete command from the control-plane.

# Jone worker-node-1 to the cluster
ssh anil@worker-node-1
sudo kubeadm join <join-token>

# Jone worker-node-2 to the cluster
ssh anil@worker-node-2
sudo kubeadm join <join-token>

# Verify the cluster nodes
ssh anil@control-plane
kubectl get nodes
```

### 12.3 Scheduling a Pod

- Schedule a Pod with the name lab123 that runs the Nginx and redis applications.

```bash
kubectl run lab123 --image=nginx --dry-run=client -o yaml > lab123.yaml
vi lab123.yaml
# add redis image
spec.containers under resources: {}
- image: redis
  name: redis

kubectl apply -f lab123.yaml
kubectl get pods
```

### 12.4 Managing Application Initialization

- Create a deployment with the name lab1234deploy which runs the Nginx image, but waits 30 seconds before starting the actual Pods.

```bash
# Search for **initpod** in the documentation
# Copy spec.containers 
kubectl create deployment lab1234deploy --image=busybox --dry-run=client -o yaml -- sleep 30 > lab1234deploy.yaml
vi lab1234deploy.yaml
# 1. Add below content under containers:
    - image: nginx
      name: nginx
    initContainers:
# 2. Delete resources: {}

kubectl apply -f lab1234deploy.yaml
kubectl get pods # It will take 30 seconds to initialize the Pod.
```

### 12.5 Setting up Persistent Storage

- Create a Persistent Volume with the name lab123 that uses HostPath on the directory /lab125.


```yaml
# Search for **persistentvolue** in the documentation
# Click **Configure a Pod to Use a PersistentVolume for Storage | Kubernetes**
# Copy **Create a PersistentVolume** example code. Modify it as per the requirement.
# vi lab125.yaml

apiVersion: v1
kind: PersistentVolume
metadata:
  name: lab125
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/lab123"
```
```bash
kubectl apply -f lab125.yaml
kubectl get pv
kubectl describe pv lab125 # Status should be : Available
```

### 12.6 Configuring Application Access

- Create a Deployment with the name lab126deploy, running 3 instances of the Nginx image.
- Configure it such that it can be accessed by external users on port 32567 on each cluster node.

```bash
kubectl create deployment lab126deploy --image=nginx --replicas=3
kubectl expose deploy lab126deploy --port=80
kubectl get all --selector app=lab126deploy
kubectl explain service.spec.ports
kubectl edit svc lab126deploy
# 1. Add spec.ports.nodePort: 32567 
# Change spec.type: to NodePort
kubectl get all --selector app=lab126deploy
curl node-port:32567
```

### 12.7 Securing Network Traffic

- Create a Namespace with the name restricted, and configure it such that it only allows access to Pods exposing port 80 for Pods coming from the Namespaces access.

```bash
# Create a namespaces
kubectl create ns restricted
kubectl create ns access

# Create a pod in restricted namespace.
kubectl run testnginx --image=nginx -n restricted

# Create a pod in access namespace
kubectl run testbox --image=busybox -n access -- sleep 3600

# Create a pod in default namespace
kubectl run testbox --image=busybox -- sleep 3600

# Search for **network policy** in documentation.
# Click **Network Policies | Kubernetes**
# Copy the example code of **The NetworkPolicy resource** and modify it as per the requirement.
# 1. Change the namespace to restricted
# 2. Delete podSelector
# 3. Delete type: - Egress
# 4. Delete complete egress block of code.
# 5. Change the port to 80
# 6. Delete ipBlock
# 7. Comment the podSelector
# The final code looks like below.

vi lab127.yaml
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: restricted
spec:
  policyTypes:
    - Ingress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              project: myproject
        # - podSelector:
        #    matchLabels:
        #      role: frontend
      ports:
        - protocol: TCP
          port: 80
...

# Create a label for namespace access.
kubectl label ns access project=myproject
kubectl get ns --show-labels

# Create network policy
kubectl create -f lab127.yaml
describe -n restricted networkpolicies.networking.k8s.io

# Create a service in restricted namespace
kubectl get pods -n restricted
kubectl expose pod testnginx --port=80 -n restricted

kubectl get svc -n restricted # Copy the ClusterIP

# Access nginx website from the access namespace 
kubectl exec -it testbox -n access -- curl ClusterIP # If curl is not found, use wget.

# Access nginx website from the default namespace. It should fail.
kubectl exec -it testnginx-default -- curl ClusterIP
```

### 12.8 Setting up Quota

- Create a Namespace with the name limited and configure it such that only 5 Pods can be started and the total amount of available memory for applications running in that Namespace is limited to 2 GiB.
- Run a webserver Deployment with the name lab128deploy and using 3 Pods in this Namespace.
- Each of the Pods should request 128MiB memory and be limited to 256MiB.

```bash
# Create namespace: limited.
kubectl create ns limited

# Create quota in limited namespace: my-quota
kubectl create quota -h | less
kubectl create quota my-quota --hard=memory=2G,pods=5 -n limited
kubectl describe ns limited

# Create a deployment
kubectl create deployment lab128deploy --image=nginx --replicas=3 -n limited
kubectl get all -n limited

# Set resources
kubectl set resources -h | less
kubectl set resources deployment lab128deploy --limits=memory=256Mi --requests=memory=128Mi -n limited
kubectl get all -n limited 
kubectl describe ns limited
```

### 12.9 Creating a Static Pod

- Configure a Pod with the name lab129pod that will be started by the kubelet on node worker2 as a static Pod.

```bash
kubectl run lab129pod --image=nginx --dry-run=client -o yaml # Copy the yaml code.
ssh anil@worker-node2
cd /etc/kubernetes/manifests/
sudo vim lab129pod.yaml # Paste the yaml code into this file.

ssh anil@control-plane
kubectl get pods
```

### 12.10 Troubleshooting Node Services

- Assume that node worker2 is not currently available. Ensure that the appropriate service is started on that node which will show the node as running.

```bash
kubectl get nodes # Make sure all nodes are ready.  If any node is not ready restart the kubelet service on that node.
sudo systemctl status kubelet
sudo systemctl start kubelet
sudo systemctl restart kubelet
```

### 12.11 Configuring Cluster Access

- Create a ServiceAccount that has permissions to create Pods, Deployments, DaemonSets and StatefulSets in the Namespace "access". 

```bash
kubectl create ns access

# Search for **role** in the documentation
# Click **Using RBAC Authorization | Kubernetes**
# Look for "get", "list", "watch", "create", "update", "patch", "delete"
kubectl create role -h | less
kubectl create role appcreator \
  --verb=get \
  --verb=list \
  --verb=watch \
  --verb=create \
  --verb=update \
  --verb=patch \
  --verb=delete \
  --resource=pods,deployment,daemonset,statefulset -n access

kubectl get role -n access 
kubectl describe role -n access

# Create a service account.
kubectl create sa appcreator -n access
kubectl get sa -n access

# Create rolebinding.
kubectl create rolebinding -h | less
kubectl create rolebinding appcreator --role=appcreator --serviceaccount=access:appcreator -n access
kubectl get role,rolebinding,ServiceAccount -n access
```

### 12.12 Configuring Taints and Tolerations

- Configure node worker2 such that it will noly allow Pods to run that have been configured with the setting type:db
- After verifying this works, remove the node restriction to return to normal operation.

```bash
# Search for **taint** in documentation
# Click **Taints and Tolerations | Kubernetes**

# Create a taint.
kubectl taint nodes worker-node-1 type=db:NoSchedule

# Generate deployment manifest file 
kubectl create deployment toleratenginx --image=nginx --replicas=3 --dry-run=client -o yaml > lab1212.yaml

# Add tolerations spec.spec
vi lab1212.yaml

tolerations:
- key: "type"
  operator: "Equal"
  value: "db"
  effect: "NoSchedule"

# Create deployment.
kubectl create -f lab1212.yaml
kubectl get pods -o wide

# Create a test deployment, all pods should not be running on worker-node-1.
kubectl create deployment testdeploy --image=nginx --replicas=6
kubectl get all --selector app=testdeploy -o wide

# Delete resources
kubectl delete deploy testdeploy
kubectl taint nodes worker-node-1 type=db:NoSchedule-
kubectl delete -f lab1212.yaml
```
