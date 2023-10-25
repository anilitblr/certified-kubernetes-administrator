# Lesson 13 Practice CKA Exam 2

- [Lesson 13 Practice CKA Exam 2](#lesson-13-practice-cka-exam-2)
    - [13.1 Question Overview](#131-question-overview)
      - [Question 1: Configuring a High Availability Cluster](#question-1-configuring-a-high-availability-cluster)
      - [Question 2: Etcd Backup and Restore](#question-2-etcd-backup-and-restore)
      - [Question 3: Performing a Control Node Upgrade](#question-3-performing-a-control-node-upgrade)
      - [Question 4: Configuring Application Logging](#question-4-configuring-application-logging)
      - [Question 5: Managing Persistent Volume Claims](#question-5-managing-persistent-volume-claims)
      - [Question 6: Investigating Pod Logs](#question-6-investigating-pod-logs)
      - [Question 7: Analyzing Performance](#question-7-analyzing-performance)
      - [Question 8: Managing Scheduling](#question-8-managing-scheduling)
      - [Question 9: Configuring Ingress](#question-9-configuring-ingress)
      - [Question 10: Preparing for Node Maintenance](#question-10-preparing-for-node-maintenance)
      - [Question 11: Scaling Applications](#question-11-scaling-applications)
    - [13.2 Configuring a High Availability Cluster](#132-configuring-a-high-availability-cluster)
    - [13.3 Etcd Backup and Restore](#133-etcd-backup-and-restore)
      - [Backing up the etcd](#backing-up-the-etcd)
      - [Verifying the Etcd Backup](#verifying-the-etcd-backup)
      - [Restoring the Etcd](#restoring-the-etcd)
    - [13.4 Performing a Control Node Upgrade](#134-performing-a-control-node-upgrade)
    - [13.5 Configuring Application Logging](#135-configuring-application-logging)
    - [13.6 Managing Persistent Volume Claims](#136-managing-persistent-volume-claims)
    - [13.7 Investigating Pod Logs](#137-investigating-pod-logs)
    - [13.8 Analyzing Performance](#138-analyzing-performance)
    - [13.9 Managing Scheduling](#139-managing-scheduling)
    - [13.10 Configuring Ingress](#1310-configuring-ingress)
    - [13.11 Preparing for Node Maintenance](#1311-preparing-for-node-maintenance)
    - [13.12 Scaling Applications](#1312-scaling-applications)

### 13.1 Question Overview

#### Question 1: Configuring a High Availability Cluster

- Configure a High Availability cluster with three control plane nodes and two worker nodes.
- Ensure that each control plane node can be used as a client as well.
- Use the scripts provided in the course Git repository to install the CRI, kubetools and load balancer.

#### Question 2: Etcd Backup and Restore

Note: all tasks from here on should be performed on a non-HA cluster. 
- Before creating the backup, creating a Deployment that runs nginx.
- Create a backup of the etcd and write it to /tmp/etcdbackup.
- Delete the Deployment you just created.
- Restore the backup that you have created in the first step of this procedure and verify that the Deployment is available again.

#### Question 3: Performing a Control Node Upgrade

- Notice that this task requires you to have a control node running an older version of Kubernetes available.
- Update the control node to the latest version of Kubernetes.
- Ensure tht the kubelet and kubectl are updated as well.

#### Question 4: Configuring Application Logging

- Create a Pod with a logging agent that runs as a sidecar container.
- Configure the main application to use Busybox and run the Linux **date** command every minute. The reslut of this command should be written to the directory /output/date.log.
- Set up a sidecar container that runs Nginx and provide access to the date.log file on /usr/share/nginx/html/date.log

#### Question 5: Managing Persistent Volume Claims

- Create a PersistentVolume that uses 1GB of HostPath storage.
- Create a PersistentVolumeClaim that uses the PersistentVolume; the PersistentVolumeClaim should request 100 MiB storage.
- Run a Pod with the name storage, using the Nginx image and mounting this PVC on the directory /data.
- After creating the configuration, change the PersistentVolumeClaim to request a size of 200MiB.

#### Question 6: Investigating Pod Logs

- Run a Pod with the name failngdb, which starts the mariadb image without any further options (it should fail).
- Investigate the Pod logs and write all lines that start with ERROR to /tmp/failingdb.log

#### Question 7: Analyzing Performance

- Find out which Pod currently has the highest CPU load.

#### Question 8: Managing Scheduling

- Run a Pod with the name lab139pod.
- Ensure that it only runs on nodes that have the label storage=ssd set.

#### Question 9: Configuring Ingress

- Run a Pod with the name lab1310pod, using the Nginx image.
- Expose this Pod using a NodePort type Service.
- Configure Ingress such that its web content is available on the path lab1310.info/hi
- You will not have to configure an Ingress controller for this assignment, just the API resource is enough.

#### Question 10: Preparing for Node Maintenance

- Schedule node worker2 for maintenance in such a way that all running Pods are evicted.

#### Question 11: Scaling Applications

- Run a Deployment with the name lab1312deploy using the Nginx image.
- Scale it such that it runs 6 application instances.

### 13.2 Configuring a High Availability Cluster

- Configure a High Availability cluster with three control plane nodes and two worker nodes.
- Ensure that each control plane node can be used as a client as well.
- Use the scripts provided in the course Git repository to install the CRI, kubetools and load balancer.
 
```bash
# Note: Follow - 7.6 Setting up a Highly Available Kubernetes Cluster for more details.
# 1. Open - https://kubernetes.io/docs/home/
# 2. Click **Creating Highly Available Clusters with kubeadm | Kubernetes**
sudo kubeadm init --control-plane-endpoint "192.168.1.200:8443" --upload-certs

mkdir ~/.kube
sudo cp -i /etc/kubernetes/admin.conf ~/.kube/config
sudo chown $(id -u):$(id -g) ~/.kube/config
kubectl get all

# Install network plugin.
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml

# Copy the this command and execute it on the control-node-1 and 2
sudo kubeadm join 192.168.x.x:8443 <token>

# Copy the this command and execute it on the worker-node-1 and 2
sudo kubeadm join 192.168.x.x:8443 <token>

ssh anil@control1
kubectl get nodes
``` 

### 13.3 Etcd Backup and Restore

Note: all tasks from here on should be performed on a non-HA cluster. 
- Before creating the backup, creating a Deployment that runs nginx.
- Create a backup of the etcd and write it to /tmp/etcdbackup.
- Delete the Deployment you just created.
- Restore the backup that you have created in the first step of this procedure and verify that the Deployment is available again.

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

### 13.4 Performing a Control Node Upgrade

- Notice that this task requires you to have a control node running an older version of Kubernetes available.
- Update the control node to the latest version of Kubernetes.
- Ensure tht the kubelet and kubectl are updated as well.

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

### 13.5 Configuring Application Logging

- Create a Pod with a logging agent that runs as a sidecar container.
- Configure the main application to use Busybox and run the Linux **date** command every minute. The reslut of this command should be written to the directory /output/date.log.
- Set up a sidecar container that runs Nginx and provide access to the date.log file on /usr/share/nginx/html/date.log

```bash
# Open - https://kubernetes.io/docs
# Search for **logging**
# Click **Logging Architecture | Kubernetes**
# Copy the YAML code which is present under this `Here's a manifest for a pod that has two sidecar containers:`
# vi lab135.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: counter
spec:
  containers:
  - name: count
    image: busybox:1.28
    args:
    - /bin/sh
    - -c
    - >
      while sleep 60;
      do
        echo "$(date)" >> /output/date.log
      done      
    volumeMounts:
    - name: varlog
      mountPath: /output
  - name: count-log-1
    image: nginx
    volumeMounts:
    - name: varlog
      mountPath: /usr/share/nginx/html
  volumes:
  - name: varlog
    emptyDir: {}
```

```bash
kubectl apply -f lab135.yaml;
kubectl exec -it counter -c count-log-1 -- cat /usr/share/nginx/html/date.log 
```

### 13.6 Managing Persistent Volume Claims

- Create a PersistentVolume that uses 1GB of HostPath storage.
- Create a PersistentVolumeClaim that uses the PersistentVolume; the PersistentVolumeClaim should request 100 MiB storage.
- Run a Pod with the name storage, using the Nginx image and mounting this PVC on the directory /data.
- After creating the configuration, change the PersistentVolumeClaim to request a size of 200MiB.

```bash
vim resize_pvc.yaml
kubectl apply -f resize_pvc.yaml
kubectl get pv,pvc
kubectl edit pvc mypvc
# Change the storage 100Mi to 200Mi in spec.storage
# Save and close it.
```

### 13.7 Investigating Pod Logs

- Run a Pod with the name failngdb, which starts the mariadb image without any further options (it should fail).
- Investigate the Pod logs and write all lines that start with ERROR to /tmp/failingdb.log

```bash
kubectl run failngdb --image=mariadb
kubectl get pods
kubectl logs failngdb
kubectl logs failngdb | tail -2 > /tmp/failingdb.log
```

### 13.8 Analyzing Performance

- Find out which Pod currently has the highest CPU load.

```bash
kubectl top pods
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

kubectl get ns
kubectl get -pods -n kube-system
kubectl edit -n kube-system deploy metrics-server
# spec.container.args: - --kubelet-insecure-tls
kubectl get -pods -n kube-system
kubectl top podss
```

### 13.9 Managing Scheduling

- Run a Pod with the name lab139pod.
- Ensure that it only runs on nodes that have the label storage=ssd set.

```bash
kubectl label node node-worker-1 storage=ssd
kubectl run lab139pod --image=nginx --dry-ru=client -o yaml > lab139pod.yaml
vim lab139pod.yaml
spec.nodeSelector.storage: ssd

kubectl apply -f lab139pod.yaml
kubectl get pods -o wide
```

### 13.10 Configuring Ingress

- Run a Pod with the name lab1310pod, using the Nginx image.
- Expose this Pod using a NodePort type Service.
- Configure Ingress such that its web content is available on the path lab1310.info/hi
- You will not have to configure an Ingress controller for this assignment, just the API resource is enough.

```bash
kubectl run lab1310pod --image=nginx
kubectl expose pod lab1310pod --port=80 --type=NodePort
kubectl get svc

kubectl create ingress --help
kubectl create ingress simple --rule="lab1310.info/hi=lab1310pod:80"
kubectl describe ingress simple
```

### 13.11 Preparing for Node Maintenance

- Schedule node worker2 for maintenance in such a way that all running Pods are evicted.

```bash
kubectl drain worker-node-2 --ignore-daemonsets --force --delete-emptydir-data

# Bring back the node after maintenance.
kubectl uncordon worker-node-2
```

### 13.12 Scaling Applications

- Run a Deployment with the name lab1312deploy using the Nginx image.
- Scale it such that it runs 6 application instances.

```bash
kubectl create deploy lab1312deploy --image=nginx
kubectl get all --selector app=lab1312deploy
kubectl scale deployment lab1312deploy --replicas=6
kubectl get all --selector app=lab1312deploy
```
