# Certified Kubernetes Administrator (CKA)

- [Certified Kubernetes Administrator (CKA)](#certified-kubernetes-administrator-cka)
- [Module 1 Building a Kubernetes Cluster](#module-1-building-a-kubernetes-cluster)
    - [Lesson 1 Understanding Kubernetes Architecture](#lesson-1-understanding-kubernetes-architecture)
    - [Lesson 2 Creating a Kubernetes Cluster with kubeadm](#lesson-2-creating-a-kubernetes-cluster-with-kubeadm)
- [Module 2 Running Applications](#module-2-running-applications)
    - [Lesson 3 Deploying Kubernetes Applications](#lesson-3-deploying-kubernetes-applications)
    - [Lesson 4 Managing Storage](#lesson-4-managing-storage)
    - [Lesson 5 Managing Application Access](#lesson-5-managing-application-access)
- [Module 3 Managing Kubernetes Clusters](#module-3-managing-kubernetes-clusters)
    - [Lesson 6 Managing Clusters](#lesson-6-managing-clusters)
    - [Lesson 7 Performing Node Maintenance Tasks](#lesson-7-performing-node-maintenance-tasks)
    - [Lesson 8 Managing Scheduling](#lesson-8-managing-scheduling)
    - [Lesson 9 Networking](#lesson-9-networking)
    - [Lesson 10 Managing Security Settings](#lesson-10-managing-security-settings)
    - [Lesson 11 Logging, Monitoring, and Troubleshooting](#lesson-11-logging-monitoring-and-troubleshooting)
- [Module 4 Practice Exams](#module-4-practice-exams)
    - [Lesson 12 Practice CKA Exam 1](#lesson-12-practice-cka-exam-1)
    - [Lesson 13 Practice CKA Exam 2](#lesson-13-practice-cka-exam-2)

# Module 1 Building a Kubernetes Cluster

### [Lesson 1 Understanding Kubernetes Architecture](lessons/Lesson-1-Understanding-Kubernetes-Architecture.md)

- 1.1 Vanilla Kubernetes and the Ecosystem
- 1.2 Running Kubernetes in Cloud or on Premise
- 1.3 Kubernetes Distributions
- 1.4 Kubernetes Node Roles

### [Lesson 2 Creating a Kubernetes Cluster with kubeadm](lessons/Lesson-2-Creating-a-Kubernetes-Cluster-with-kubeadm.md)

- 2.1 Understanding Cluster Node Requirements
- 2.2 Understanding Node Networking Requirements
- 2.3 Understanding Cluster Initialization
- 2.4 Installing the Cluster
- 2.5 Using kubeadm init
- 2.6 Adding Nodes to the Kubernetes Cluster
- 2.7 Configuring the Kubernetes Client
- Lesson 2 Lab Building a Kubernetes Cluster
- Lesson 2 Lab Solution Building a Kubernetes Cluster

# Module 2 Running Applications

### [Lesson 3 Deploying Kubernetes Applications](lessons/Lesson-3-Deploying-Kubernetes-Applications.md)

- 3.1 Using Deployments
- 3.2 Running Agents with DaemonSets
- 3.3 Using StatefulSets
- 3.4 The Case for Running Individual Pods
- 3.5 Managing Pod Initialization
- 3.6 Scaling Applications
- 3.7 Using Sidecar Containers for Application Logging
- Lesson 3 Lab Running a DaemonSet
- Lesson 3 Lab Solution Running a DaemonSet

### [Lesson 4 Managing Storage](lessons/Lesson-4-Managing-Storage.md)

- 4.1 Understanding Kubernetes Storage Options
- 4.2 Accessing Storage Through Pod Volumes
- 4.3 Configuring Persistent Volume (PV) Storage
- 4.4 Configuring PVCs
- 4.5 Configuring Pod Storage with PV and PVCs
- 4.6 Using StorageClass
- 4.7 Understanding Storage Provisioners
- 4.8 Using ConfigMaps and Secrets as Volumes
- Lesson 4 Lab Setting up Storage
- Lesson 4 Lab Solution Setting up Storage

### [Lesson 5 Managing Application Access](lessons/Lesson-5-Managing-Application-Access.md)

- 5.1 Exploring Kubernetes Networking
- 5.2 Understanding Network Plugins
- 5.3 Using Services to Access Applications
- 5.4 Running an Ingress Controller
- 5.5 Configuring Ingress
- 5.6 Using Port Forwarding for Direct Application Access
- Lesson 5 Lab Managing Networking
- Lesson 5 Lab Solution Managing Networking

# Module 3 Managing Kubernetes Clusters

### [Lesson 6 Managing Clusters](lessons/Lesson-6-Managing-Clusters.md)

- 6.1 Analyzing Cluster Nodes
- 6.2 Using crictl to Manage Node Containers
- 6.3 Running Static Pods
- 6.4 Managing Node State
- 6.5 Managing Node Services
- Lesson 6 Lab Running Static Pods
- Lesson 6 Lab Solution Running Static Pods

### [Lesson 7 Performing Node Maintenance Tasks](lessons/Lesson-7-Performing-Node-Maintenance-Tasks.md)

- 7.1 Using Metrics Server to Monitor Node and Pod State
- 7.2 Backing up the Etcd
- 7.3 Restoring the Etcd
- 7.4 Performing Cluster Node Upgrades
- 7.5 Understanding Cluster High Availability (HA) Options
- 7.6 Setting up a Highly Available Kubernetes Cluster
- Lesson 7 Lab Etcd Backup and Restore
- Lesson 7 Lab Solution Etcd Backup and Restore

### [Lesson 8 Managing Scheduling](lessons/Lesson-8-Managing-Scheduling.md)

- 8.1 Exploring the Scheduling Process
- 8.2 Setting Node Preferences
- 8.3 Managing Affinity and Anti-Affinity Rules
- 8.4 Managing Taints and Tolerations
- 8.5 Understanding LimitRange and Quota
- 8.6 Configuring Resource Limits and Requests
- 8.7 Configuring LimitRange
- Lesson 8 Lab Configuring Taints
- Lesson 8 Lab Solution Configuring Taints

### [Lesson 9 Networking](lessons/Lesson-9-Networking.md)

- 9.1 Managing the CNI and Network Plugins
- 9.2 Understanding Service Auto Registration
- 9.3 Using Network Policies to Manage Traffic Between Pods
- 9.4 Configuring Network Policies to Manage Traffic Between Namespaces
- Lesson 9 Lab Using NetworkPolicies
- Lesson 9 Lab Solution Using NetworkPolicies

### [Lesson 10 Managing Security Settings](lessons/Lesson-10-Managing-Security-Settings.md)

- 10.1 Understanding API Access
- 10.2 Managing SecurityContext
- 10.3 Using ServiceAccounts to Configure API Access
- 10.4 Setting Up Role Based Access Control (RBAC)
- 10.5 Configuring Cluster Roles and RoleBindings
- 10.6 Creating Kubernetes User Accounts
- Lesson 10 Lab Managing Security
- Lesson 10 Lab Solution Managing Security

### [Lesson 11 Logging, Monitoring, and Troubleshooting](lessons/Lesson-11-Logging-Monitoring-and-Troubleshooting.md)

- 11.1 Monitoring Kubernetes Resources
- 11.2 Understanding the Troubleshooting Flow
- 11.3 Troubleshooting Kubernetes Applications
- 11.4 Troubleshooting Cluster Nodes
- 11.5 Fixing Application Access Problems
- Lesson 11 Lab Troubleshooting Nodes
- Lesson 11 Lab Solution Troubleshooting Nodes

# Module 4 Practice Exams

### [Lesson 12 Practice CKA Exam 1](lessons/Lesson-12-Practice-CKA-Exam-1.md)

- 12.1 Question Overview
- 12.2 Creating a Kubernetes Cluster
- 12.3 Scheduling a Pod
- 12.4 Managing Application Initialization
- 12.5 Setting up Persistent Storage
- 12.6 Configuring Application Access
- 12.7 Securing Network Traffic
- 12.8 Setting up Quota
- 12.9 Creating a Static Pod
- 12.10 Troubleshooting Node Services
- 12.11 Configuring Cluster Access
- 12.12 Configuring Taints and Tolerations

### [Lesson 13 Practice CKA Exam 2](lessons/Lesson-13-Practice-CKA-Exam-2.md)

- 13.1 Question Overview
- 13.2 Configuring a High Availability Cluster
- 13.3 Etcd Backup and Restore
- 13.4 Performing a Control Node Upgrade
- 13.5 Configuring Application Logging
- 13.6 Managing Persistent Volume Claims
- 13.7 Investigating Pod Logs
- 13.8 Analyzing Performance
- 13.9 Managing Scheduling
- 13.10 Configuring Ingress
- 13.11 Preparing for Node Maintenance
- 13.12 Scaling Applications
