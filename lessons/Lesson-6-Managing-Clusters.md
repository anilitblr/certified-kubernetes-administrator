# Lesson 6 Managing Clusters

- [Lesson 6 Managing Clusters](#lesson-6-managing-clusters)
    - [6.1 Analyzing Cluster Nodes](#61-analyzing-cluster-nodes)
      - [Analyzing Cluster Nodes](#analyzing-cluster-nodes)
      - [Demo: Analyzing Node State](#demo-analyzing-node-state)
    - [6.2 Using crictl to Manage Node Containers](#62-using-crictl-to-manage-node-containers)
      - [Understanding crictl](#understanding-crictl)
      - [Using crictl](#using-crictl)
    - [6.3 Running Static Pods](#63-running-static-pods)
      - [Understanding Static Pods](#understanding-static-pods)
    - [6.4 Managing Node State](#64-managing-node-state)
      - [Managing Node State](#managing-node-state)
      - [Demo: Managing Node State](#demo-managing-node-state)
    - [6.5 Managing Node Services](#65-managing-node-services)
      - [Maganing Node Services](#maganing-node-services)
      - [Demo: Maganing Node Services](#demo-maganing-node-services)
    - [Lesson 6 Lab Running Static Pods](#lesson-6-lab-running-static-pods)
    - [Lesson 6 Lab Solution Running Static Pods](#lesson-6-lab-solution-running-static-pods)

### 6.1 Analyzing Cluster Nodes

#### Analyzing Cluster Nodes

- Kubernetes cluster nodes run Linux processes. To monitor these processes, generic Linux rules apply.
  - Use **systemclt status kubelet** to get runtime information about the kubelet.
  - Use log files in /var/log as well as **journalctl** output to get access to logs.
- Generic node information is obtained through **kubectl describe**
- If the Metrics Server is installed, use **kubectl top nodes** to get a summary of CPU/memory usage on a node. See Lession 7.1 for more about this.

#### Demo: Analyzing Node State

```bash
kubectl get nodes

# Stop kubelet service.
ssh anil@worker-node-1
sudo systemctl stop kubelet
exit

kubectl get nodes
kubectl describe node worker-node-1

# Start kubelet service.
ssh anil@worker-node-1
sudo systemctl start kubelet
exit

kubectl get nodes
kubectl describe node worker-node-1

sudo ls -lrt /var/log
sudo journalctl # Use UPPER G to go all the way down.
sudo journalctl -u kubelet
systemctl status kubelet
```

### 6.2 Using crictl to Manage Node Containers

#### Understanding crictl

- All Pods are started as containers on the nodes.
- **crictl** is a generic tool that communicates to the container runtime to get information about running containers.
- As such, it replaces generic tools like **docker** and **podman**
- To use it, a runtime-endpoint and image-endpoint need to be set.
- The most convenient way to do so, is by defining the /etc/crictl.yaml file on the nodes where you want to run **crictl**

#### Using crictl

```bash
# List containers.
sudo crictl ps

# List Pods that have been scheduled on this node.
sudo crictl pods

# Inspect container configuration.
sudo crictl inspect <name-or-id>

# Pull an images.
sudo crictl pull <imagename>
sudo crictl pull docker.io/library/mysql

# List images
sudo crictl images

# For more options, use
sudo crictl --help
```

### 6.3 Running Static Pods

#### Understanding Static Pods

- The kubelet systemd process is configured to run static Pods from the /etc/kubernetes/manifests directory.
- On the control node, static Pods are an essential part  of how kubernetes works: systemd starts kubelet, and kubelet starts core Kubernetes services as static Pods.
- Administrators can manually add static Pods if so desired, just copy a manifest file into the /etc/kubernetes/manifests directory and the kubelet process will pick it up.
- To modify the path where Kubelet picks up the static Pods, edit staticPodPath in /var/lib/kubelet/config.yaml and use **sudo systemctl restart kubelet** to restart.
- Never do this on the control node!

```bash
kubectl run staticpod --image=nginx --dry-run=client -o yaml # Copy the output.
ssh anil@worker-node-1
vi staticpod.yaml # Paste the content into this file.
sudo cp staticpod.yaml /etc/kubernetes/manifests/ 
exit
kubectl get pods -o wide
```

### 6.4 Managing Node State

#### Managing Node State

- **kubectl crodon** is used to mark a node a unschedulable.
- **kubectl drain** is used to mark a node as unschedulable and remove all running Pods from it.
  - Pods that have been started from a DaemonSet will not be removed while using **kubectl drain**, add **--ignore-daemonsets** to ignore that.
  - Add **--delete-emptydir-data** to delete data from emptyDir Pod volumes.
- While using **cordon or drain**, a taint is set on the nodes (see Lesson 8.4 for more information)
- Use **kubectl uncordon** to get the node back in a schedulable state.

#### Demo: Managing Node State

```bash
kubectl get nodes
kubectl get nodes -o wide
kubectl cordon worker-node-2
kubectl get nodes
kubectl describe node worker-node-2 # Look for taints.
kubectl get nodes

kubectl create deploy manynginx --image=nginx --replicas=10
kubectl get pods --selector app=manynginx

kubectl uncordon worker-node-2
kubectl get nodes

kubectl drain worker-node-2
kubectl drain worker-node-2 --ignore-daemonsets
kubectl get nodes
kubectl get pods -o wide

kubectl uncordon worker-node-2
kubectl get nodes
```

### 6.5 Managing Node Services

#### Maganing Node Services

- The container runtime (often containerd) and kubelet are managed by the Linux systemd service manager.
- Use **systemctl status kubelet** to check the current status of the kubelet.
- To manually start it, use **sudo systemctl start kubelet**
- Notice that Pods that are scheduled on a node show as container processes in **ps aux** output. Don't use Linux tools to manage Pods!

#### Demo: Maganing Node Services

```bash
ssh anil@worker-node-1
ps aux | grep kubelet
sudo kill -9 $(pidof kubelet)
ps aux | grep containerd
sudo systemctl status kubelet
sudo systemctl cat kubelet.service
sudo systemctl stop kubelet
sleep 10
sudo systemctl status kubelet
sudo systemctl start kubelet
sudo systemctl status containerd
```

### Lesson 6 Lab Running Static Pods

- On node worker-node-1, run a static Pod with the name mypod, using an Nginx image and no further configuration.
- Use the appropriate tools to verify that the static Pod has started successfully.

### Lesson 6 Lab Solution Running Static Pods

```bash
kubectl run mypod --image=nginx --dry-run=client -o yaml # Copy the output.
ssh anil@worker-node-1
vi mypod.yaml # Paste the content into this file.
sudo cp mypod.yaml /etc/kubernetes/manifests/
sudo crictl pods | grep mypod
exit
kubectl get pods -o wide 
```
