# Lesson 4 Managing Storage

- [Lesson 4 Managing Storage](#lesson-4-managing-storage)
    - [4.1 Understanding Kubernetes Storage Options](#41-understanding-kubernetes-storage-options)
    - [4.2 Accessing Storage Through Pod Volumes](#42-accessing-storage-through-pod-volumes)
      - [Using Pod Volumes](#using-pod-volumes)
    - [4.3 Configuring Persistent Volume (PV) Storage](#43-configuring-persistent-volume-pv-storage)
      - [Managing Persistent Volumes](#managing-persistent-volumes)
    - [4.4 Configuring PVCs](#44-configuring-pvcs)
      - [Configuring PersistentVolumeClain](#configuring-persistentvolumeclain)
    - [4.5 Configuring Pod Storage with PV and PVCs](#45-configuring-pod-storage-with-pv-and-pvcs)
    - [4.6 Using StorageClass](#46-using-storageclass)
      - [Understanding StorageClass](#understanding-storageclass)
      - [Using StorageClass](#using-storageclass)
    - [4.7 Understanding Storage Provisioners](#47-understanding-storage-provisioners)
      - [Using an NFS Storage Provisioner](#using-an-nfs-storage-provisioner)
      - [Understanding Requirements](#understanding-requirements)
      - [Demo: Configuring a Storage Provisioner](#demo-configuring-a-storage-provisioner)
      - [Demo: Configuring a Storage Provisioner](#demo-configuring-a-storage-provisioner-1)
      - [Demo: Creating the PVC](#demo-creating-the-pvc)
      - [Demo: Understanding Default StorageClass](#demo-understanding-default-storageclass)
    - [4.8 Using ConfigMaps and Secrets as Volumes](#48-using-configmaps-and-secrets-as-volumes)
      - [Understanding ConfigMap](#understanding-configmap)
      - [Demo: Creating a ConfigMap](#demo-creating-a-configmap)
    - [Lesson 4 Lab Setting up Storage](#lesson-4-lab-setting-up-storage)
    - [Lesson 4 Lab Solution Setting up Storage](#lesson-4-lab-solution-setting-up-storage)

### 4.1 Understanding Kubernetes Storage Options

- Explanation

### 4.2 Accessing Storage Through Pod Volumes

#### Using Pod Volumes

- Pod Volumes are a part of the Pod sepcification and have the storage reference hard coded in the Pod manifest.
- This is not bad, but it doesn't allow for flexible storage allocation.
- Pod Volumes can be used for any storage type.
- Also, the ConfigMap can be used to mount Pod Volumes
```bash
kubectl explain pod.spec.volumes | less
vim labs/morevolumes.yaml
kubectl apply -f morevolumes.yaml
kubectl get pods
kubectl describe pods morevol
kubectl exec -it morevel -c centos1 -- touch /centos1/centos1file
kubectl exec -it morevel -c centos2 -- ls /centos2
```

### 4.3 Configuring Persistent Volume (PV) Storage

#### Managing Persistent Volumes

- PersistentVolumes (PV) are an API resource that represents specific storage.
- PVs can be created manually, or automatically using StorageClass and storage provisioners.
- Pods do not connect to PVs directly, but indirectly using PersistentVolumeClain (PVC)

```bash
vim labs/pv.yaml
kubectl apply -f labs/pv.yaml
kubectl get pv
kubectl get pv -o yaml
```

### 4.4 Configuring PVCs

#### Configuring PersistentVolumeClain

- PVCs allows Pods to connect to any type of storage that is provided at a specific site.
- Site-specific storage needs to be created as a PersistenVolume, either manually or automatically using StorageClass.
- Behind StorageClass a storage provisioner is required.

```bash
vim labs/pvc.yaml
kubectl apply -f labs/pvc.yaml
kubectl get pvc
kbuectl describe pvc pv-claim
vim labs/pvc.yaml
spec.storageClassName: demo

kubectl delete -f labs/pvc.yaml
kubectl apply -f labs/pvc.yaml
kubectl get pvc,pv
```

### 4.5 Configuring Pod Storage with PV and PVCs

```bash
vi pv-pod.yaml
kubectl apply -f pv-pod.yaml
kubectl get pvc,pv
kubectl exec -it pv-pod -- touch /usr/share/nginx/html/hellothere
kubectl describe pv
kubectl get pods pv-pod -o wide
ssh anil@worker-node-2
ls /mydata
```

### 4.6 Using StorageClass

#### Understanding StorageClass

- StorageClass is an API resource that allows storage to be automatically provisioned.
- StorageClass can also be used as property that connectcs PVC and PV without using an actual StorageClass resource.
- Multiple StorageClass resources can co-exist in the same cluster to provide access to different types of storage.
- For automatic working, one StorageClass must be set as default.
  - **kubectl patch storageclass mysc -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'**

#### Using StorageClass

- To enable automatic provisioning, StorageClass needs a backing storage provisioner.
- In the PV and PVC definition, a storageClass property can be set to connect to a specific StorageClass which is useful if multiple StorageClass resources are available.
- If the storageClass property is not set, the PVC will get storage from the default StorageClass.
- If also no default StorageClass is set, the PVC will get stuck in a status of Pending.

### 4.7 Understanding Storage Provisioners

#### Using an NFS Storage Provisioner

- The Storage Provisioner works with a StorageClass to automatically provide storage.
- It runs as a Pod in the kubernetes cluster, provided with access control configured through Roles, RoleBinding, and ServiceAccounts.
- Once operational, you don't have to manually create PersistentVolumes anymore.

#### Understanding Requirements

- To create a storage provisioner, access permissions to the API are required.
- Roles and RoleBindings are created to provide these permissions.
- A ServiceAccount is created to connect the Pod the appropriate RoleBinding.
- More information about Role Based Access Control (RBAC) is in Lesson 10.3

#### Demo: Configuring a Storage Provisioner

```bash
# On control-plane
sudo apt install nfs-server -y
sudo mkdir /nfsexport
sudo sh -c 'echo "/nfsexport *(rw,no_root_squash)" >> /etc/exports'
sudo systemctl restart nfs-server

# On worker nodes
sudo apt install nfs-client -y
showmount -e control-plane
```

#### Demo: Configuring a Storage Provisioner

- Fetch the **helm** binary from **https://github.com/helm/helm/releases** and install it to **/usr/local/bin**
- Add the helm repo using **helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner**
- Install the package using **helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner --set nfs.server=control-plane.lab.com --set nfs.path=/nfs-subdir-external-provisioner**
- Use **kubectl get pods** to verify that the nfs-subdir-provisioner Pod is running.
```bash
wget https://get.helm.sh/helm-v3.12.3-linux-amd64.tar.gz
tar -xzf helm-v3.12.3-linux-amd64.tar.gz
sudo cp linux-amd64/helm /usr/local/bin/
helm version

helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/

helm repo list

helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner --set nfs.server=192.168.1.200 --set nfs.path=/nfsexport 

kubectl get pods
```

#### Demo: Creating the PVC

- Type **kubectl get pv** to verify that currently no PVs are available.
- Use **kubectl apply -f nfs-provisioner-pvc-test.yaml** to create a PVC.
- Use **kubectl get pvc,pv** to verify the PVC is created and bound to an automatically created PV.
- Create any Pod that mounts the PVC storage, and verify data end up in the NFS share.

```bash
kubectl get pv
kubectl get storageclass
vi nfs-provisioner-pvc-test.yaml
kubectl apply -f nfs-provisioner-pvc-test.yaml
kubectl get pvc,pv
```

#### Demo: Understanding Default StorageClass

```bash
vi another-pvc-test.yaml
kubectl apply -f another-pvc-test.yaml
# will show pending, there is no default StorageClass
kubectl get pvc
kubectl describe pvc another-nfs-pvc-test
kubectl get storageclasses.storage.k8s.io -o yaml

kubectl patch storageclass nfs-client -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'

# will work now
kubectl get pvc 

ls -l /nfsexport # output 2 directories.
```

### 4.8 Using ConfigMaps and Secrets as Volumes

#### Understanding ConfigMap

- A ConfigMap is an API resource used to store site-specific data.
- A Secret is a base64 encoded ConfigMap.
- ConfigMap are used to store either environment variables, startup parameters or configuration files.
- When a Configuration File is used in a ConfigMap or Secret, it is mounted as a volume to provide access to its contents.

#### Demo: Creating a ConfigMap

```bash
echo "hello world" > index.html
kubectl create cm webindex --from-file=index.html
kubectl describe cm webindex
kubectl create deploy webserver --image=nginx
kubectl edit deploy webserver

spec.template.spec
volumes:
- name: cmvol
  configMap:
    name: webindex

spec.template.spec.containers
volumeMounts:
- mountPath: /usr/share/nginx/html
  name: cmvol

kubectl get deploy
kubectl exec webserver-xxx-yyy -- cat /usr/share/nginx/html/index.html
```

### Lesson 4 Lab Setting up Storage

- Create a PersistentVolume, using the HostPath storage type to access the directory /storage
- Create a file /storage/index.html, containing the text "hello lab4"
- Run a Pod that uses an Nginx image and mounts the HostPath storage on the directory /usr/share/ginx/html
- On the running Pod, use **kubectl exec** to verify the existence of the file /usr/share/nginx/html

### Lesson 4 Lab Solution Setting up Storage

```bash
sudo mkdir /storage # Create it on worker-node-{1, 2}
sudo vim /storage/index.html # Add some content hello worker {1, 2}

# Create PV

# Open - **https://kubernetes.io/docs/**
# Search for **persistent volume**
# Click **Configure a Pod to Use a PersistentVolume for Storage | Kubernetes**
# Copy **Create a PersistentVolume** YAML code.
vi lab4-pv.yaml
# ```yaml
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: lab4-volume
  labels:
    type: local
spec:
  storageClassName: lab4
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/storage"
# ```

kubectl apply -f lab4-pv.yaml
kubectl describe pv lab4-volume

# Create PVC

# Copy **Create a PersistentVolumeClaim** YAML code.
vim lab4-pvc.yaml 
# ```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: lab4-claim
spec:
  storageClassName: lab4
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
# ```
kubectl apply -f lab4-pvc.yaml 
kubectl get pv,pvc

# Create Pod

# Copy **Create a Pod** YAML code.
vim lab4-pod.yaml 

# ```yaml
apiVersion: v1
kind: Pod
metadata:
  name: lab4-pod
spec:
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
        claimName: lab4-claim
  containers:
    - name: task-pv-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: task-pv-storage
#```
kubectl apply -f lab4-pod.yaml
kubectl get pods

kubectl exec lab4-pod -- cat /usr/share/nginx/html/index.html
```
