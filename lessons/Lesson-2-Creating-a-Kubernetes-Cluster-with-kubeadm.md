# Lesson 2 Creating a Kubernetes Cluster with kubeadm

- [Lesson 2 Creating a Kubernetes Cluster with kubeadm](#lesson-2-creating-a-kubernetes-cluster-with-kubeadm)
    - [2.1 Understanding Cluster Node Requirements](#21-understanding-cluster-node-requirements)
      - [Node Requirements](#node-requirements)
      - [Installing a Container Runtime](#installing-a-container-runtime)
      - [Installing Kubernetes Tools](#installing-kubernetes-tools)
    - [2.2 Understanding Node Networking Requirements](#22-understanding-node-networking-requirements)
      - [Understanding Kubernetes Networking](#understanding-kubernetes-networking)
      - [Understanding Network Add-On](#understanding-network-add-on)
      - [Common Network Add-On](#common-network-add-on)
    - [2.3 Understanding Cluster Initialization](#23-understanding-cluster-initialization)
      - [Understanding Cluster Initialization](#understanding-cluster-initialization)
    - [2.4 Installing the Cluster](#24-installing-the-cluster)
      - [Nodes](#nodes)
      - [Setup static IP address and hostname](#setup-static-ip-address-and-hostname)
      - [Setup hostname resolution](#setup-hostname-resolution)
      - [kubeadm init Setup Procudure](#kubeadm-init-setup-procudure)
    - [2.5 Using kubeadm init](#25-using-kubeadm-init)
      - [Using kubeadm init](#using-kubeadm-init)
    - [2.6 Adding Nodes to the Kubernetes Cluster](#26-adding-nodes-to-the-kubernetes-cluster)
      - [Adding Nodes to the Cluster](#adding-nodes-to-the-cluster)
    - [2.7 Configuring the Kubernetes Client](#27-configuring-the-kubernetes-client)
      - [Understanding the Kubernetes Client](#understanding-the-kubernetes-client)
      - [Understanding Client Configuration](#understanding-client-configuration)
      - [Understanding Connectivity Parameters](#understanding-connectivity-parameters)
    - [Lesson 2 Lab Building a Kubernetes Cluster](#lesson-2-lab-building-a-kubernetes-cluster)
    - [Lesson 2 Lab Solution Building a Kubernetes Cluster](#lesson-2-lab-solution-building-a-kubernetes-cluster)

### 2.1 Understanding Cluster Node Requirements

#### Node Requirements

- To install a Kubernetes cluster using **kubeadm**, you will need at least two nodes that meet the following requirements.
  - Running a recent version of Ubuntu or CentOS
  - 2GiB RAM or more
  - 2 CPUs or more on the control-plane node
  - Network connectivity between the nodes
- Before setting up the cluster with **kubeadm**, install the following.
  - A container runtime
  - The Kubernetes tools

#### Installing a Container Runtime

- The container runtime is the component that allows you to run containers.
- Kubernetes supports different container runtimes.
  - containerd
  - CRI-O
  - Docerk Engine
  - Mirantis Container Runtime
- Installing a contaier runtime is not a CKA exam requirement

#### Installing Kubernetes Tools

- Before starting the installation, you will have to install the Kubernetes tools.
- These include the following.
  - **kubeadm**: used to install and manage a Kubernetes cluster.
  - **kubelet**: the core Kubernetes service that starts all Plds.
  - **kubectl**: the interface that allows you to run and manage applications in Kubernetes.
- Installing the Kubernetes tools is not a CKA requirement.

### 2.2 Understanding Node Networking Requirements

#### Understanding Kubernetes Networking

Different types of network communication are used in Kubernetes.
- **Node communication**: handled by the physical network.
- **External-to-Service communication**: handled by Kubernetes Service resources.
- **Pod-to-Service communication**: handled by Kubernetes Service.
- **Pod-to-Pod communication**: handled by the network plugin.
- **Container-to-Container communication**: handled within the Pod.

#### Understanding Network Add-On

- To create the software defined Pod network, a network add-on is needed.
- Different network add-ons are provided by the Kubernetes ecosystem.
- Vanilla Kubernetes doesn't come with a default add-on, as it doesn't want to favor a specific solution.
- Kubernetes provides the Container Network Interface (CNI), a generic interface that allows different solution.
- Availability of specific features depends on the network plugin that is used.
  - NetworkPolicy
  - IPv6
  - Role Based Access Control (RBAC)

#### Common Network Add-On

- **Calico**: probably the most common network plugin with support for all relevant features.
- **Flannel**: a generic network add-on that was used a lot in the past, but doesn't support NetworkPolicy.
- **Multus**: a plugin that can work with multiple network plugins. Current default in OpenShift.
- **Weave**: a common network add-on that does support common features.

### 2.3 Understanding Cluster Initialization

#### Understanding Cluster Initialization

While running **kubeadm init**, different phases are executed.
- **preflight**: ensures all conditions are met and core container images are downloaded.
- **certs**: a self-signed Kubernetes CA is generated, and related certificates are created for apiserver, etcd, and proxy.
- **kubeconfig**: configuration files are generated for core Kubernetes services.
- **kubelet-start**: the kubelet is started.
- **control-plane**: static Pod manifests are created and started for apiserver, controller manager, and scheduler.
- **etcd**: static Pod manifests are created and started for etcd.
- **uplaod-config**: ConfigMaps are created for ClusterConfiguration and kubelet component config.
- **upload-certs**: uploads all certificates to /etc/kubernetes/pki
- **mark-control-plane**: marks the node as control plane.
- **bootstrap-token**: generates the token that can be used to join other nodes.
- **kubelet-finalize**: Finalizes kubelet settings.
- **add-on**: installs coredns and kube-proxy add-ons

### 2.4 Installing the Cluster

#### Nodes

|ID | HOSTNAME              | IP ADDRESS    |
|---| --------------------- | ------------- |
| 1 | control-plane.lab.com | 192.168.1.200 |
| 2 | worker-node-1.lab.com | 192.168.1.201 |
| 3 | worker-node-2.lab.com | 192.168.1.202 |
|

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

# Note: Other commands to set up calico network.
# Install network add-on.
# ref: https://docs.tigera.io/calico/latest/getting-started/kubernetes/quickstart

# kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/tigera-operator.yaml

# kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/custom-resources.yaml
```

### 2.5 Using kubeadm init

#### Using kubeadm init

- While using **kubeadm init**, different arguments can be used to further define the cluster.
  - **--apiserver-advertise-address**: the Ip address on which the API server is listening.
  - **--config**: the name of a configuration file used for additional configuration.
  - **--dry-run**: performs a dry-run before actually installing for real.
  - **--pod-network-cidr**: sets the CIDR used for the Pod network.
  - **--service-cidr**: sets the service CIDR to something other than 10.96.0.0/12
- In most cases, just using **kubeadm init** will be enough.

```bash
kubeadm init -h | less
kubeadm -h
```

### 2.6 Adding Nodes to the Kubernetes Cluster

#### Adding Nodes to the Cluster

- When the Kubernetes cluster is initialized, a join token is generated.
- Use this join token to join other hosts, using **sudo kubeadm join** from the hosts you want to join.
- In case the join token is lost or expired, use **sudo kubeadm token create -- print-join-command**

```bash
sudo kubeadm token create -- print-join-command
```

### 2.7 Configuring the Kubernetes Client

#### Understanding the Kubernetes Client

- After cluster initialization, the /etc/kubernetes/admin.conf file is copied to ~/.kube/config to provide admin access to the kubernetes cluster.
- Alternatively, as Linux root user, use export.
- For more advanced user configuration, users must be created and provided with authrorizations using Role Based Access Control (RBAC)

#### Understanding Client Configuration

- The context groups client access parameters using a convenient name.
- By selecting a context, a specific group of parameters can be accessed.
- Each context has three parameterts.
  - **Cluster**: the cluster you want to connect to.
  - **Namespace**: the default Namespace.
  - **User**: the user account used.
- Use **kubectl config view** to view the context.
- Use **kubectl set-context** to set a different context.
- Use **kubectl use-context** to use a specific context.
```bash
cd ~/.kube
less config
kubectl config view
```

#### Understanding Connectivity Parameters

- The cluster is a Kubernetes cluster, defined by its endpoint and the certificate of the CA that has signed its kyes.
  - Use **kubectl config --kubeconfig=~/.kube/config set-cluster devcluster -- server=https://192.168.29.120 --certificate-authority=clusterca.crt**
- The namespace is the default namespace defined in this context.
  - Use **kubectl create ns** if it doesn't exist yet.
- The user is a account, defined by its X.509 certificates or other.
  - **kubectl config --kubeconfig=~/.kube/config set-credentials anil --client-certificate=anil.crt --client-key=anil.key**
- After defining all, use **kubectl set-context devcluster --cluster=devcluster --namespace=devspace --user=anil** to define the new context.

### Lesson 2 Lab Building a Kubernetes Cluster

- Apply all required steps to build a 3-node Kubernetes cluster.

### Lesson 2 Lab Solution Building a Kubernetes Cluster

- Follow this docs from the beginning to set 3-node cluster.