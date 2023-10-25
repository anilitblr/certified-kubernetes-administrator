# Lesson 1 Understanding Kubernetes Architecture

- [Lesson 1 Understanding Kubernetes Architecture](#lesson-1-understanding-kubernetes-architecture)
    - [1.1 Vanilla Kubernetes and the Ecosystem](#11-vanilla-kubernetes-and-the-ecosystem)
      - [Understanding Vanilla Kubernetes](#understanding-vanilla-kubernetes)
      - [Understanding the Ecosystem](#understanding-the-ecosystem)
    - [1.2 Running Kubernetes in Cloud or on Premise](#12-running-kubernetes-in-cloud-or-on-premise)
      - [Running Kubernetes Anywhere](#running-kubernetes-anywhere)
    - [1.3 Kubernetes Distributions](#13-kubernetes-distributions)
      - [Understanding Kubernetes Distributions](#understanding-kubernetes-distributions)
      - [Common Kubernetes Distributions](#common-kubernetes-distributions)
    - [1.4 Kubernetes Node Roles](#14-kubernetes-node-roles)
      - [Kubernetes Node Roles](#kubernetes-node-roles)

### 1.1 Vanilla Kubernetes and the Ecosystem

#### Understanding Vanilla Kubernetes

- Vanilla Kubernetes is open-source Kubernetes, installed with **kubeadm** directly from the Kubernetes project Git repositories.
- A new release of Vanilla Kubernetes is published every 4 months.
- It provides core functionality, but doesn't contain some essential components.
  - Networking
  - Support
  - Graphical Dshboard and more
- To make a completely working environment, additional sulutions from the Kubernetes ecosystem are added.
- Navigate to **https://www.cncf.io** 
- Click **Projects** 
- Click **Graduated**
- You can see list of graduated Projects from CNCF.

#### Understanding the Ecosystem

- Cloud Native Computing Foundation (CNCF) hosts many projects related to the cloud native computing.
- Kubernetes is among the most important projects, but many other objects are offered as well, implementing a wide range of functionality.
  - Networking
  - Dashboard
  - Storage
  - Observability
  - Ingress
- To get a completely working Kubernetes solution, products from the ecosystem need to be installed also.
- This can be done manually, or by using a distribution.

### 1.2 Running Kubernetes in Cloud or on Premise

#### Running Kubernetes Anywhere

- Kubernetes is a platform for cloud native computing, and as such is commonly used in cloud.
- All major cloud providers have their own integrated Kubernetes distribution.
- Kubernetes can also be installed on premise, within the secure boundaries of your own datacenter.
- And also, there are all-on-one solutions which are prefect for learning Kubernetes.

### 1.3 Kubernetes Distributions

#### Understanding Kubernetes Distributions

- Kubernetes distribution add products from the ecosystem to vanilla Kubernetes and provide support.
- Normally, distributions run one or two Kubernetes versions behind.
- Some distributions are opinionated: they pick one product for a specific solution and support only that.
- Other distributions are less opinionated and integrate multiple products to offer specific solutions.

#### Common Kubernetes Distributions

- In Cloud
  - Amazon Elastic Kubernetes Service (EKS)
  - Azure Kubernetes Service (AKS)
  - Google Kubernetes Service (GKE)
- On Premises
  - OpenShift
  - Google Antos
  - Rancher
  - Canonical Charmed Kubernetes
- Minimal (learning) Solutions
  - Minikube
  - K3s

### 1.4 Kubernetes Node Roles

#### Kubernetes Node Roles

- The control plane runs Kubernetes core services, kubernetes agents, and no user workloads.
- The worker plane runs usre workloads and Kubernetes agents.
- All nodes are configured with a container runtime, which is required for running containerized workloads.
- The **kubelet** systemd service is responsible for running orchestrated containers as Pods on any node.
- Explain about how control and worker nodes communicate each other.
