# CORE CONCEPTS

## What is Kubernetes?

Kubernetes is an open-source container orchestration platform that automates the deployment, scaling, and management of containerized applications.

## Kubernetes Cluster Architecture

[What is Kubernetes Architecture? | VMware Glossary](https://www.vmware.com/in/topics/glossary/content/kubernetes-architecture.html)

![K8-Architecture-1024x580](https://github.com/anshumanrana331/CKA-Guide-2023/assets/56511928/dda4e537-cdfe-4571-81d6-12a57085054e)

To summarise, we have Master and Worker Nodes.

On the Master node, we have:

- ETCD which is key-value datastore that stores information about the cluster.
- Kube-Scheduler which schedules containers on Nodes.
- Kubernetes Controller Manager which manages several controllers to maintain the desired state of resources within the cluster. It manages different controllers like Node Controller, Replication Controller etc.
- Kube API Server which orchestrates all operations within the cluster.

On the Worker node, we have:

- Kubelet which listens for instructions from Kube API server and manages containers.
- Kube Proxy which helps in enabling communication between services within the cluster.

## CRI

CRI (Container Runtime Interface) is an abstraction layer that defines the standard interface between the Kubernetes kubelet and the container runtime, enabling Kubernetes to work with different container runtimes without modifications to its core components. For ex. Docker, ContainerD, CriO etc.

## ETCD

Etcd in Kubernetes is a distributed key-value store used as the primary data store for storing and managing the cluster's configuration data, state, and metadata. It acts as a reliable and consistent storage solution for coordination and communication among the various components of a Kubernetes cluster. It is SOURCE OF TRUTH for the cluster.

The default port used by the Etcd service in Kubernetes is 2379/tcp for client communication and 2380/tcp for server-to-server communication.

**************************************************************************************Deploying ETCD in K8s cluster (built from scratch) :************************************************************************************** Download binaries, install and run it as a service
**************************************************************************************Deploying ETCD in K8s cluster (built from Kubeadm) :************************************************************************************** Installed as a pod

The **`etcdctl`** command-line tool provides an interface for interacting with the Etcd cluster. etcdctl can interact with ETCD Server using 2 API versions – Version 2 and Version 3.  By default it’s set to use Version 2.

For example, ETCDCTL version 2 supports the following commands:

`etcdctl backup`

`etcdctl cluster-health`

`etcdctl mk`

`etcdctl mkdir`

`etcdctl set`

Whereas the commands are different in version 3

`etcdctl snapshot save`

`etcdctl endpoint health`

`etcdctl get`

`etcdctl put`

To set the right version of API set the environment variable ETCDCTL_API command

`export ETCDCTL_API=3`

When the API version is not set, it is assumed to be set to version 2. And version 3 commands listed above don’t work. When API version is set to version 3, version 2 commands listed above don’t work.

## Kube-API Server

The kube-apiserver is a core component of Kubernetes that serves as the API endpoint for cluster management. It acts as the front-end for the Kubernetes control plane, accepting API requests from various sources such as kubectl (the command-line interface) and other Kubernetes components. The kube-apiserver validates and processes these requests, ensuring the desired state of the cluster and communicating with other control plane components to make the necessary changes. In summary, the kube-apiserver provides the central API interface for interacting with and managing a Kubernetes cluster.

**************************************************************************************Deploying Kube-APIserver in K8s cluster (built from scratch) :************************************************************************************** Download binaries, install and run it as a service

```bash
# View api-server options:
cat /etc/systemd/system/kube-apiserver.service

# Running processes can be seen by listing process on master node:
ps -aux | grep kube-apiserver
```

**************************************************************************************Deploying Kube-APIserver in K8s cluster (built from Kubeadm) :************************************************************************************** Installed as a pod

```bash
# View api-server options -kubeadm: 
cat /etc/kubernetes/manifests/kube-apiserver.yaml

# Running processes can be seen by listing process on master node:
ps -aux | grep kube-apiserver
```

## Kube Controller Manager

The kube-controller-manager is a core component of Kubernetes responsible for managing various controllers that ensure the desired state of resources within the cluster. It monitors the state of objects, such as pods, deployments, and services, and takes actions to reconcile any differences between the desired and actual state. The kube-controller-manager runs as a process on the Kubernetes master node and helps maintain the overall cluster state and enforce cluster-level policies. Some of the controllers managed by kube-controller-manager are:

- **Node-Controller :** Checks status of nodes every 5 seconds also called Node Monitor period. If a node is unreachable for 40s (Node Monitor Grace period), it is marked as unreachable, after which it waits for 5min (POD Eviction Timeout) to remove the node and provisions the PODS on the healthy nodes.
- **Replication-Controller :** Monitors status of replica sets and ensuring that desired no of PODs are available at all times within the set.

**************************************************************************************Deploying Kube-Controller Manager in K8s cluster (built from scratch) :************************************************************************************** Download binaries, install and run it as a service

```bash
# View kube-controller options:
cat /etc/systemd/system/kube-controller-manager.service

# Running processes can be seen by listing process on master node:
ps -aux | grep kube-controller-manager
```

**************************************************************************************Deploying Kube-Controller Manager in K8s cluster (built from Kubeadm) :************************************************************************************** Installed as a pod

```bash
# View kube-controller options -kubeadm:
cat /etc/kubernetes/manifests/kube-controller-manager.yaml

# Running processes can be seen by listing process on master node:
ps -aux | grep kube-controller-manager
```

## Kube Scheduler

The kube-scheduler in Kubernetes is responsible for assigning pods to nodes in the cluster based on resource requirements, constraints, and policies. It ensures efficient utilization of resources and optimal distribution of workloads.

When determining which pod should run on which node, the kube-scheduler follows a two-step process:

1. Filtering: The kube-scheduler applies filters to eliminate nodes that are ineligible for hosting the pod. These filters consider factors such as node capacity, resource constraints, affinity, anti-affinity rules, and taints/tolerations.
2. Scoring: The remaining nodes after filtering are assigned scores based on various factors, including resource availability, pod affinity/anti-affinity rules, inter-pod communication requirements, and other user-defined constraints. The node with the highest score is selected as the preferred node for running the pod.

By combining filtering and scoring, the kube-scheduler intelligently determines the most suitable node for each pod, taking into account both technical and policy considerations.

**NOTE:**
Kube Scheduler is responsible for deciding which pod goes on which node. It does not place the pod on that node, that’s the job of **Kubelet**. 

**************************************************************************************Deploying Kube-Scheduler in K8s cluster (built from scratch) :************************************************************************************** Download binaries, install and run it as a service

```bash
# View kube-scheduler options:
cat /etc/systemd/system/kube-scheduler.service

# Running processes can be seen by listing process on master node:
ps -aux | grep kube-scheduler
```

**************************************************************************************Deploying Kube-Scheduler in K8s cluster (built from Kubeadm) :************************************************************************************** Installed as a pod

```bash
# View kube-scheduler options -kubeadm:
cat /etc/kubernetes/manifests/kube-scheduler.yaml

# Running processes can be seen by listing process on master node:
ps -aux | grep kube-scheduler
```

## Kubelet

The Kubelet in the worker node, registers the node with the Kubernetes cluster. When it receives instructions from kube-api server, to load a container or a POD on the node, it requests the container run time engine (CRI) to pull the required image and run an instance. The kubelet then continues to monitor the state of the POD and the containers in it and reports to the kube-api server on a timely basis.

**************************************************************************************Deploying Kubelet in K8s cluster (built from Kubeadm) :************************************************************************************** Unlike deploying other components, the Kubelet is not automatically deployed as a pod. Therefore, it is necessary to manually install the Kubelet on worker nodes, similar to deploying it in a K8s cluster built from scratch.

**************************************************************************************Deploying Kubelet in K8s cluster (built from scratch) :************************************************************************************** Download binaries, install and run it as a service

```bash
# View kubelet options:
cat /etc/systemd/system/kubelet.service

# Running processes can be seen by listing process on worker node:
ps -aux | grep kubelet
```

## Kube Proxy

In Kubernetes, a pod on one node can reach any pod on another node using the POD network. For example, if a Webserver pod needs to access a Database Pod, it can be done by using the IP address. However, since pods are ephemeral and their IPs are not fixed, we can create a service. The service allows the webserver app to access the database using either the service name or the IP address of the service. The service then forwards the traffic to the backend pod. 

Now, the service is a virtual component residing in the memory of Kubernetes and does not have any interfaces or listening processes like pods. So, how does the service forward traffic to the correct pod without being in the pod network? The answer lies in Kube Proxy.

Kube-proxy is a network proxy and load balancer in Kubernetes. It runs on each node in the cluster and is responsible for routing network traffic to the appropriate pods and services. It enables communication between pods and allows external access to services by managing network rules (for ex. IPTABLE rules) and performing network address translation (NAT). Kube-proxy ensures service discovery and load balancing within the cluster, facilitating seamless connectivity between applications.

**************************************************************************************Deploying Kube-Proxy in K8s cluster (built from scratch) :************************************************************************************** Download binaries, install and run it as a service

**************************************************************************************Deploying Kube-Proxy in K8s cluster (built from Kubeadm) :************************************************************************************** Installed as a pod, infact as a Daemonset, so that a single POD is always deployed on each node in the cluster.

### What is a Proxy and Load Balancer?

A proxy is an intermediary between clients and servers, acting as a gateway for network traffic. It receives requests from clients and forwards them to the appropriate servers, often providing additional functionality such as caching, authentication, or security.

A load balancer, on the other hand, distributes incoming network traffic across multiple servers or backend resources. It helps evenly distribute the workload, improving performance, scalability, and availability by preventing any single server from becoming overwhelmed.

In summary, a proxy is a network intermediary that handles client-server communication, while a load balancer is a mechanism that evenly distributes network traffic across multiple servers or resources. A load balancer can make use of a proxy to perform additional functions, such as SSL termination or content caching.

## PODs

In the software development lifecycle, after the application is developed and built into Docker images and made available on a Docker repository, Kubernetes can pull it down. The application can then be run as a container, but in an encapsulated form known as a pod.

A pod is the smallest and simplest unit in Kubernetes. It represents a single instance of a running process in a cluster. A pod encapsulates one or more containers, along with shared storage and network resources. Pods are used to deploy and manage applications in Kubernetes, allowing for easy scaling, monitoring, and orchestration of containerized workloads.

### How is Pod created?

[How does Kubernetes create a Pod?](https://youtu.be/BgrQ16r84pM)

Authenticate User → Validate Request → Retrieve Data → Update ETCD → Scheduler → Kubelet

**Note:** 
The kube-apiserver is the only component that interacts directly with the etcd datastore.
The other components such as the scheduler, kube-controller-manager and kubelet uses kube-apiserver to perform updates in the cluster in their respective areas.

### POD Scaling

If the user base accessing the application increases, the question arises: how do we scale the application? The answer is to spin up additional new instances to share the load. 

However, where should we spin up those instances? Instead of creating a new container instance within the same pod, we should spin up new pods with new instances of the same application. This approach allows us to maintain a one-to-one relationship between pods and containers. 

What if the user load further increases and the current node has no sufficient capacity? Well, then we can deploy pods on a new node in the cluster to increase the overall physical capacity.

### Multi-Container PODs

Multi-container pods are a concept in Kubernetes where multiple containers are co-located and share the same resources within a single pod. Helper containers, also known as sidecar containers, are additional containers within a pod that provide support to the main application container.

In a multi-container pod, containers share the same storage and network namespaces, allowing them to communicate and access shared resources easily. They are created and destroyed together as part of the pod lifecycle, ensuring consistency and co-location of the containers within the pod.

### What is Kubectl?

Kubectl is the command-line tool used to interact with Kubernetes clusters. It allows users to manage and control various aspects of Kubernetes, such as deploying and managing applications, inspecting cluster resources, scaling deployments, and troubleshooting. Kubectl provides a convenient interface to interact with the Kubernetes API and perform operations on the cluster from the command line.

### Basic Commands

```bash
# To create a Pod (Imperative command)
kubectl run pod_name --image=image_name

# To create a Pod with label (Imperative command)
kubectl run pod_name --image=image_name -l key=value
# We can also use --label or --selector in place of -l.
# They all serve the same purpose.

# To create a Pod and exposing it on a container port
kubectl run pod_name --image=image_name --port=port_number

# To get list of Pods
kubectl get pods or po # Lists down pods in default namespace

# In output, what does the READY column indicate?
# Answer : Running Containers in Pod / Total Containers in Pod

# To get list of Pods with additional info like Nodename, pod IP address
kubectl get pods -o wide

# To get lists of pods with multiple labels
kubectl get pods -l key1=value1,key2=value2,key3=value3

# To get the list of nodes
kubectl get nodes or no
kubectl get nodes -o wide

# To get detailed info about a Pod
kubectl describe pod pod_name

# To delete a Pod
kubectl delete pod pod_name

# To delete multiple pods using their names
kubectl delete pod pod1 pod2 pod3

# To delete pods with label selector and namespace
kubectl delete pods -l key=value -n namespace_name
```

## YAML in Kubernetes

YAML (YAML Ain't Markup Language) is a human-readable data serialization format commonly used for configuration files in software applications. YAML is used in Kubernetes to define and configure resources, allowing users to manage their applications and infrastructure declaratively. It provides a readable and structured way to create, update, and delete Kubernetes resources.

### Basic YAML syntax for defining a Pod

```yaml
# Creating a pod-definition.yaml or pod-definition.yml file

apiVersion: v1

kind: Pod 

metadata:
  name: myapp-pod
  labels:
    app: myapp
    type: front-end

spec:
  containers:
    - name: container-name
      image: image-name
```

In this YAML snippet:

- **`apiVersion`** specifies the Kubernetes API version being used.
- **`kind`** defines the type of resource, which in this case is a Pod.
- **`metadata`** contains information about the Pod, including its name. Under `labels` we can have the freedom to create custom key-value pairs according to our preferences.
- **`spec`** describes the desired state of the Pod, including the containers it should run. The spec section can be different for different objects.
- Under **`containers`**, you can define one or more containers within the Pod, specifying the container name and the Docker image to use. We can have different names and images for the containers.

```bash
# Creating an object using the YAML file
kubectl create -f pod-definition.yml # creates the pod
# or
kubectl apply -f pod-definition.yml # creates the pod

# We can create multiple objects from a directory of config files
kubectl apply -f /path/to/config-files
```

****************************************Difference between using ‘create -f’ and ‘apply -f’ :****************************************

**`kubectl create -f`** for initial resource creation based on the configuration specified in YAML or JSON file. If the resource already exists, it will throw an error.

**`kubectl apply -f`** for creating or updating resources based on the configuration specified in YAML or JSON file. If the resource already exists, it will be updated with the new configuration. If the resource does not exist, it will be created.

### ‘Kind’ and ‘apiVersion’ values for different K8s objects

| Object | Kind | API Version |
| --- | --- | --- |
| Pod | Pod | v1 |
| Deployment | Deployment | apps/v1 |
| ReplicaSet | ReplicaSet | apps/v1 |
| Service | Service | v1 |
| Namespace | Namespace | v1 |
| ConfigMap | ConfigMap | v1 |
| Secret | Secret | v1 |
| PersistentVolume | PersistentVolume | v1 |
| StatefulSet | StatefulSet | apps/v1 |
| Job | Job | batch/v1 |
| CronJob | CronJob | batch/v1beta1 |
| Ingress | Ingress | networking.k8s.io/v1beta1 |

Please note that the **`kind`** and **`apiVersion`** values may vary depending on the Kubernetes version and specific configuration. It's always a good practice to refer to the official Kubernetes documentation for the most up-to-date information.

## ReplicaSet

A ReplicaSet in Kubernetes is a controller that ensures a specified number of identical pods are running at all times, providing high availability and scalability for applications.

ReplicaSet is an enhanced and more capable replacement for the ReplicationController, providing improved label matching, rolling updates, and better support for managing pod replicas in Kubernetes.

A ReplicaSet in Kubernetes manages pods based on the labels associated with them. It uses label selectors to identify and control the pods it manages. When a ReplicaSet is created, it selects and manages pods that match its specified labels. If there are existing pods with matching labels, the ReplicaSet takes control of them. If new pods are created that match the ReplicaSet's labels, it will also manage them. This allows the ReplicaSet to ensure the desired number of replicas is maintained, regardless of whether the pods were created by the ReplicaSet or not.

### Basic YAML syntax for ReplicaSet

```yaml
apiVersion: apps/v1

kind: ReplicaSet

metadata:
  name: my-replicaset # name of replicaset

spec:
  template: #contains the specification for the pods managed by the ReplicaSet.
    metadata: 
      labels:
        app: my-app
    spec:
      containers:
      - name: my-container
        image: my-image:latest

	replicas: 3 #specifies the desired number of replicas.

  selector: #defines the label selector used to match the pods
    matchLabels:
      app: my-app

# Note that 'template' , 'replicas' and 'selector' are direct child of 'spec'
```

**Note:** 

1. The 'template' section is included in the Replicaset configuration to ensure that if any pod gets deleted in the future, the Replicaset can create a new pod based on the defined template, to maintain the specified number of replicas.
2. The **`matchLabels`** field in the **`selector`** is used to select the pods that should be part of the ReplicaSet based on their labels. The **`labels`** field in the **`metadata`** of the **`template`** specifies the labels to be applied to the pods managed by the ReplicaSet.
By having the same key-value pairs in both places, the ReplicaSet ensures that it selects and manages the pods with matching labels.

### Commands

```bash
kubectl create -f rs-definition.yml # creates the replicaset

kubectl get replicaset or rs # to see list of replicasets (in default namespace)

kubectl delete replicaset replicaset_name # deletes the replicaset 
# and all underlying pods

# Methods of Scaling the Replicas:

# 1. Edit the replicas in the yaml file and run the command:
		 kubectl apply -f 
		 #or
		 kubectl replace -f rs-definition.yml #to replace existing configs
		 kubectl replace --force -f rs-definition.yml #forcefully replace

# 2. Using edit command
		 kubectl edit replicaset replicaset_name

# 3. Using scale command
		 kubectl scale --replicas=6 -f rs-definition.yml # using config file name
		 #or
		 kubectl scale --replicas=6 replicaset replicaset_name # using Type and Name

# Note that by using the edit or scale command, the config file is not updated.
```

****************************************Difference between using ‘apply -f’ and ‘replace -f’ :****************************************

**`kubectl apply -f`** is generally preferred for managing resources in declarative way because it is more flexible and preserves existing configurations. 

**`kubectl replace -f`** is useful when you want to ensure a complete replacement of the resource's configuration.

## Deployment

A Deployment in Kubernetes is a higher-level abstraction that manages and orchestrates the creation and scaling of ReplicaSets. It provides a declarative way to define and manage the desired state of a set of Pods.

Deployments provide a higher-level, more flexible, and convenient way to manage application deployments, updates, and rollbacks compared to ReplicaSets.

The YAML syntax of creating Deployment configuration file is same as the ReplicaSet. Only the `kind` field is ‘Deployment’ instead of ‘ReplicaSet’.

### Commands

```bash
# To create a deployment using config file
kubectl create -f deploy-def.yaml
# or
kubectl apply -f deploy-def.yaml

# To create a deployment using create command
kubectl create deployment --image=image_name deployment_name

# To see list of deployment in current namespace
kubectl get deployment or deploy

# To get info about all K8s objects or resources in current namespace
kubectl get all
```

## Exam **Tips**

As we have seen already, it is a bit difficult to create and edit YAML files, especially in the CLI. During the exam, we might find it difficult to copy and paste YAML files from browser to terminal. 

We can use the below commands to get better:

Reference (Bookmark this page for exam. It will be very handy):

https://kubernetes.io/docs/reference/kubectl/conventions/

```bash
# Create an NGINX Pod
kubectl run nginx --image=nginx

# Generate POD Manifest YAML file (-o yaml). Don’t create it(–dry-run)
kubectl run nginx --image=nginx --dry-run=client -o yaml

# Create a deployment
kubectl create deployment --image=nginx nginx

# Generate Deployment YAML file (-o yaml). Don’t create it(–dry-run)
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml

# Generate Deployment YAML file (-o yaml). Don’t create it(–dry-run) 
# and save it to a file.
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml

# In k8s version 1.19+, we can specify the –replicas option to 
# create a deployment with 4 replicas.
kubectl create deployment --image=nginx nginx --replicas=4 --dry-run=client -o yaml > nginx-deployment.yaml

# Make necessary changes to the file (for example, adding more replicas) 
# and then create the deployment.
kubectl create -f nginx-deployment.yaml
# or
kubectl apply -f nginx-deployment.yaml

# REMEMBER: 
# > is used for overwriting the content of the file and 
# >> is used for appending to the file
```

## Services in Kubernetes

In Kubernetes, a Service is an abstraction representing a logical group of Pods and a policy for accessing them. It provides a stable network endpoint for communication with the Pods. Services enable decoupling between frontend and backend components, offering load balancing, scaling, and service discovery within a cluster.

They act as intermediaries, abstracting individual Pod IP addresses and providing a consistent endpoint. Different types of Services, such as ClusterIP, NodePort, LoadBalancer, and ExternalName, determine how they are exposed and accessed.

Using Services ensures reliable communication, load balancing, and simplified service discovery in Kubernetes, promoting seamless interaction between application components.

### What are Endpoints?

In Kubernetes, endpoints are a list of network addresses (IPs and ports) that correspond to a set of Pods targeted by a Service. Endpoints define where the traffic should be routed to when accessing the Service. They essentially represent the backend Pods that the Service is responsible for load balancing and directing traffic to.

## NodePort Service

Suppose there is a web server app running in a pod on a node, we can access it from the same node but what if an external user wants to access the application. We can use NodePort service for it.

A NodePort service in Kubernetes exposes an application running inside the cluster to external clients. It maps a specific port on all cluster nodes to the target port of the service, allowing external access by connecting to any cluster node's IP address and the assigned NodePort. NodePort services are commonly used for external access during development or testing.

![Service-Port-NodePort-TargetPort.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b6b2abe3-db8c-4930-89cd-6c474c8fc997/Service-Port-NodePort-TargetPort.png)

### NodePort YAML Syntax

```yaml
# Creating service-definition.yml file

apiVersion: v1
kind: Service
metadata: # metadata specifies the name of the service
  name: my-nodeport-service
spec:
  type: NodePort
  ports:
    - port: 8080 # port number exposed internally in cluster (mandatory field)
      targetPort: 80 # port that containers/pods are listening on
      nodePort: 30000 # access service via this external port number
			# Default range for nodePort is 30000 to 32767
  selector:
    app: my-app
	# the selector field specifies the pods to which the service should 
	# route traffic based on their labels
```

### Commands

```bash
kubectl create -f service-definition.yml # creates the service

kubectl get services or svc # lists down the services

kubectl describe service service_name # gives info about the service

# Now we can curl to node's ip address to access the application
curl http://worker_node_ip:nodePort
```

### NodePort Service Scenarios

1. Single pod on a single node: In this scenario, the NodePort service provides a static port on every node in the cluster. External traffic can reach the pod through any node's IP address and the assigned NodePort. This setup allows for direct access to the single pod.
2. Multiple pods on a single node: When multiple pods are running on a single node, the NodePort service will distribute incoming traffic to any of the available pods on that node. It automatically adapts to changes, such as adding or removing pods, ensuring load balancing and flexibility.
3. Multiple pods on multiple nodes: In this case, the NodePort service distributes incoming traffic across all nodes in the cluster. Each node then routes the traffic to the appropriate pod based on the service's internal routing rules. As pods are added or removed, the service updates its routing accordingly, providing seamless communication across multiple nodes and pods.

In summary, NodePort services in Kubernetes facilitate external access to pods, regardless of the pod and node distribution. They offer load balancing, adaptability, and consistent entry points for accessing the pods.

## ClusterIP Service

A ClusterIP service in Kubernetes provides a stable internal IP address for accessing pods within the cluster. It is the **default service type** and is used for internal communication among application components in the cluster.

The ClusterIP service assigns a virtual IP address to enable communication between pods. It is not accessible from outside the cluster and is designed for internal use only.

ClusterIP services are commonly used in micro-services architectures to facilitate communication between different components within the cluster.

![1_dLlC4L2qpImyZS6gOntUjg.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e9cbe0a1-0d95-4bc6-bb35-3c6e0323203d/1_dLlC4L2qpImyZS6gOntUjg.png)

### ClusterIP YAML syntax

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: ClusterIP
  selector: # specifies the labels of the pods that the service should target
    app: my-app
  ports: # defines the port configuration for the service
    - protocol: TCP

      port: 80 # port number on which the service will listen for incoming traffic.
               # It represents the port that clients can use to access the service.

      targetPort: 8080 # specifies the port number on the Pods to which the traffic 
											 # should be forwarded. It represents the port on which the 
											 # application or service inside the Pod is listening to.

# While the port field is typically required, the targetPort field is optional.
# If targetPort is not specified, Kubernetes will use the same value 
# as the port field by default. This means that the service will forward 
# traffic to the same port number on the Pods.
```

## LoadBalancer Service

A LoadBalancer service in Kubernetes is a type of service that automatically provisions an external load balancer to distribute incoming traffic across the pods in a cluster. It provides a stable external IP address or DNS name that can be used to access the service from outside the cluster. This allows for scaling and distributing traffic to the application running inside the pods, ensuring high availability and reliability.

![Screenshot 2023-06-17 at 5.08.14 PM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4463cfb5-66b0-4a43-ac84-0f9735ea0e99/Screenshot_2023-06-17_at_5.08.14_PM.png)

### Loadbalancer YAML syntax

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: LoadBalancer
  ports: # specifies the protocol, port, and targetPort for the service
    - protocol: TCP
      port: 80
      targetPort: 8080
  selector: # used to match the service to the pods with the corresponding labels
    app: my-app
```

**NOTE:**

On cloud providers which support external load balancers, setting the `type` field to `LoadBalancer` provisions a load balancer for your Service.

When a **`LoadBalancer`** Service is created in a cluster that doesn't support external load balancers, it may automatically fall back to behaving like a **`NodePort`** Service. This means that the Service will be exposed on a specific port on each cluster node, allowing external traffic to reach the Service by accessing any node's IP address along with the assigned NodePort.

However, it's important to note that the fallback behavior may vary depending on the Kubernetes distribution and configuration.

## Namespaces in Kubernetes

In Kubernetes, namespaces are a way to organize and isolate resources within a cluster. They provide a virtual partitioning of a Kubernetes cluster, allowing multiple teams or applications to coexist independently. Namespaces provide a scope for resources such as Pods, Services, Deployments, and ConfigMaps, preventing naming conflicts and providing better resource management and access control.

In a Kubernetes cluster, several namespaces are automatically created during the setup, depending on the Kubernetes distribution or setup. These include:

1. **`default`**: The default namespace where objects are created if no namespace is specified.
2. **`kube-system`**: This namespace contains the essential Kubernetes system components and services.
3. **`kube-public`**: This namespace is readable by all users and is generally used for public information or resources.
4. **`kube-node-lease`**: This namespace contains lease objects associated with each node in the cluster.

### DNS in Namespace

In the same namespace, DNS resolution works by directly referencing the service name. Each service within the namespace gets a DNS record with its name, allowing other pods within the same namespace to resolve it.

In different namespaces, DNS resolution works by including the service name along with the namespace name. Pods in different namespaces can access services in other namespaces by using the fully qualified domain name (FQDN) in the format: 
**`<service-name>.<namespace-name>.svc.cluster.local`**. 
This FQDN allows cross-namespace communication and resolves to the correct service IP address.

### Namespace YAML syntax

```yaml
# Creating ns-def.yml

apiVersion: v1
kind: Namespace
metadata:
  name: my-namespace
```

### Commands

```bash
# To see list of namespaces
kubectl get namespaces

# To get the count of namespaces
kubectl get namespaces --no-headers | wc -l 

# To see pods of a namespace
kubectl get pods --namespace=namespace_name or -n namespace_name

# To see pods of all namespaces
kubectl get pods --all-namespaces or -A

# To create pod in a namespace
kubectl run pod_name --image=image_name -n namespace_name

# To create a pod in a namespace using pod config file
kubectl create -f pod-definition.yml --namespace=namespace_name
# or simply add metadata.namespace=dev in pod config file
kubectl create -f pod-definition.yml

# To create namespace using config file
kubectl create -f ns-def.yml

# To create namespace using 'create' command
kubectl create namespace namespace_name

# To switch between namespaces
kubectl config set-context --current --namespace=<namespace-name>
# where <namespace-name> with the name of the namespace you want to switch to.

# Verify the active namespace by
kubectl config view --minify --output 'jsonpath={..namespace}'

# Note: 
# The output of kubectl config view displays the Kubernetes configuration 
# details in YAML format, including the current context, clusters, users, 
# and namespaces.
```

### Resource Limits

‘Resource Limits’ control the resource allocation per Pod or Container within a namespace. These limits ensure that each Pod or Container does not exceed the specified resource allocation.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: my-image
    resources:
      limits:
        cpu: "1"
        memory: "512Mi"
```

The resource limits are defined within the **`resources`** field under the **`containers`** section for a specific Pod. The **`limits`** field specifies the maximum amount of CPU and memory resources that can be allocated.

### Resource Quotas

‘Resource Quotas’ control the resource allocation at the namespace level. They define the maximum amount of CPU, memory, and storage resources that can be used collectively by all Pods and Containers within the namespace.

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: my-quota
spec:
  hard:
    cpu: "4"
    memory: "4Gi"
    pods: "10"
```

Here, a **`ResourceQuota`** object is created, and the **`hard`** field specifies the maximum resource limits for CPU, memory, and number of Pods allowed within the namespace.

## Imperative vs Declarative

In Kubernetes, imperative and declarative commands refer to two different approaches for managing resources and making changes to the cluster.

1. Imperative Commands: Imperative commands are action-based commands that directly instruct Kubernetes on what actions to perform. With imperative commands, you specify the exact steps and changes to be made. Examples of imperative commands include **`kubectl create`**, **`kubectl delete`**, and **`kubectl scale`**.
    
    For example, to create a deployment imperatively, you would use a command like **`kubectl create deployment my-deployment --image=my-image`**.
    
    Imperative commands are often used for quick, one-time operations or for troubleshooting purposes. They can be useful when you want precise control over the actions being taken.
    
2. Declarative Commands: Declarative commands, on the other hand, focus on describing the desired state of the system rather than the specific actions to be taken. Instead of instructing Kubernetes on how to perform each step, you provide a YAML or JSON file that defines the desired state of the resources.
    
    For example, to create a deployment declaratively, you would define the desired state in a YAML file and use the **`kubectl apply`** command to apply that configuration.
    
    Declarative commands allow you to define the desired state of your resources and let Kubernetes handle the details of achieving and maintaining that state. They are often used for managing resources in a more automated and consistent manner.
    

Both imperative and declarative commands have their own benefits and use cases. Imperative commands offer more explicit control and immediate results, while declarative commands promote consistency and automation. The choice between them depends on your specific needs and preferences.

## Exam Tips - 2

While you would be working mostly the declarative way – using definition files, imperative commands can help in getting one time tasks done quickly, as well as generate a definition template easily. This would help save considerable amount of time during your exams.

Before we begin, familiarize with the two options that can come in handy while working with the below commands:

- `--dry-run`: By default as soon as the command is run, the resource will be created. If you simply want to test your command , use the `--dry-run=client` option. This will not create the resource, instead, tell you whether the resource can be created and if your command is right.
- `-o yaml`: This will output the resource definition in YAML format on screen.

Use the above two in combination to generate a resource definition file quickly, that you can then modify and create resources as required, instead of creating the files from scratch.

### **Service**

****************************************************Create a Pod named ‘httpd’ and a service ‘httpd’ of type ClusterIP and expose it on 80****************************************************

`kubectl run httpd --image=httpd:alpine --port=80 --expose`

**Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379**

`kubectl expose pod redis --port=6379 --name=redis-service --dry-run=client -o yaml`

(This will automatically use the pod’s labels as selectors)

Or

`kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml` 
(This will not use the pods labels as selectors, instead it will assume selectors as **app=redis.** 
[You cannot pass in selectors as an option.](https://github.com/kubernetes/kubernetes/issues/46191) So it does not work very well if your pod has a different label set. So generate the file and modify the selectors before creating the service)

**Create a Service named nginx of type NodePort to expose pod nginx’s port 80 on port 30080 on the nodes:**

`kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml`

(This will automatically use the pod’s labels as selectors, [but you cannot specify the node port](https://github.com/kubernetes/kubernetes/issues/25478). You have to generate a definition file and then add the node port in manually before creating the service with the pod.)

Or

`kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml`

(This will not use the pods labels as selectors)

Both the above commands have their own challenges. While one of it cannot accept a selector the other cannot accept a node port. I would recommend going with the `kubectl expose` command. If you need to specify a node port, generate a definition file using the same command and manually input the nodeport before creating the service.

## How Kubectl Apply (declarative) command works?

When using **`kubectl apply`**, the local configuration file is the YAML or JSON file that contains the desired configuration for the resources. This file specifies how the resources should be created or updated in the cluster.

The **`kubectl apply`** command leverages the concept of the last applied configuration. It compares the desired configuration in the local file with the existing configuration of the resources in the cluster. It updates only the fields that have changed in the configuration, preserving any existing settings that are not explicitly modified in the local file.

The configuration of resources in the cluster is stored in the etcd database, which serves as the persistent storage for Kubernetes. When applying configuration changes with **`kubectl apply`**, the changes are applied to the cluster's etcd database, and the cluster's state is updated accordingly. There is no separate cache memory configuration involved.

## How ‘kubectl edit’ (imperative) command works?

When using the **`kubectl edit`** command, the changes are made directly to the live object in the cluster, and the local configuration file is not automatically updated. If someone updates the local config file after using **`kubectl edit`**, any changes made through the imperative command will not be reflected in the local config file. The changes made through **`kubectl edit`** are stored in the cache of the Kubernetes client and are not persistently saved in a local file.