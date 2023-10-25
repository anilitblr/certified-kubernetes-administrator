# Lesson 5 Managing Application Access

- [Lesson 5 Managing Application Access](#lesson-5-managing-application-access)
    - [5.1 Exploring Kubernetes Networking](#51-exploring-kubernetes-networking)
      - [Exploring Kubernetes Networking](#exploring-kubernetes-networking)
    - [5.2 Understanding Network Plugins](#52-understanding-network-plugins)
      - [Understanding Network Plugins](#understanding-network-plugins)
    - [5.3 Using Services to Access Applications](#53-using-services-to-access-applications)
      - [Understanding Services](#understanding-services)
      - [Configuring Services](#configuring-services)
      - [Demo: Creating Services](#demo-creating-services)
    - [5.4 Running an Ingress Controller](#54-running-an-ingress-controller)
      - [Understanding Ingress](#understanding-ingress)
      - [Demo 1: Installing the Nginx Ingress Controller](#demo-1-installing-the-nginx-ingress-controller)
      - [Demo 2: Installing the Nginx Ingress Controller](#demo-2-installing-the-nginx-ingress-controller)
    - [5.5 Configuring Ingress](#55-configuring-ingress)
      - [Managing Rules](#managing-rules)
      - [Understanding IngressClass](#understanding-ingressclass)
      - [Demo 1: Configuring Ingress Rules](#demo-1-configuring-ingress-rules)
      - [Demo 2: Configuring Ingress Rules](#demo-2-configuring-ingress-rules)
    - [5.6 Using Port Forwarding for Direct Application Access](#56-using-port-forwarding-for-direct-application-access)
      - [Using Port Forwarding](#using-port-forwarding)
    - [Lesson 5 Lab Managing Networking](#lesson-5-lab-managing-networking)
    - [Lesson 5 Lab Solution Managing Networking](#lesson-5-lab-solution-managing-networking)

### 5.1 Exploring Kubernetes Networking

#### Exploring Kubernetes Networking

In Kubernetes, networking happens at different levels.
- **Between containers**: implemented as IPC.
- **Between Pods**: implemented by network plubins.
- **Between Pods and Services**: implemented by Service resources.
- **Between external users and Services**: implemented by Services, with the help of Ingress.

### 5.2 Understanding Network Plugins

#### Understanding Network Plugins

- Network plugins are required to implemented network traffic between Pods.
- Network plugin are provided by the Kubernetes Ecosystem.
- Vanilla Kubernetes does not come with a default network plugin, and you will have to install it while installing a cluster.
- Different plugins provide different features.
- Currently, the Calico plugin is commonly used because of its support fo features like NetworkPlicy.

### 5.3 Using Services to Access Applications

#### Understanding Services

- Service resources are used to provide access to Pods.
- if multiple Pods are used a Service endpoint, the Service will load balance traffic to the Pods.
- Different types of Service can be configured.
  - **ClusterIP**: the Service is internally exposed and is reachable only from within the cluster.
  - **NodePort**: the Service is exposed at each node's Ip address as a port. The Service can be reached from outside the cluster at nodeip:nodeport
  - **LoadBalancer**: the cloud provider offers a load balancer that routes traffice to either NodePort or ClusterIP-based Services.
  - **ExternalName**: the Service is mapped to an esternal name that is implemented as a DNS CNAME record.

#### Configuring Services

- Use **kubectl expose** to expose Applications through their Pods, ReplicaSet or Deployment (recommended)
- Use **kubectl create service** as an alternative.

#### Demo: Creating Services

```bash
kubectl create deploy webshop --image=nginx --replicas=3
kubectl get pods --selector app=webshop -o wide
kubectl expose deploy webshop --type=NodePort --port=80
kubectl describe svc webshop
kubectl get svc
curl nodeip:nodeport
```

### 5.4 Running an Ingress Controller

#### Understanding Ingress

- Ingress is an API object that manages external access to services in a cluster.
- Ingress works with external DNS to provide URL-based access to Kubernetes applications.
- Ingress consists of two parts.
  - A load balancer available on the external network.
  - An API resource that contacts the Service resources to find out about available back-end Pods.
- Ingress load balancers are provided by the Kubernetes ecosystem, different load balancers are available.
- Ingress exposes HTTP and HTTPS routes from outside the cluster to Services within the cluster.
- Ingress uses the selectorlabel in Services to connect to the Pod endpoints.
- Traffic routing is controlled by rules defined on the Ingress resource.
- Ingress can be configured to do the following, according to funtionality provided by the load balancer.
  - Give Services externally-reachable URLs.
  - Load balancer traffic
  - Terminate SSL/TLS
  - Offer name based virtual hosting

#### Demo 1: Installing the Nginx Ingress Controller

```bash
helm upgrade --install ingress-nginx ingress-nginx --repo https://kubernetes.github.io/ingress-nginx --namespace ingress-nginx --create-namespace

kubectl get pods -n ingress-nginx
kubectl get all -n ingress-nginx
kubectl create deploy nginxsvc --image=nginx --port=80
kubectl expose deploy nginxsvc
kubectl get all --selector app=nginxsvc
```

#### Demo 2: Installing the Nginx Ingress Controller

```bash
kubectl create ingress -h | less
kubectl create ingress example.com --class=nginx --rule=example.com/*=nginxsvc:80
kubectl port-forward -n ingress-nginx svc/ingress-nginx-controller 8080:80
sudo echo "127.0.0.1 nginxsvc.info" >> /etc/hosts
curl nginxsvc.info:8080
kubectl get ingress
kubectl describe ingress nginxsvc
```

### 5.5 Configuring Ingress

#### Managing Rules

- Ingress rules catch incoming traffic that matches a specific path and optical hostname and connects that to a Service and port.
- Use **kubectl create ingress** to create rules.
- Different paths can be defined on the same host.
  - **kubectl create ingress myingress --rule="/myingress=myingress:80" --rule="/youringress=youringress:80"**
- Different virtual hosts can be defined in the same Ingress.
  - **kubectl create ingress nginxsvc --class=nginx --rule="nginxsvc.info/*=nginxsvc:80" --rule="otherserver.org/*=otherserver:80"**

#### Understanding IngressClass

- In one cluster, different Ingress controllers can be hosted, each with its own configuration.
- Controllers can be included in an IngressClass.
- While defining Ingress rules, the --class option should be used to implement the role on a specific Ingress controller.
  - If this option is not used, a default IngressClass must be defined.
  - Set **ingressclass.kubernetes.io/is-default-class: "true"** as an annotation on the IngressClass to make it the default.
- After creating the Ingress controller as described before, an IngressClass API resource has been created.
- Use **kubectl get ingressclass -o yaml** to investigate its content.

```bash
kubectl get ingreesclass
kubectl get ingreesclass -o yaml
kubectl edit ingreesclass nginx 

    ingressclass.kubernetes.io/is-default-class: "true" # Add this above creatingTimestamp
```

#### Demo 1: Configuring Ingress Rules

Note: this demo continues on the demo in Lession 5.3
```bash
kubectl get deployment
kubectl get svc webshop
kubectl create ingress webshop-ingress --rule="/=webshop:80" --rule="/hello=newdep:8080"
sudo vim /etc/hosts
    127.0.0.1   webshop.info
kubectl get ingress
kubectl describe ingress webshop-ingress # Error `"newdep" not found` continue to fix it.
```

#### Demo 2: Configuring Ingress Rules

```bash
kubectl create deployment newdep --image=gcr.io/google-sample/hello-app:2.0
kubectl expose deployment newdep --port=8080
kubectl describe ingress webshop-ingress
```

### 5.6 Using Port Forwarding for Direct Application Access

#### Using Port Forwarding

- **kubectl port-forward** can be used to connect to applications for analyzing and troubleshooting.
- If torwards traffic coming in to a local port on the kubectl client machine to a port that is available in a Pod.
- Using port forwarding allows you to test application access without the need to configure Services and Ingress.
- Use **kubectl port-forward mypod 1234:80** to forward local port 1234 to Pod port 80.
- To run in the background, use Ctrl-z or start with a **&** at the end of the **kubectl port-forward** command.

```bash
kubectl get pods
kubectl port-forward pods/webser-xxx-yyy 1234:80
curl localhost:1234
```

### Lesson 5 Lab Managing Networking

- Run a deployment with the name apples, using 3 replicas and the Nginx image.
- Expose this deployment in such a way that it is accessible on my.fruit
- Use port forwarding to test.

### Lesson 5 Lab Solution Managing Networking

```bash
kubectl create deploy apples --image=nginx --replicas=3
kubectl get deploy --selector app=apples
kubectl expose deploy apples --port=80
kubectl get svc apples
kubectl create ingress apples-ingress --class=nginx --rule=my.fruit/*=apples:80
sudo vi /etc/hosts
    127.0.0.1      my.fruit
kubectl get pods --selector app=apples
kubectl port-forward pod/apples-78656fd5db-pj9vd 12345:80 &
curl my.fruit:12345
```
