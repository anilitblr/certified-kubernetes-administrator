# Lesson 10 Managing Security Settings

- [Lesson 10 Managing Security Settings](#lesson-10-managing-security-settings)
    - [10.1 Understanding API Access](#101-understanding-api-access)
    - [10.2 Managing SecurityContext](#102-managing-securitycontext)
      - [Understanding SecurityContext](#understanding-securitycontext)
      - [Setting SecurityContext](#setting-securitycontext)
      - [Demo: Setting securityContext](#demo-setting-securitycontext)
    - [10.3 Using ServiceAccounts to Configure API Access](#103-using-serviceaccounts-to-configure-api-access)
      - [Understanding Kubernetes Users](#understanding-kubernetes-users)
    - [10.4 Setting Up Role Based Access Control (RBAC)](#104-setting-up-role-based-access-control-rbac)
      - [Configuring Roles](#configuring-roles)
      - [Creating RoleBinding](#creating-rolebinding)
      - [Creating ServiceAccounts](#creating-serviceaccounts)
      - [Demo 1: Configuring ServiceAccounts](#demo-1-configuring-serviceaccounts)
      - [Demo 2: Configuring ServiceAccounts](#demo-2-configuring-serviceaccounts)
    - [10.5 Configuring Cluster Roles and RoleBindings](#105-configuring-cluster-roles-and-rolebindings)
      - [Understanding ClusterRoles](#understanding-clusterroles)
    - [10.6 Creating Kubernetes User Accounts](#106-creating-kubernetes-user-accounts)
      - [Understanding User Accounts](#understanding-user-accounts)
      - [Demo: Creating User Accounts](#demo-creating-user-accounts)
    - [Lesson 10 Lab Managing Security](#lesson-10-lab-managing-security)
    - [Lesson 10 Lab Solution Managing Security](#lesson-10-lab-solution-managing-security)

### 10.1 Understanding API Access

- Explanation.

### 10.2 Managing SecurityContext

#### Understanding SecurityContext

A SecurityContext defines provilege and access control settings for Pods or containers and can include the following.
- UID and GID based Discretionary Access Control.
- SELinux security labels.
- Linux Capabilities.
- Seccomp.
- The AllowPrivilegeEscalation setting.
- The runAsNonRoot setting.

#### Setting SecurityContext

- SecurityContext can be set at Pod level as well as container level.
- See **kubectl cxplain pod.spec.securityContext**
- Settings applied at the container level will overwrite settings applied at the Pod level.

#### Demo: Setting securityContext

```bash
kubectl explain pod.spec.securityContext | less
kubectl explain pod.spec.containers.securityContext | less
cd labs/
vi security-context.yaml
kubectl apply -f security-context.yaml
kubectl get pod security-context-demo
kubectl exec -it security-context-demo -- sh

    ps # will show processes running UID 1000
    cd /data; ls -l # will show fsGroup and owning GID
    ls -l
    cd demo
    touch file
    ls -l file # Look at the UID and GID(1000 and 2000)
    id 
    exit
```

### 10.3 Using ServiceAccounts to Configure API Access

#### Understanding Kubernetes Users

- The Kubernetes API doesn't define users for people to authenticate and authorize.
- Users are obtained externally.
  - Defined by X.509 certificates.
  - Obtained from external OpenID-based authentication (Google, AD and many more).
- ServiceAccounts are used to authorize Pods to get access to sepcific API resources.

```bash
kubectl get sa
kubctl get sa default -o yaml
kubectl get sa -A
kubectl get pods
kubectl get pod nfs-xxx-yyy -o yaml | less # Look for serviceAccount:
```

### 10.4 Setting Up Role Based Access Control (RBAC)

#### Configuring Roles

- Roles are used on Namespaces and use Verbs to specify access to specific resources in that Namespaces.
- Use **kubectl create role** to create role

```bash
kubectl create role -h | less
kubectl get roles
kubectl get role leader-locking-nfs-subdir-external-provisioner -o yaml
kubectl get roles -A
```

#### Creating RoleBinding

- RoleBindings connect users or ServiceAccounts to Roles.
- Use **kubectl create rolebinding** to create it.

```bash
kubectl get rolebinding
kubectl get rolebinding -A 
kubectl get rolebinding -n 
kubectl get rolebindings.rbac.authorization.k8s.io -n ingress-nginx ingress-nginx -o yaml 
kubectl create rolebinding -h | less
```

#### Creating ServiceAccounts

- A ServiceAccount is used to authorize Pods to get informtion from the API.
- All Pods have a default ServiceAccount which provides minimal access.
- If more access is needed, specific ServiceAccounts can be created.
- ServiceAccounts don't have specific configuration, they are used in RoleBinding to get access to specific Roles.

#### Demo 1: Configuring ServiceAccounts

```bash
# 1. Creaet a Pod, using the standard ServiceAccount
cd labs/
vi mypod.yaml
kubectl apply -f mypod.yaml

# 2. Use to check current SA configuration.
kubectl get pods mypod -o yaml # Look for serviceAccount:

# 3. Access the Pod using and try to list Pods using curl on the API.
kubectl exec -it mypod -- sh
  apk add --update curl
  curl https://kubernetes/api/v1 --insecure # will be forbidden.

# 4. Use the default ServiceAccount token and try again.
TOKEN=$(cat /run/secrets/kubernetes.io/serviceaccount/token)
echo $TOKEN
curl -H "Authorization: Bearer $TOKEN" https://kubernetes/api/v1/ --insecure

# 5. Try the same, but this time to list Pods - it will fail.
curl -H "Authorization: Bearer $TOKEN" https://kubernetes/api/v1/namespaces/default/pods/ --insecure # will be forbidden.
exit
```

#### Demo 2: Configuring ServiceAccounts

```bash
# 1. Create a ServiceAccount
vi mysa.yaml
kubectl apply -f mysa.yaml

# 2. Define a role that allows to list all Pods in the default NameSpace
vi list-pods.yaml
kubectl apply -f list-pods.yaml

# 3. Define a RoleBinding that binds the mysa to the Role just created.
vi list-pods-mysa-binding.yaml
kubectl apply -f list-pods-mysa-binding.yaml

# 4. Create a Pod that uses the mysa SA to access this Role.
vi mysapod.yaml
kubectl apply -f mysapod.yaml

# 5. Access the Pod, use the mysa ServiceAccount token and try again.
kubectl exec -it mysapod -- sh
  apk add --update curl
  TOKEN=$(cat /run/secrets/kubernetes.io/serviceaccount/token)
  echo $TOKEN
  curl -H "Authorization: Bearer $TOKEN" https://kubernetes/api/v1/ --insecure 

# 6. Try the same, but this time to list Pods
curl -H "Authorization: Bearer $TOKEN" https://kubernetes/api/v1/namespaces/default/pods/ --insecure # it works.

exit
```

### 10.5 Configuring Cluster Roles and RoleBindings

#### Understanding ClusterRoles

- Roles have a Namespaces scope, ClusterRoles apply to the entire cluster.
- The working is similar to the working of Roles.
- To provide access to ClusterRoles, use users or ServiceAccounts and provide access through a ClusterRolesBinding.

```bash
kubectl get clusterrole
kubectl get clusterrole | wc -l
kubectl get clusterrole edit -o yaml
kubectl get clusterrolebindings
```

### 10.6 Creating Kubernetes User Accounts

#### Understanding User Accounts

- Kubernetes has no User objects.
- User accounts consist of an authorized certificate that is completed with some authorization as defined in RBAC.
- To create a user account, the following stesp need to be performed.
  - Create a public/private key pair.
  - Create a Certificate Signing Request.
  - Sign the Certificate.
  - Create a configuration file that uses these keys to access the K8s cluster.
  - Create an RBAC Role.
  - Create an RBAC RoleBinding.

#### Demo: Creating User Accounts

- Step 1: Create a user working environment

  ```bash
  kubectl create ns students
  kubectl create ns staff
  kubectl config get-contexts
  ```

- Step 2: Create the User account

  ```bash
  sudo useradd -m -G sudo -s /bin/bash development
  echo -e "password\npassword" | sudo passwd development
  su - development
  openssl genrsa -out development.key 2048
  ls

  openssl req -new -key development.key \
  -out development.csr \
  -subj "/CN=development/O=k8s"
  
  ls

  sudo openssl x509 -req -in development.csr \
  -CA /etc/kubernetes/pki/ca.crt \
  -CAkey /etc/kubernetes/pki/ca.key \
  -CAcreateserial \
  -out development.crt \
  -days 1800

  ls -l
  ```

- Step 3: Update the Kubernetes Credentials Files for the new user

  ```bash
  mkdir /home/development/.kube
  sudo cp -i /etc/kubernetes/admin.conf /home/development/.kube/config
  sudo chown -R development.development /home/development/.kube
  
  kubectl config set-credentials development \
  --client-certificate=/home/development/development.crt \
  --client-key=/home/development/development.key
  ```

- Step 4: Create a Default Context for the new user

  ```bash
  kubectl config set-context development-context \
  --cluster=kubernetes \
  --namespace=staff \
  --user=development

  kubectl config use-context development-context # will set context permanently
  kubectl config get-contexts
  kubectl get pods # will fail as no RBAC has been configured yet.
  ```

- Step 5: Configure RBAC to define a staff role

  ```bash
  su - anil
  cd labs/
  vi staff-role.yaml
  kubectl apply -f staff-role.yaml
  ```

- Step 6: Bind a user to the new role

  ```bash
  vim rolebind.yaml
  kubectl apply -f rolebind.yaml # Change User.name: staff to development
  ```

- Step 7: Test it

  ```bash
  su - development
  kubectl config view
  kubectl create deployment nginx --image=nginx
  kubectl get pods
  ```

- Step 8: Create a View-only Role for development user

  ```bash
  su - development
  kubectl get pods -n default

  vi students-role.yaml
  kubectl apply -f students-role.yaml

  vi rolebindstudents.yaml # Change subjects.kind.User.name: to development
  kubectl apply -f rolebindstudents.yaml

  kubectl get pods -n default
  ```

### Lesson 10 Lab Managing Security

- Create a Role that allows for viewing of Pods in the deafult namespace.
- Configure a RoleBinding that allows all authenticated users to use this role.

### Lesson 10 Lab Solution Managing Security

```bash
kubectl create role -h | less

# Create role calld defaultpodviewer
kubectl create role defaultpodviewer --verb=get --verb=list --verb=watch --resource=pods -n default
kubectl get clusterrolebindings
kubectl get pods --as system:basic-user # List Pods using system:basic-user

# Create a rolebinding called defaultpodviwer
kubectl create rolebinding defaultpodviwer --role=defaultpodviewer --user=system:basic-user -n default
kubectl get pods --as system:basic-user
```
