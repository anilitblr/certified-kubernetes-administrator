# Lesson 8 Managing Scheduling

- [Lesson 8 Managing Scheduling](#lesson-8-managing-scheduling)
    - [8.1 Exploring the Scheduling Process](#81-exploring-the-scheduling-process)
      - [Understanding Scheduling](#understanding-scheduling)
      - [From Scheduler to kubelet](#from-scheduler-to-kubelet)
    - [8.2 Setting Node Preferences](#82-setting-node-preferences)
      - [Setting Node Preferences](#setting-node-preferences)
      - [Demo: Using Node Preferences](#demo-using-node-preferences)
    - [8.3 Managing Affinity and Anti-Affinity Rules](#83-managing-affinity-and-anti-affinity-rules)
      - [Understanding Affinity and Anti-Affinity](#understanding-affinity-and-anti-affinity)
      - [How it Works](#how-it-works)
      - [Setting Node Affinity](#setting-node-affinity)
      - [Defining Affinity Labels](#defining-affinity-labels)
      - [Defining Affinity Labels](#defining-affinity-labels-1)
      - [Demo: Checking Out Some Examples](#demo-checking-out-some-examples)
      - [Understanding TopologyKey](#understanding-topologykey)
      - [Demo: Using Pod Anti-Affinity](#demo-using-pod-anti-affinity)
    - [8.4 Managing Taints and Tolerations](#84-managing-taints-and-tolerations)
      - [Understanding Taints](#understanding-taints)
      - [Understanding Tains Types](#understanding-tains-types)
      - [Setting Taints](#setting-taints)
      - [Understanding Tolerations](#understanding-tolerations)
      - [Understanding Taint Key and Value](#understanding-taint-key-and-value)
      - [Node Conditions and Taints](#node-conditions-and-taints)
      - [Demo: Using Taints](#demo-using-taints)
    - [8.5 Understanding LimitRange and Quota](#85-understanding-limitrange-and-quota)
      - [Understanding LimitRange](#understanding-limitrange)
      - [Understanding Quota](#understanding-quota)
    - [8.6 Configuring Resource Limits and Requests](#86-configuring-resource-limits-and-requests)
      - [Demo: Managing Quota](#demo-managing-quota)
    - [8.7 Configuring LimitRange](#87-configuring-limitrange)
      - [Demo: Defining Limitrange](#demo-defining-limitrange)
    - [Lesson 8 Lab Configuring Taints](#lesson-8-lab-configuring-taints)
    - [Lesson 8 Lab Solution Configuring Taints](#lesson-8-lab-solution-configuring-taints)

### 8.1 Exploring the Scheduling Process

#### Understanding Scheduling

- Kube-scheduling takes care of finding a nod eto schedule new Pods.
- Nodes are filtered according to specific requirements that may be set.
  - Resource requirements.
  - Affinity and anti-affinity.
  - Taints and tolerations and more.
- The scheduler first finds feasible nodes then scores them, it then picks the node with the highest score.
- Once this node is found, the scheduler notifies the API server in a process called binding.

#### From Scheduler to kubelet

- Once the scheduler decision has been make, it is picket up by the kubelet.
- The kubelet will instruct the CRI to fetch the iamge of the required container.
- After fetching the image, the container is created and started.

### 8.2 Setting Node Preferences

#### Setting Node Preferences

- The nodeSelector field in the pod.spec specifies a key-value pair that must match a babel which is set on nodes that are eligible to run the Pod.
- Use **kubectl label nodes work1.example.com disktype=ssd** to set the lable on a node.
- use **nodeSelector: disktype: ssd** in the pod.spec to match the Pod to the specific node.
- **nodeName** is part of the pod.spec and can be used to always run a Pod on a node with a specific name.
  - Not recommended: if that node is not currently available, the Pod will never run.

#### Demo: Using Node Preferences

```bash
kubectl lable nodes worker2 disktype=ssd
cd labs/
vim selector-pod.yaml
kubectl cordon worker2
kubectl apply -f selector-pod.yaml
kubectl get pods
kubectl describe pod nginx
kubectl uncordon worker2
kubectl get pods
```

### 8.3 Managing Affinity and Anti-Affinity Rules

#### Understanding Affinity and Anti-Affinity

- (Anti-)Affinity is used to define advanced scheduler rules.
- Node affinity is used to constrain a node that can receive a Pod by matching labels of these nodes.
- Inter-pod affinity constrains nodes to receive Pods by matching labels of existing Pods already running on that node.
- Anti-affinity can only be applied between Pods.

#### How it Works

- A Pod that has a node affinity label of key=value will only be scheduled to nodes with a matching label.
- A Pod that has a Pod affinity label of key=value will only be scheduled to nodes running Pods with the matching label.

#### Setting Node Affinity

- To define node affinity, two different statements can be used.
- **requiredDuringSchedulingIgnoredDuringExecution** requires the node to meet the constraint that is defined.
- **preferredDuringSchedulingIgnoredDuringExecution** defines a soft affinity that is ignored if it cannot be fulfilled.
- At the moment, affinity is only applied while scheduling Pods, and cannot be used to change where Pods are already running.

#### Defining Affinity Labels

- Affinity rules go beyond labels that use a key=value label.
- A matchexpression is used to define a **key** (the label), an **operator** as well as optionally one or more values.
```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: type
          operator: In
          values:
          - blue
          - green
``` 
- Matches any node that has **type** set to either **blue** or **green**

#### Defining Affinity Labels

```yaml
nodeSelectorTerms:
- matchExpressions:
  - key: storage
    operator: Exists
```
- Matches any node where the key storage is defined.

#### Demo: Checking Out Some Examples

```bash
cd labs/
vim pod-with-node-affinity.yaml
kubectl apply -f pod-with-node-affinity.yaml

vim pod-with-node-anti-affinity.yaml
kubectl appy -f pod-with-node-anti-affinity.yaml

vim pod-with-pod-affinity.yaml
kubectl apply -f pod-with-pod-affinity.yaml
```

#### Understanding TopologyKey

- When defining Pod affinity and anti-affinity, a toplogyKey property is required.
- The TopologyKey refers to a label that exists on nodes, and typically has a format containing a slash.
  - kubernetes.io/host
- Using TopologyKey allows the Pods only to be assigned to hosts matching the TopologyKey.
- This allows administrators to use zones where the workloads are implemented.
- If no matching TopologyKey is found on the host, the specified TopologyKey will be ignored in the affinity.

#### Demo: Using Pod Anti-Affinity

```bash
kubectl create -f redis-with-pod-affinity.yaml
kbuectl get pods
# On a two-node cluster, one Pod stays in a state of pending.
kubectl create -f web-with-pod-affinity.yaml
kbuectl get pods
# This will run web instances only on nodes where redis is running as well.
```

### 8.4 Managing Taints and Tolerations

#### Understanding Taints

- Taints are applied to a node to mark that the node should not accept any Pod that doesn't tolerate that taint.
- Tolerations are applied to Pods and allow (but do not require) Pods to schedule on nodes with matching Taints - so they are an exception to taints that are applied.
- Where Affinities are used on Pods to attract them to specific nodes, Taints allow a node to repel a set of Pods.
- Taints and Tolerations are used to ensure Pods are not scheduled on inappropriate nodes, and thus make sure that dedicated nodes can be configured for dedicated tasks.

#### Understanding Tains Types

- Three types of Taint can be applied.
  - **NoSchedule**: Does not schedule new Pods.
  - **PreferNoSchedule**: Does not schedule new Pods, unless there is no other option.
  - **NoExecute**: Migrates all Pods away from this node.

#### Setting Taints

- Taints are set in different ways.
- Control plane nodes automatically get taints that won't schedule user Pods.
- When **kubectl drain** and **kubectl cordon** are used, a taint is applied on the target node.
- Taints can be set automatically by the cluster when critical conditions arise, such as a node running out of disk space.
- administrators can use **kubectl taint** to set taints.
```bash
# Add taint.
kubectl taint nodes worker1 key1=value1:NoSchedule 

# Remove taint.
kubectl taint nodes worker1 key1=value1:NoSchedule-
```

#### Understanding Tolerations

- To allow a Pod to run on a node with a specific taint, a toleration can be used.
- This is essential for running core Kubernetes Pods on the control plane nodes.
- While creating taints and tolerations, a key and value are defined to allow for more specific access.
  - **kubectl taint nodes worker1 storage=ssd:NoSchedule**
- This will allow a Pod to run if it has a toleration containing the key **storage** and the value **ssd**

#### Understanding Taint Key and Value

- While defining a toleration, the Pod needs a key, operator, and value.
```yaml
tolerations:
- key: "storage"
  operator: "Equal"
  value: "ssd"
```
- The default value for the operator is "Equal", as an alternative, "Exists" is commonly used.
- If the operator "Exists" is used, the key should match the taint key and the value is ignored.
- If the operator "Equal" is used, the key and value must match.

#### Node Conditions and Taints

- Node conditions can automatically create taints on nodes if one of the following applies.
  - memory-pressure
  - disk-pressure
  - pid-pressure
  - unschedulable
  - network-unavailable
- If any of these conditions apply, a taint is automatically set.
- Node conditions can be ignored by addign corresponding Pod tolerations.

#### Demo: Using Taints

```bash
kubectl taint nodes worker1 storage=ssd:NoSchedule
kubectl get nodes
kubectl describe nodes worker1
kubectl create deployment nginx-taint --image=nginx
kubectl scale deployment nginx-taint --replicas=3
kubectl get pods -o wide --selector app=nginx-taint # will show that pods are all on worker2
cd labs/
vim taint-toleration.yaml
kubectl create -f taint-toleration.yaml # will run
kubectl get pods -o wide

vim taint-toleration2.yaml
kubectl create -f taint-toleration2.yaml # will not run
kubectl get pods -o wide
```

### 8.5 Understanding LimitRange and Quota

#### Understanding LimitRange

- LimitRange is an API object that limits resource usage per container or Pod in a Namespace.
- It uses three relevant options.
  - type: specifies whether it applies to Pods or containers.
  - defaultRequest: the default resources the application will request.
  - default: the maximum resources the application can use.

#### Understanding Quota

- Quota is an API object that limits total resources available in a Namespace.
- If a Namespace is configured with Quota, applications in that Namespace must be configured with resource settings in pod.spec.containers.resources
- Where the goal of the LimitRange is to set default restrictions for each application running in a Namespace, the goal of Quota is to define maximum resources that can be consumed within a Namespace by all applications.

### 8.6 Configuring Resource Limits and Requests

#### Demo: Managing Quota

```bash
kubectl create quota -h | less
kubectl create ns limited
kubectl create quota qtest --hard pods=3,cpu=100m,memory=500Mi --namespace limited
kubectl describe quota --namespace limited
kubectl create deploy nginx --image=nginx:latest --replicas=3 -n limited
kubectl get all -n limited # no pods
kubectl describe -n limited rs nginx-xxxxx # it fails because no quota have been set on the deployment.
kubectl set resources deploy nginx --requests cpu=100m,memory=5Mi --limits cpu=200m,memory=20Mi -n limited
kubectl get all -n limited 
kubectl describe quota --namespace limited

# Increase quota limit.
kubectl edit quota qtest -n limited
```

### 8.7 Configuring LimitRange

#### Demo: Defining Limitrange

```bash
kubectl explain limitrange.spec.limits
kubectl create ns limited
cd labs/
vi limitrange.yaml
kubectl apply -f limitrange.yaml -n limited
kubectl describe ns limited
# vi limitedpod.yaml
# kubectl apply -f limitedpod.yaml -n limited

kubectl run limited --image=nginx -n limited
kubectl describe pod limited -n limited # Look for Limits:
```

### Lesson 8 Lab Configuring Taints

- Create a taint on node worker2, that doesn't allow new Pods to te scheduled that don't have an SSD hard disk, unless they have the appropriate toleration set.
- Remove the taint after verifying that it works.

### Lesson 8 Lab Solution Configuring Taints

```bash
kubectl get nodes
kubectl get nodes -o wide
kubectl edit nodes worker-node-1 # Delete taints: block if it is exists.
kubectl taint nodes worker-node-2 storage=ssd:NoSchedule
kubectl create deployment newtaint --image=nginx --replicas=3
kubectl get all --selector app=newtaint -o wide

kubectl edit deployment newtaint
# - Open https://kubernetes.io/docs
# Search for **toleration**
# Click **Taints and Tolerations | Kubernetes**

# Add below content to Pod specification (under terminationGracePeriodSeconds:)
      tolerations:
      - key: "storage"
        operator: "Equal"
        value: "ssd"
        effect: "NoSchedule"

kubectl scale deployment newtaint --replicas=8
kubectl get pods --selector app=newtaint -o wide # Should schedule Pods on worker-node-2.

# Clean up
kubectl delete deployment newtaint
kubectl edit nodes worker-node-2 # Delete taints: block if it is exists.
kubectl get nodes
```
