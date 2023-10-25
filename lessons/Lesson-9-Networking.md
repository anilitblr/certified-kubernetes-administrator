# Lesson 9 Networking

- [Lesson 9 Networking](#lesson-9-networking)
    - [9.1 Managing the CNI and Network Plugins](#91-managing-the-cni-and-network-plugins)
      - [Understanding the CNI](#understanding-the-cni)
      - [Exploring CNI Configuration](#exploring-cni-configuration)
    - [9.2 Understanding Service Auto Registration](#92-understanding-service-auto-registration)
      - [Understanding Service Auto Registration](#understanding-service-auto-registration)
      - [Accessing Service in other Namespaces](#accessing-service-in-other-namespaces)
      - [Demo: Accessing Services by Name](#demo-accessing-services-by-name)
      - [Demo: Accessing Pods in other Namespaces](#demo-accessing-pods-in-other-namespaces)
    - [9.3 Using Network Policies to Manage Traffic Between Pods](#93-using-network-policies-to-manage-traffic-between-pods)
      - [Understanding NetworkPolicy](#understanding-networkpolicy)
      - [Using NetworkPolicy Identifiers](#using-networkpolicy-identifiers)
      - [Demo: Exploring NetworkPolicy](#demo-exploring-networkpolicy)
    - [9.4 Configuring Network Policies to Manage Traffic Between Namespaces](#94-configuring-network-policies-to-manage-traffic-between-namespaces)
      - [Applying NetworkPolicy to Namespaces](#applying-networkpolicy-to-namespaces)
      - [Demo 1: Using NetworkPolicy between Namespaces](#demo-1-using-networkpolicy-between-namespaces)
      - [Demo 2: Using NetworkPolicy between Namespaces](#demo-2-using-networkpolicy-between-namespaces)
    - [Lesson 9 Lab Using NetworkPolicies](#lesson-9-lab-using-networkpolicies)
    - [Lesson 9 Lab Solution Using NetworkPolicies](#lesson-9-lab-solution-using-networkpolicies)

### 9.1 Managing the CNI and Network Plugins

#### Understanding the CNI

- The Container Network Interface (CNI) is the common interface used for networking when starting kubelet on a worker node.
- The CNI doesn't take care of networking, that is done by the network plugin.
- CNI ensures the pluggable nature of networking, and make it easy to select between different network plugins provided by the ecosystem.

#### Exploring CNI Configuration

- The CNI plugin configuration is in /etc/cni/net.d
- Other plugins have generic settings, and are using additional configuration.
- Often, the additional configuration is implemented by Pods.
- Generic CNI documentatioin is on https://github.com/containernetworking/cni

```bash
sudo -i
cd /etc/cni/net.d/
cat calico-kubeconfig
cat 10-calico.conflist
kubectl get ns
kubectl get all -n kube-system
ps aux | grep api
```

### 9.2 Understanding Service Auto Registration

#### Understanding Service Auto Registration

- Kubernetes runs the coredns Pods in the kube-system Namespace as internal DNS servers.
- These Pods are exposed by the kubedns Service.
- Service register with this kubedns Service.
- Pods are automatically configured with the IP address of the kubedns Service as their DNS resolver.
- As a result, all Pods can access all Services by name.

#### Accessing Service in other Namespaces

- If a Service is running in the same Namespace, it can be reached by the short hostname.
- If a Service is running in another Namespace, an FQDN consisting of servicename.namespace.svc.clustername must be used.
- The clustername is defined in the coredns Correfile and set to cluster.local if it hasn't been changed, use **kubectl get cm -n kube-system coredns -o yaml** to verify.

#### Demo: Accessing Services by Name

```bash
kubectl get cm -n kube-system coredsn -o yaml
kubectl run webserver --image=nginx
kubectl expose pod webserver --port=80
kubectl run testpod --image=busybox -- sleep 3600
kubectl get svc
kubectl exec -it testpod -- wget webserver
```

#### Demo: Accessing Pods in other Namespaces

```bash
kubectl create ns remote
kubectl run internginx --image=nginx
kubectl run remotebox --image=busybox -n remote -- sleep 3600
kubectl expose pod internginx --port=80
kubectl exec -it remotebox -n remote -- cat /etc/resolv.conf
kubectl exec -it remotebox -n remote -- nslookup internginx # fails
kubectl exec -it remotebox -n remote -- nslookup
kubectl exec -it remotebox -n remote -- nslookup internginx.default.svc.cluster.local
```

### 9.3 Using Network Policies to Manage Traffic Between Pods

#### Understanding NetworkPolicy

- By default, there are no restrictions to network traffic in K8s.
- Pods can always communicate, even if they're in other Namespaces.
- To limit this, NetworkPolicies can be used.
- NetworkPolicies need to be supported by the network plugin though.
  - The weave plugin does NOT support NetworkPolicies.
- If in a policy there is no match, traffic will be denied.
- If no NetworkPolicy is used, all traffic is allowd.

#### Using NetworkPolicy Identifiers

- In NetworkPolicy, three different identifiers can be used.
  - **Pods**: (podSelector) note that a Pod cannot block access to itself.
  - **Namespaces**: (namespaceSelector) to grant access to specific Namespaces.
  - **IP blocks**: (ipBlock) notice that traffic to and from the node where a Pod is runnign is always allowed.
- When defining a Pod or Namespace-based NetworkPolicy, a selector label is used to specify what traffic is allowed to and from the Pods that match the selector.
- NetworkPolicies do not conflict, they are additive.

#### Demo: Exploring NetworkPolicy

```bash
cd labs/
vim nwpolicy-complete-example.yaml
kubectl apply -f nwpolicy-complete-example.yaml
kubectl expose pod nginx --port=80
kubectl exec -it busybox -- wget --spider --timeout=1 nginx # will fail 
kubectl label pod busybox access=true
kubectl exec -it busybox -- wget --spider --timeout=1 nginx # will work.
kubectl describe networkpolicy access-nginx
```

### 9.4 Configuring Network Policies to Manage Traffic Between Namespaces

#### Applying NetworkPolicy to Namespaces

- To apply a NetworkPolicy to a Namespace, use **-n namespace** in the definition of the NetworkPolicy.
- To allow ingress and egress traffic, use the namespaceSelector to match the traffic.

#### Demo 1: Using NetworkPolicy between Namespaces

```bash
kubectl create ns nwp-namespace
vi nwp-lab9-1.yaml
kubectl create -f nwp-lab9-1.yaml
kubectl expose pod nwp-nginx --port=80
kubectl exec -it nwp-busybox -n nwp-namespace -- wget --spider --timeout=1 nwp-nginx # gives a bad address error.
kubectl exec -it nwp-busybox -n nwp-namespace -- nslookup nwp-nginx # explains that it's looking in the wrong ns.
kubectl exec -it nwp-busybox -n nwp-namespace -- wget --spider --timeout=1 nwp-nginx.default.svc.cluster.local # is allowed.
```

#### Demo 2: Using NetworkPolicy between Namespaces

```bash
vi nwp-lab9-2.yaml
kubectl create -f nwp-lab9-2.yaml
kubectl exec -it nwp-busybox -n nwp-namespace -- wget --spider --timeout=1 nwp-nginx.default.svc.cluster.local # is not allowed.
kubectl create deployment busybox --image=busybox -- sleep 3600
kubectl exec -it busybox[TAB] -- wget --spider --timeout=1 nwp-nginx

# Clean up
kubectl delete -f nwp-lab9-2.yaml
kubectl exec -it nwp-busybox -n nwp-namespace -- wget --spider --timeout=1 nwp-nginx.default.svc.cluster.local # will work.
```

### Lesson 9 Lab Using NetworkPolicies

- Run a webserver with the name lab9server in Namespace **restricted**, using the Nginx image and ensure it is exposed by a Service.
- From the default Namespace start two Pods: sleepybox1 and sleepybox2, each based on the busybox image using the **sleep 3600** command as the command.
- Create a NetworkPolicy that limits Ingress traffic to restricted, in such a way that only the sleepybox1 Pod from the default Namespace has access and all other access is forbidden.

### Lesson 9 Lab Solution Using NetworkPolicies

```bash
vi lesson9lab.yaml
kubectl apply -f lesson9lab.yaml
kubectl get all -n restricted
kubectl expose pod lab9server -n restricted --port=80
kubectl get all -n restricted
kubectl exec -it sleepybox1 -- wget --spider --timeout=1 lab9server.restricted.svc.cluster.local # is allowed
kubectl exec -it sleepybox2 -- wget --spider --timeout=1 lab9server.restricted.svc.cluster.local # is not allowed.

# Help
# - Open https://kubernetes.io/docs
# Search for **networkpolicy**
# Click **Network Policies | Kubernetes**
# Sample code of **The NetworkPolicy resource**
```
