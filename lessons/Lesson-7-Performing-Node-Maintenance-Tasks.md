# Lesson 7 Performing Node Maintenance Tasks

- [Lesson 7 Performing Node Maintenance Tasks](#lesson-7-performing-node-maintenance-tasks)
    - [7.1 Using Metrics Server to Monitor Node and Pod State](#71-using-metrics-server-to-monitor-node-and-pod-state)
      - [Understanding Kubernetes Monitoring](#understanding-kubernetes-monitoring)
    - [7.2 Backing up the Etcd](#72-backing-up-the-etcd)
      - [Understanding the Etcd](#understanding-the-etcd)
      - [Understanding Etcd Backup](#understanding-etcd-backup)
      - [Demo: Backing up the etcd](#demo-backing-up-the-etcd)
      - [Demo: Verifying the Etcd Backup](#demo-verifying-the-etcd-backup)
    - [7.3 Restoring the Etcd](#73-restoring-the-etcd)
      - [Understanding the Procedure](#understanding-the-procedure)
      - [Demo: Restoring the Etcd](#demo-restoring-the-etcd)
    - [7.4 Performing Cluster Node Upgrades](#74-performing-cluster-node-upgrades)
      - [Understanding the Procedure](#understanding-the-procedure-1)
      - [Control Plane Node Upgrade Overview](#control-plane-node-upgrade-overview)
    - [7.5 Understanding Cluster High Availability (HA) Options](#75-understanding-cluster-high-availability-ha-options)
      - [HA Requirements](#ha-requirements)
      - [Exploring Load Balancer Configuration](#exploring-load-balancer-configuration)
    - [7.6 Setting up a Highly Available Kubernetes Cluster](#76-setting-up-a-highly-available-kubernetes-cluster)
      - [Cluster Node Requirements](#cluster-node-requirements)
      - [Initializing the HA Setup](#initializing-the-ha-setup)
      - [Configuring the HA Client](#configuring-the-ha-client)
      - [Testing it](#testing-it)
    - [Lesson 7 Lab Etcd Backup and Restore](#lesson-7-lab-etcd-backup-and-restore)
    - [Lesson 7 Lab Solution Etcd Backup and Restore](#lesson-7-lab-solution-etcd-backup-and-restore)
      - [Backing up the etcd](#backing-up-the-etcd)
      - [Verifying the Etcd Backup](#verifying-the-etcd-backup)
      - [Restoring the Etcd](#restoring-the-etcd)

### 7.1 Using Metrics Server to Monitor Node and Pod State

#### Understanding Kubernetes Monitoring

- Kubernetes monitoring is offered by the integrated Mertics Server.
- Read github documentation.
```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/componentes.yaml # ref: https://github.com/kubernetes-sigs/metrics-server
kubectl -n kube-system get pods # look for metrics-server
kubectl logs -n kube-system metrics-server-xxx-yyy

kubectl -n kube-system edit deployment metrics-server
    spec.template.spec.containers.args # use the following
    - --kubelet-insecure-tls
    - kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname

kubeclt -n kube-system logs met4rics-server<TAB> # Should show "Generating self-signed cert" and "Serving securely on [::]443"

kubectl top pods --all-namespaces # will show most active Pods.
kubectl top nodes
```

### 7.2 Backing up the Etcd

#### Understanding the Etcd

- The etcd is a core Kubernetes service that contains all resources that have been created.
- It is started by the kubelet as a static Pod on the control node.
- Losing the etcd means losing all your configuration.

#### Understanding Etcd Backup

- To back up the etcd, root access is required to run the **etcdctl** tools
- Use **sudo apt install etcd-client** to install this tool.
- **etcdctl** uses the wrong API version by default, fix this by using **sudo ETCDCTL_API=3 etcdctl ... snapshot save**
- To use **etcdctl**, you need to specify the etcd service API endpoint, as well as cacert, cert and key to be used.
- Values for all of these can be obtained by using **ps aux | grep etcd**

#### Demo: Backing up the etcd

```bash
sudo apt install etcd-client
sudo etcdctl --help
sudo ETCDCTL_API=3 etcdctl --help
ps aux | grep etcd

sudo ETCDCTL_API=3 etcdctl --endpoints=localhost:2379 \
--cacert /etc/kubernetes/pki/etcd/ca.crt \
--cert /etc/kubernetes/pki/etcd/server.crt \
--key /etc/kubernetes/pki/etcd/server.key get / --prefix --keys-only 

sudo ETCDCTL_API=3 etcdctl --endpoints=localhost:2379 \
--cacert /etc/kubernetes/pki/etcd/ca.crt \
--cert /etc/kubernetes/pki/etcd/server.crt \
--key /etc/kubernetes/pki/etcd/server.key snapshot save /tmp/etcdbackup.db
```

#### Demo: Verifying the Etcd Backup

```bash
sudo ETCDCTL_API=3 etcdctl --write-out=table snapshot status /tmp/etcdbackup.db

# Just to be sure.
cp /tmp/etcdbackup.db /tmp/etcdbackup.db.2 
```

### 7.3 Restoring the Etcd

#### Understanding the Procedure

- **sudo ETCDCTL_API=3 etcdctl snapshot restore /tmp/etcdbackup.db --data-dir /var/lib/etcd-backup** restores the etcd backup in a non-default folder.
- To start using it, the Kubernetes core services must be stopped, after which the etcd can be reconfigured to use the new directory.
- To stop the core services, temporarily move **/etc/kubernetes/manifests/*.yml to /etc/kubernetes/**
- As the kubelet process temporarily pools for static Pod files, the etcd process will disappear within a minute.
- Use **sudo crictl ps**to verify that is has been stopped.
- Once the etcd Pod has stopped, reconfigure the etcd to use the non-default etcd path.
- In etcd.yaml you will find a HostPath volume with the name etcd-data, pointing to the location where the Etcd files are found. Change this to the location where the restored files are.
- Move back the static Pod files to /etc/kubernetes/manifest/
- Use **sudo crictl ps** to verify the Pods have restarted successfully.
- Next, **kubectl get all** should show the original Etcd resources.

#### Demo: Restoring the Etcd

```bash
kubectl get deploy
kubectl delete --all deploy # or delete only few deployments.
cd /etc/kubernetes/manifests/
sudo mv * .. # This will stop all running pods.
sudo crictl ps
sudo ETCDCTL_API=3 etcdctl snapshot restore /tmp/etcdbackup.db --data-dir /var/lib/etcd-backup
sudo ls -l /var/lib/etcd-backup/
sudo vi /etc/kubernetes/etcd.yaml # change path: /var/lib/etcd HostPath to path: /var/lib/etcd-backup
sudo mv ../*.yaml .
sudo crictl ps # should show all resources
kubectl get deploy -A
```

### 7.4 Performing Cluster Node Upgrades

#### Understanding the Procedure

- Kubernetes clusters can be upgraded from one to another minor versions.
- Skipping minor versios (1.23 to 1.25) is not supported.
- First, you will have to upgrade **kubeadm**
- Next, you will need to upgrade the control plane node.
- After that, the worker nodes are upgraded.
Exam Tip! Use "Upgrading kubeadm clusters" from the documentation.

#### Control Plane Node Upgrade Overview

```bash
# Open - https://kubernetes.io/docs
# Search for **Upgrading kubeadm clusters**
# Click **Upgrading kubeadm clusters | Kubernetes**

kubectl get nodes # look at VERSION

# Update
sudo -i
apt update
apt-cache madison kubeadm

# Upgrade kubeadm
apt-mark unhold kubeadm && \
apt-get update && apt-get install -y kubeadm='1.28.x-*' && \
apt-mark hold kubeadm # change  kubeadm='1.28.x-*' to next version to upgrade.

# Verify that the download works and has the expected version
kubeadm version

# Verify the upgrade plan
kubeadm upgrade plan # Use to check available versions.

kubeadm upgrade apply v1.xx.y # Use to run the upgrade.
exit

# Drain the node
kubectl drain control-plane --ignore-daemonsets

# Upgrade kubelet and kubectl
# 1. Upgrade the kubelet and kubectl:
sudo -i

apt-mark unhold kubelet kubectl && \
apt-get update && apt-get install -y kubelet='1.28.x-*' kubectl='1.28.x-*' && \
apt-mark hold kubelet kubectl # change  kubeadm='1.28.x-*' to next version to upgrade.

# 2. Restart the kubelet
systemctl daemon-reload
systemctl restart kubelet
exit

# Uncordon the node
kubectl uncordon control-plane # Use to bring back the control plane

# Verify the status of the cluster
kubectl get nodes # look at VERSION

# Proceed with other nodes.
# TODO: Follow the documentation to upgrade the cluster nodes.
```

### 7.5 Understanding Cluster High Availability (HA) Options

- Stacked control plane nodes requires less infrastructure as the etcd members, and control plane nodes are co-located.
  - Control plnes and etcd members are running together on the same node.
  - For optional protection, requires a minimum of 3 stacked control plane nodes.
- External etcd cluster requires more infrastructure as the control plane nodes and etcd members are separated.

#### HA Requirements

- In a Kubernetes HA cluster, a load balancer is needed to distribute the workload between the cluster nodes.
- The load balancer can be externally provided using open source software, or a load balancer appliance.
- Knowledge of setting up the load balancer is not required on the CKA exam: in this course a load balancer set up script is provided.

#### Exploring Load Balancer Configuration

- In the load balancer setup, HAProxy is running on each server to provide access to port 8443 on all IP addresses on that server.
- Incoming traffic on port 8443 is forwarded to the kube-apiserver port 6443
- The keepalived service is running on all HA nodes to provide a virtual IP address on one of the nodes.
- **kubectl** clients connect to thei VIP:8443
- Use the **setup-lb-ubuntu.sh** script provided in the labs directory for easy setup.
- Additional instructions are in the script.
- After running the load balancer setup, use **nc 192.168.x.x:8443** to verify the availability of the load balancer IP and port. 

```bash
# 1. Keep 5 virtal machines ready 1 for HA Proxy, 2 for control-plane, 2 for worker node.
On all nodes.
cd labs/
sudo ./setup-container.sh 
sudo ./setup-kubetools.sh

# Setup load balancer
cd labs/
./setup-lb-ubuntu.sh

ip a # Make sure there a Virtual IP address is set.
```

### 7.6 Setting up a Highly Available Kubernetes Cluster

#### Cluster Node Requirements

- 3 VMs to be used as controllers in the cluster. Install K8s software but don't set up the cluster yet.
- 2 VMs to be used a sworker nodes. Install K8s software.
- Ensure /etc/hosts is set up for name resolution of all nodes and copy to all nodes.
- Disable **selinux** on all noes if applicable.
- Disable firewall if applicable.

#### Initializing the HA Setup

- **sudo kubeadm init --control-plane-endpoint "192.168.x.x:8443" --upload-certs**
  - Save the output of the command which shows next steps.
- Configure networking.
  - **kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml**
- Copy the **kubectl join** command that was printed after successfully initializing the first control node.
  - Make sure to use the command that has **--control-plnae** in it
- Complete setup on other control nodes as instructed.
- Use **kubectl get nodes** to verify setup.
- Continue and join worker nodes as instructed.

```bash
ssh anil@control1
sudo kubeadm init --control-plane-endpoint "192.168.x.x:8443" --upload-certs

mkdir ~/.kube
sudo cp -i /etc/kubernetes/admin.conf ~/.kube/config
sudo chown $(id -u):$(id -g) ~/.kube/config

# Install network plugin.
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml

# Copy the this command and execute it on the control-node-1 and 2
sudo kubeadm join 192.168.x.x:8443 <token>

# Copy the this command and execute it on the worker-node-1 and 2
sudo kubeadm join 192.168.x.x:8443 <token>

ssh anil@control1
kubectl get nodes
```

#### Configuring the HA Client

- On the machine you want to use as operator workstation, create a .kube directory and copy /etc/kubernetes/admin.conf from any control node to the client machine.
- Install the **kubectl** utility.
- Ensure that host name resolution goes to the new control plane VIP.
- Verify using **kubectl get nodes**

#### Testing it

```bash
# On all nodes, find the VIP using.
ip a

# On all noes with a kubectl to verify client working.
ssh anil@control2
kubectl get all

# Shutdown the node that has the VIP.
ssh anil@control1
sudo poweroff

# Verify that still works
ssh anil@control2
kubectl get all

# Troubleshooting, consider using 
sudo systemctl restart haproxy
kubectl get all 
```

### Lesson 7 Lab Etcd Backup and Restore

- Create a backup of the etcd.
- Remove a few resources (Pods and/or Deployments)
- Restore the backup of the etcd and verify that gets your resources back.

### Lesson 7 Lab Solution Etcd Backup and Restore

#### Backing up the etcd

```bash
sudo apt install etcd-client
sudo etcdctl --help
sudo ETCDCTL_API=3 etcdctl --help
ps aux | grep etcd

sudo ETCDCTL_API=3 etcdctl --endpoints=localhost:2379 \
--cacert /etc/kubernetes/pki/etcd/ca.crt \
--cert /etc/kubernetes/pki/etcd/server.crt \
--key /etc/kubernetes/pki/etcd/server.key get / --prefix --keys-only 

sudo ETCDCTL_API=3 etcdctl --endpoints=localhost:2379 \
--cacert /etc/kubernetes/pki/etcd/ca.crt \
--cert /etc/kubernetes/pki/etcd/server.crt \
--key /etc/kubernetes/pki/etcd/server.key snapshot save /tmp/etcdbackup.db
```

#### Verifying the Etcd Backup

```bash
sudo ETCDCTL_API=3 etcdctl --write-out=table snapshot status /tmp/etcdbackup.db

# Just to be sure.
cp /tmp/etcdbackup.db /tmp/etcdbackup.db.2 
```

#### Restoring the Etcd

```bash
kubectl get deploy
kubectl delete --all deploy # or delete only few deployments.
cd /etc/kubernetes/manifests/
sudo mv * .. # This will stop all running pods.
sudo crictl ps
sudo ETCDCTL_API=3 etcdctl snapshot restore /tmp/etcdbackup.db --data-dir /var/lib/etcd-backup
sudo ls -l /var/lib/etcd-backup/
sudo vi /etc/kubernetes/etcd.yaml # change path: /var/lib/etcd HostPath to path: /var/lib/etcd-backup
sudo mv ../*.yaml .
sudo crictl ps # should show all resources
kubectl get deploy -A
```
