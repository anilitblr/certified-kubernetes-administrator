# Lesson 11 Logging, Monitoring, and Troubleshooting

- [Lesson 11 Logging, Monitoring, and Troubleshooting](#lesson-11-logging-monitoring-and-troubleshooting)
    - [11.1 Monitoring Kubernetes Resources](#111-monitoring-kubernetes-resources)
      - [Understanding Resource Monitoring](#understanding-resource-monitoring)
    - [11.2 Understanding the Troubleshooting Flow](#112-understanding-the-troubleshooting-flow)
      - [Understanding Troubleshooting Flow](#understanding-troubleshooting-flow)
    - [11.3 Troubleshooting Kubernetes Applications](#113-troubleshooting-kubernetes-applications)
      - [Troubleshooting Pods](#troubleshooting-pods)
      - [Investigating Resource Problems](#investigating-resource-problems)
    - [11.4 Troubleshooting Cluster Nodes](#114-troubleshooting-cluster-nodes)
      - [Troubleshooting Cluster Nodes](#troubleshooting-cluster-nodes)
      - [Troubleshooting Cluster Nodes](#troubleshooting-cluster-nodes-1)
    - [11.5 Fixing Application Access Problems](#115-fixing-application-access-problems)
      - [Understanding Application Access](#understanding-application-access)
    - [Lesson 11 Lab Troubleshooting Nodes](#lesson-11-lab-troubleshooting-nodes)
    - [Lesson 11 Lab Solution Troubleshooting Nodes](#lesson-11-lab-solution-troubleshooting-nodes)

### 11.1 Monitoring Kubernetes Resources

#### Understanding Resource Monitoring

- **kubectl get** can be used on any resource and shows generic resource health.
- If metrics are collected, use **kubectl top pods** and **kubectl top nodes** to get performance-related information about Pods and Nodes.

```bash
kubectl top pods 
kubectl top nodes
```

### 11.2 Understanding the Troubleshooting Flow

#### Understanding Troubleshooting Flow

- Resources are first created in the Kubernetes etcd database.
- Use **kubectl describe** and **kubectl events** to see how that has been going.
- After adding the resources to the database, the Pod application is started on the node s where it is scheduled.
- Before it can be started, the Pod image needs to be fetched.
  - Use **sudo crictl images** to get a list.
- Once the application is started, use **kubectl logs** to read the output of the application.

```bash
kubectl run testdb --image=mysql
kubectl describe pod testdb
ssh anil@worker-node-2
sudo crictl images
exit

ssh anil@control-plane
kubectl describe pod testdb
```

### 11.3 Troubleshooting Kubernetes Applications

#### Troubleshooting Pods

- The first step is to use **kubectl get**, which will give a generic overview of Pod states.
- A Pod can be in any of the following states:
  - **Pending**: the Pod has been created in etcd, buty is waiting for an eligible node.
  - **Running**: the Pod is in healthy state.
  - **Succeeded**: the Pod has done its work and there is no need to restart it.
  - **Failed**: one or more containers in the Pod have ended with an error code and will not be restarted.
  - **Unknown**: the state could not be obtained, often related to network issues.
  - **Completed**: the Pod has run to completion.
  - **CrashLoopBackOff**: one or more containers in the Pod have generated an error, but the schedulre is still trying to run them.

#### Investigating Resource Problems

- If **kubectl get** indicates that there is an issue, the next step is to use **kubectl describe** to get more information.
- **kubectl describe** shows API information about the resource and often has good indicators what is going wrong.
- If **kubectl describe** shows that a Pod has an issue starting its primary container, use **kubectl logs** to investigate the application logs.
- If the Pod is running, but not behaving as expected, open an interactive shell on the Pod for further troubleshooting: **kubectl exec -it mypod -- sh**

```bash
kubectl get pods
kubectl describe pod testdb
kubectl logs testdb # Delete the pod and recreate it with evvironment variables.
```

### 11.4 Troubleshooting Cluster Nodes

#### Troubleshooting Cluster Nodes

- Use **kubectl cluster-info** for a generic impression of cluster health.
- Use **kubectl cluster-info dump** for (too much) information coming from all the cluster log files.
- **kubectl get nodes** will give a generic overview of node health.
- **kubectl get node -n kube-system** shows Kubernetes core services running on the control node.
- **kubectl describe node nodename** shows detailed information about nodes, check the "Conditions" section for operational information.

#### Troubleshooting Cluster Nodes

- **sudo systemctl status kubelet** will show current status information about the kubelet.
- **sudo systemctl restart kubelet** allows you to restart it.
- **sudo openssl x509 -in /var/lib/kubelet/pki/kubelet.crt -text** allows you to verify kubelet certificates and verify they are still valid.
- The kube-proxy Pods are running to ensure connectivity with worker nodes, use **kubectl get pods -n kube-system** for an overview.

```bash
kubectl cluster-info
kubectl cluster-info dump
kubectl get node -n kube-system
kubectl describe node nodename
sudo systemctl status kubelet
sudo openssl x509 -in /var/lib/kubelet/pki/kubelet.crt -text
```

### 11.5 Fixing Application Access Problems

#### Understanding Application Access

- To access applications running in the Pods, Services and Ingress are used.
- The Service resource uses a selector label to connect to Pods with a matching label.
- The Ingress resource connects to a Service and picks up its selector label to connect to the backend Pods directly.
- To troubleshoot application access, check the labels in all of these resources.

```bash
kubectl get endpoints
kubectl get svc
curl <cluster-ip>
kubectl edit svc webserver
```

### Lesson 11 Lab Troubleshooting Nodes

- Use the appropriate tools to find out if the cluster nodes are in good health.

### Lesson 11 Lab Solution Troubleshooting Nodes

```bash
kubectl get nodes
kubectl describe node nodename | less
ssh anil@workder-node-1
sudo systemctl status kubelet
sudo systemctl status containerd
```
