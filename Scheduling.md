# SCHEDULING

Scheduling in Kubernetes refers to the process of assigning pods to nodes in the cluster. It is responsible for determining which node should run each pod based on resource availability, constraints, and other factors.

## Manual Scheduling (NodeName)

Manual scheduling in Kubernetes refers to the process of explicitly specifying the node on which a pod should be scheduled. By assigning a specific node, you can control the placement of the pod and override the default scheduling behavior of Kubernetes.

**Scenario 1**

When creating a new pod, manual scheduling in Kubernetes allows you to specify the specific node where the pod should be deployed. This is done by setting the `nodeName` field under `spec` section in the pod's configuration.

**Scenario 2**

In the case of an already created pod, manual scheduling involves moving the pod from its current node to a different node. This can be done by setting the `nodeName` field to the desired node either by using the `kubectl edit` command or updating the pod's configuration. 
However, it's important to note that manually moving a pod can disrupt its availability and may require additional steps to ensure data integrity and application stability.

An alternative method can be achieved by using pod binding. This involves creating a binding object that specifies the pod's name and the desired node. The binding is then sent to the Kubernetes API server to assign the pod to the specified node. However, it's important to note that manually binding pods should be done with caution, considering the potential impact on availability and application stability.

YAML syntax for Pod Binding:

```yaml
apiVersion: v1
kind: Binding
metadata:
  name: <binding-name> # name for the binding object
target:
  apiVersion: v1
  kind: Node
  name: <node-name> # name of the desired node to which the pod should be assigned

# Run the kubectl apply -f command to perform the pod binding.
```

## Labels and Selectors

Labels and selectors in Kubernetes are used for identifying and grouping resources within the cluster. Labels are key-value pairs attached to Kubernetes objects, adding metadata and attributes. 

Selectors allow filtering and selecting resources based on their labels. They enable effective organization, grouping, and operations on specific subsets of resources.

## Taints and Tolerations

Taints and tolerations in Kubernetes are used to control the scheduling of pods on nodes.

Taints are applied to nodes and indicate a preference for or restriction against running certain pods on that node. Taints are set at the node level and can be used to mark nodes as "unavailable" for certain types of workloads.

Tolerations, on the other hand, are set on pods and allow them to tolerate or accept nodes with specific taints. Tolerations override the taints and allow pods to be scheduled on nodes that would otherwise reject them.

By using taints and tolerations, you can enforce specific placement rules for pods and ensure that they are scheduled on appropriate nodes based on the desired constraints and requirements of your workload.

### Taint Command

```bash
kubectl taint nodes <node-name> <key>=<value>:<effect>

# Example:
kubectl taint nodes node-1 app=frontend:NoSchedule

# To see the taint applied on a node
kubectl describe node <node-name> | grep -i taint

# To delete the taint applied on a node, add hyphen at the end.
kubectl taint nodes <node-name> <key>=<value>:<effect>-
```

- **`<node-name>`**: The name of the node to which the taint should be applied.
- **`<key>`**: The key of the taint, representing a specific attribute or characteristic.
- **`<value>`**: The value associated with the taint key, providing additional details about the taint.
- **`<effect>`**: It determines how pods are scheduled on the tainted node.
The effect of the taint, which can be one of three options:
    1. **`NoSchedule`**: The **`NoSchedule`** effect means that no new pods will be scheduled onto the tainted node by default. Existing pods on the node are unaffected.
    2. **`PreferNoSchedule`**: The **`PreferNoSchedule`** effect allows new pods to be scheduled onto the tainted node, but Kubernetes will try to avoid doing so if possible. Existing pods on the node are unaffected.
    3. **`NoExecute`**: The **`NoExecute`** effect is similar to **`NoSchedule`**, but it also evicts any existing pods on the node that do not tolerate the taint. This effect is typically used for maintenance or critical scenarios where pods need to be forcefully evicted from the node. The evicted pods are terminated ie. they are not rescheduled on any node.

### Toleration YAML syntax

```yaml
# Add this under the 'spec' section of the pod's config file
tolerations:
  - key: <key>
    operator: <operator>
    value: <value>
    effect: <effect>

# Example
tolerations:
    - key: "app"
      operator: "Equal"
      value: "frontend"
      effect: "NoSchedule"
```

- **`key`**: The key is the name of the taint that the toleration is targeting.
- **`operator`**: The operator specifies how to compare the toleration value with the taint value. Possible values are **`Exists`** (toleration matches any taint with the specified key), and
**`Equal`** (toleration matches a taint with the exact key-value).
- **`value`**: The value is the value associated with the taint key. This field is optional and can be used to match tolerations with specific values.
- **`effect`**: The effect field specifies the effect of the toleration, which can be one of **`NoSchedule`**, **`PreferNoSchedule`**, or **`NoExecute`**. This determines how the toleration affects the scheduling of pods.

You can add multiple tolerations to a pod's YAML definition by including them in a list under the **`tolerations`** field.

### **NOTE:**

1. By default, the master node in a Kubernetes cluster does have a taint applied to it. This taint is typically used to ensure that only specific pods or workloads that tolerate the master node's taint are scheduled on it. This is done to segregate critical system components and control the workload distribution across the cluster.
2. Taints and tolerations do not instruct the pods to which node they should go; instead, they instruct the nodes to accept pods with specific tolerations.
    
    Tolerations allow pods to be scheduled on nodes with specific taints, but they don't guarantee that a pod will be scheduled on the exact node with the corresponding taint. It's possible for a pod to be scheduled on a different node, as long as that node tolerates the specified taint.
    
    To decide the pod to be placed upon a particular node, we can use the Affinity rules.
    

## Node Selector

Node Selector in Kubernetes is a feature that allows you to specify criteria for selecting nodes where a particular pod should be scheduled. It is a way to control the placement of pods on specific nodes based on node labels.

To use Node Selector, you need to assign labels to your nodes using key-value pairs.

```bash
# To Label a node
kubectl label nodes <node-name> <label-key>=<label-value>

# To see the labels assigned to the node
kubectl describe node <node-name>
```

Now in the pod's YAML configuration, you can specify the **`nodeSelector`** field under the **`spec`** section to define the labels of the nodes where the pod should be scheduled. Only nodes that have matching labels will be considered for scheduling the pod. Nodes not having the matching labels will be ignored for scheduling the pod.

```yaml
nodeSelector:
	label-key: label-value
```

Node Selector is useful when you have specific node requirements for your pods, such as running them on dedicated nodes or nodes with specific hardware capabilities.

### How ‘Node Selectors’ are different from ‘nodeName’ (Manual Scheduling) ?

Node Selectors and nodeName are two different ways to control the placement of pods on specific nodes in Kubernetes.

- Node Selector provides more flexibility and allows you to define complex criteria using labels to select nodes for scheduling. It provides a way to dynamically assign pods to nodes based on matching labels.
- nodeName is a static assignment and directly specifies the node where the pod should be scheduled. It does not consider labels or any other criteria for node selection. This overrides the default scheduler behaviour and forces the pod to be scheduled on the specified node.

## Node Affinity

Node affinity and anti-affinity are features in Kubernetes that allow you to influence the scheduling and placement of pods on specific nodes or against certain nodes, based on node labels and attributes. They enable you to optimize resource allocation, improve high availability, and achieve better performance and resilience in your Kubernetes clusters.

`nodeSelector` only selects nodes with all the specified labels. Affinity/anti-affinity gives you more control over the selection logic. Node affinity is conceptually similar to `nodeSelector`, allowing you to constrain which nodes your Pod can be scheduled on based on node labels. There are two types of node affinity:

- `requiredDuringSchedulingIgnoredDuringExecution`: The scheduler can't schedule the Pod unless the rule is met. This functions like `nodeSelector`, but with a more expressive syntax.
- `preferredDuringSchedulingIgnoredDuringExecution`: The scheduler tries to find a node that meets the rule. If a matching node is not available, the scheduler still schedules the Pod.

`IgnoredDuringExecution` means that if the node labels change after Kubernetes schedules the Pod, the Pod continues to run.

### YAML Syntax:

```yaml
# Added under 'spec' section of Pod's config file
affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: key1
            operator: In
            values:
            - value1
          - key: key2
            operator: Exists
						# Values are not needed in 'Exists' or 'DoesNotExist' operator

      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        preference:
          matchExpressions:
          - key: key3
            operator: Exists

# VI Editor TIP:
When moving the lines a little right (making them the child), 
press capital V and select the lines to be moved,
then press > (Shift and .)
```

The **`requiredDuringSchedulingIgnoredDuringExecution`** section defines the required affinity rules, while the **`preferredDuringSchedulingIgnoredDuringExecution`** section defines the preferred affinity rules. 

### Operators

In node affinity rules in Kubernetes, you can use the following operators to define the relationship between the node affinity rule and the nodes:

1. **`In`**: The node's label value must be one of the specified values in the affinity rule.
2. **`NotIn`**: The node's label value must not be any of the specified values in the affinity rule.
3. **`Exists`**: The node must have the specified label key present, regardless of its value.
4. **`DoesNotExist`**: The node must not have the specified label key present.
5. **`Gt`**: The node's label value must be greater than the specified value in the affinity rule.
6. **`Lt`**: The node's label value must be less than the specified value in the affinity rule.

Read [Operators](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#operators) to learn more about how these work.

### **Note:**

1. The `weight` parameter is used to specify the relative importance or preference of a node affinity rule. It ranges from 1 to 100.
It is applicable when using the `preferredDuringSchedulingIgnoredDuringExecution` section of node affinity.
2. You can use the `operator` field to specify a logical operator for Kubernetes to use when interpreting the rules. You can use `In`, `NotIn`, `Exists`, `DoesNotExist`, `Gt` and `Lt`.
`NotIn` and `DoesNotExist` allow you to define node anti-affinity behavior. Alternatively, you can use [node taints](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/) to repel Pods from specific nodes.
3. If you specify both `nodeSelector` and `nodeAffinity`, *both* must be satisfied for the Pod to be scheduled onto a node.
4. If you specify multiple terms in `nodeSelectorTerms`, then the Pod can be scheduled onto a node if one of the specified terms can be satisfied (terms are ORed).
5. If you specify multiple expressions in a single `matchExpressions` field in `nodeSelectorTerms`, then the Pod can be scheduled onto a node only if all the expressions are satisfied (expressions are ANDed).

### **Inter-pod affinity and anti-affinity**

Inter-pod affinity and anti-affinity allow you to constrain which nodes your Pods can be scheduled on based on the labels of **Pods** already running on that node, instead of the node labels.

Inter-pod affinity and anti-affinity rules take the form "this Pod should (or, in the case of anti-affinity, should not) run in an X if that X is already running one or more Pods that meet rule Y", where X is a topology domain like node, rack, cloud provider zone or region, or similar and Y is the rule Kubernetes tries to satisfy.

**Note:** Inter-pod affinity and anti-affinity require substantial amount of processing which can slow down scheduling in large clusters significantly. We do not recommend using them in clusters larger than several hundred nodes.

## Taints and Tolerations vs Node Affinity

Taints and tolerations allow nodes to repel or tolerate pods based on the taints applied to the nodes and the tolerations specified by the pods. Taints are applied to nodes to indicate that they should only accept pods with matching tolerations. This mechanism is useful when you want to restrict pods from running on certain nodes or reserve specific nodes for special workloads.

Node affinity, on the other hand, is a mechanism that allows you to define rules for pod placement based on node labels. It specifies the desired characteristics of the nodes where pods should be scheduled. Node affinity rules ensure that pods are scheduled on nodes that meet specific label criteria, such as requiring pods to be placed on nodes with certain labels or avoiding nodes with specific labels.

While taints and tolerations provide a way to control node behavior and pod placement based on node attributes, node affinity provides more fine-grained control over pod scheduling by leveraging node labels.

However, by combining taints and tolerations with node affinity, you can create more sophisticated scheduling policies. For example, you can use taints and tolerations to restrict certain pods from running on nodes with specific attributes, while using node affinity to ensure that pods are scheduled on nodes that have the desired characteristics.

The combination of taints and tolerations with node affinity provides greater flexibility and control over pod scheduling, allowing you to fine-tune the placement of pods based on both node attributes and labels.

## Resource Requests and Resource Limits

Resource requests refer to the minimum amount of compute resources (CPU and memory) that a pod requires to run. These requests serve as a way for the Kubernetes scheduler to determine the appropriate node for scheduling the pod. The scheduler ensures that a node has enough available resources to meet the requested amount before scheduling the pod.

On the other hand, resource limits define the maximum amount of compute resources that a pod can consume. These limits are set to prevent pods from using excessive resources and affecting the stability and performance of the cluster. If a pod exceeds its resource limit, Kubernetes takes actions such as throttling (when it exceeds CPU limit) or terminating (when it exceeds memory limit with OOM error) the pod to prevent resource starvation and maintain cluster stability.

Resource requests and limits can be specified in the pod's YAML configuration using the **`resources`** field under the **`spec`** section. The resources can be specified in terms of CPU units (cores) and memory (bytes or other units like Mi, Gi).

**YAML syntax for Resource Requests and Limits:**

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
      requests:
        cpu: "0.5"
        memory: "512Mi"
      limits:
        cpu: "1"
        memory: "1Gi"

# In this case, the container has a request of 0.5 CPU units and 512 megabytes 
# of memory, and a limit of 1 CPU unit and 1 gigabyte of memory.
```

### Limit Ranges

LimitRanges in Kubernetes are used to enforce resource constraints on pods within a namespace. They define the minimum and maximum resource limits for CPU, memory, and other resources that can be requested by pods. LimitRanges help ensure fair resource allocation and prevent individual pods from using excessive resources, which can affect the stability and performance of the cluster. By defining limits at the namespace level, administrators can control and manage resource utilization across different applications and teams within the cluster.

**YAML syntax :**

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: example-limit-range
spec:
  limits:
  - type: Container
    max:
      cpu: "1"
      memory: 1Gi
    min:
      cpu: "100m"
      memory: 100Mi
    default:
      cpu: "500m"
      memory: 500Mi
    defaultRequest:
      cpu: "200m"
      memory: 200Mi
    maxLimitRequestRatio:
      cpu: "4"

# In this example, the LimitRange is defining resource limits for containers 
# within a namespace. It specifies the maximum and minimum CPU and 
# memory limits, default values for CPU and memory, default CPU 
# and memory requests, and the maximum limit to request ratio for CPU.
```

LimitRanges are not enforced by default in Kubernetes. They need to be explicitly defined and applied to the namespace in order to take effect. Read more [here.](https://kubernetes.io/docs/concepts/policy/limit-range/)

**Commands:**

```bash
# We can see the Resource requests and limits by running
kubectl describe pod pod_name 
```

### Limit Ranges vs Resource Limits vs Resource Quotas

- LimitRanges: A LimitRange is a Kubernetes resource that allows you to specify default resource limits and usage constraints for pods in a namespace. It provides fine-grained control over the resource allocations for pods, such as setting maximum CPU and memory limits, minimum CPU and memory requests, and default values. LimitRanges ensure that pods adhere to the defined limits and prevent resource overutilization within a namespace.
- Resource Limits: Resource limits, specified within the **`resources`** section of a pod's configuration, define the maximum amount of CPU and memory that a container can consume. These limits are set at the container level and help ensure that the container does not exceed the specified resource boundaries. Resource limits are used to allocate resources based on the specific requirements of individual containers within a pod.
- Resource Quotas: Resource quotas, defined at the namespace level, establish limits on the aggregate resource consumption of all pods and other Kubernetes resources within a namespace. Quotas are used to manage and control resource allocation across multiple pods and enforce fairness and resource sharing within a namespace. Quotas can limit the overall CPU, memory, and storage capacity used by pods, as well as the number of objects, such as pods, services, and persistent volume claims, that can be created within a namespace.

In summary, LimitRanges are used to set constraints on individual pods, resource limits are used to define the maximum resource usage per container, and resource quotas provide limits on aggregate resource consumption within a namespace. These mechanisms work together to ensure resource utilization control and management in a Kubernetes cluster.

## NOTE on editing PODs and Deployments

Remember, you **CANNOT** edit specifications of an existing POD other than the below.

- spec.containers[*].image
- spec.initContainers[*].image
- spec.activeDeadlineSeconds
- spec.tolerations

For example you cannot edit the environment variables, service accounts, resource limits (all of which we will discuss later) of a running pod. But if you really want to, you have 2 options:

1. Run the `kubectl edit pod <pod name>` command.  This will open the pod specification in an editor (vi editor). Then edit the required properties. When you try to save it, you will be denied. This is because you are attempting to edit a field on the pod that is not editable.

!https://img-c.udemycdn.com/redactor/raw/2019-05-30_14-46-21-89ea56fea6b993ee0ccff1625b13341e.PNG

!https://img-c.udemycdn.com/redactor/raw/2019-05-30_14-47-14-07b2638d1a72cb2d5b000c00971f6436.PNG

A copy of the file with your changes is saved in a temporary location as shown above.

You can then delete the existing pod by running the command:

`kubectl delete pod webapp`

Then create a new pod with your changes using the temporary file

`kubectl create -f /tmp/kubectl-edit-ccvrq.yaml`

2. The second option is to extract the pod definition in YAML format to a file using the command

`kubectl get pod webapp -o yaml > my-new-pod.yaml`

Then make the changes to the exported file using an editor (vi editor). Save the changes

`vi my-new-pod.yaml`

Then delete the existing pod

`kubectl delete pod webapp`

Then create a new pod with the edited file

`kubectl create -f my-new-pod.yaml`

### **Edit Deployments**

With Deployments you can easily edit any field/property of the POD template. Since the pod template is a child of the deployment specification,  with every change the deployment will automatically delete and create a new pod with the new changes. So if you are asked to edit a property of a POD part of a deployment you may do that simply by running the command

`kubectl edit deployment my-deployment`

## DaemonSets

DaemonSet is a type of workload that ensures that a copy of a specific Pod is running on every node in the cluster. It ensures that the designated Pod is automatically created and scheduled on new nodes added to the cluster and terminated when nodes are removed.

Unlike other workload types like Deployments or ReplicaSets, which aim for a desired number of Pods across the cluster, a DaemonSet focuses on running a single copy of a Pod on each node.

DaemonSets are commonly used for tasks such as log collection, monitoring agents, network plugins, or other cluster-wide background tasks.

### YAML syntax:

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: example-daemonset
spec:
  selector:
    matchLabels:
      app: example
  template:
    metadata:
      labels:
        app: example
    spec:
      containers:
      - name: example-container
        image: example-image
        # Additional container configuration
      # Additional pod specification
  # Additional DaemonSet configuration
```

In this example:

- **`metadata`**: Specifies the metadata for the DaemonSet, including its name and labels.
- **`spec.selector`**: Specifies the label selector used to identify the Pods controlled by the DaemonSet.
- **`spec.template`**: Specifies the template for the Pods created by the DaemonSet.
- **`spec.template.spec.containers`**: Defines the containers to run within each Pod, including the container name and image.

### **Commands:**

```yaml
# To see list of daemonsets
kubectl get daemonsets or ds

# To get info about a daemonset
kubectl describe daemonset daemonset_name

# An easy way to create a DaemonSet is to first generate a YAML file for a 
# Deployment with the command :
kubectl create deployment deployment_name --image=image_name 
-n namespace_name --dry-run=client -o yaml > file.yaml
# Next, remove the replicas, strategy and status fields from the 
# YAML file using a text editor. 
# Also, change the kind from Deployment to DaemonSet.
# Finally, create the Daemonset by running 
kubectl create -f file.yaml
```

## Static PODs

Static Pods in Kubernetes are pods that are managed directly by the kubelet on a specific node, rather than being managed by the Kubernetes control plane. They are defined and configured locally on the node where they are running, and the kubelet is responsible for monitoring and managing their lifecycle.

Unlike other types of pods that are created and managed through the Kubernetes API server, static pods are created by placing their pod manifest files in a specific directory on the node's filesystem. The kubelet detects these manifest files and automatically creates and manages the associated pods.

```yaml
# There are 2 methods of passing the location of that directory
# in kubelet.service file.

# 1. Specifying the location directly in the service file:
--pod-manifest-path=/etc/kubernetes/manifests

# or

# 2. Provide a path of another config file using the config option:
--config=config_file_name.yaml
# And in the config_file_name.yaml
staticPodPath: /etc/kubernetes/manifests

# Clusters setup by KubeADM use the 2nd approach.

# So, in the labs, first run :
ps -aux | grep kubelet
# or
ps -aux | grep /usr/bin/kubelet
# Now to check for that directory, look for '--pod-manifest-path' first,
# and if that is not present, look for '--config' option and 
# open that config file and inspect 'staticPodPath' in it.

# An example to create a static pod:
kubectl run pod_name --restart=Never --image=image_name --dry-run=client 
-o yaml --command -- sleep 1000 > pod.yaml
# Now place the pod config file in specified directory.

# To see the static pods:
docker ps 
#or
kubectl get pods

# To remove the static pod, we need to remove pod_manifest file from the
# specified directory.
```

Static Pods are tightly bound to the node where they are running, and they cannot be scheduled or moved to different nodes by the Kubernetes scheduler. They are useful in scenarios where pods need to be managed directly on specific nodes, without going through the Kubernetes control plane.

The kubelet automatically tries to create a [mirror Pod](https://kubernetes.io/docs/reference/glossary/?all=true#term-mirror-pod) on the Kubernetes API server for each static Pod. This means that the Pods running on a node are visible on the API server, but cannot be controlled from there. We can see the mirror Pods (copies of Static Pods) on the API server: `kubectl get pods` 
The Static Pod names will be suffixed with the node hostname with a leading hyphen.
We cannot delete the static pod from the Kube API server ie. it deletes and restarts those pods automatically.

**NOTE:**
We cannot do the static deployment of ReplicaSets or Deployments or Services by placing them in a designated directory, as they require Controlplane components like Replication Controller, Deployment Controller etc. 

### Static Pods vs DaemonSets

![Screenshot 2023-06-22 at 4.15.01 PM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ff4ea520-5cf7-4ec7-8f67-67147e623dec/Screenshot_2023-06-22_at_4.15.01_PM.png)

Remember, setting up K8s cluster using the KubeADM tool deploys control plane components as static pods.

### Task

```bash
Task: A static pod is created named ‘xyz’. Find it and delete it.

Solution:

# 1. Get a list of pods by:
k get po
# In the output we'll see the pod 'static-xyz-node_name'

# 2. SSH to that node
ssh node_name

# 3. Check the kubelet service
ps -aux | grep kubelet

# 4. Now to check for that directory, look for '--pod-manifest-path' first,
# and if that is not present, look for '--config' option and 
# open that config file and inspect 'staticPodPath' in it.

# 5. Delete the pod manifest file from that directory.
```

## Multiple Schedulers

In Kubernetes, it is possible to configure multiple schedulers to handle the scheduling of pods. By default, Kubernetes uses the **`kube-scheduler`** as the default scheduler. However, you can deploy and use additional custom schedulers alongside the default scheduler.

NOTE: **All schedulers running in the cluster must have unique names.**

The **`--config`** option in a custom scheduler configuration file is used to specify the path to the scheduler's configuration file. The configuration file contains various settings and parameters that control the behavior of the custom scheduler.

In a Kubernetes cluster with multiple schedulers, leader election is used to ensure that only one scheduler is actively scheduling pods at a given time. Leader election helps avoid conflicts and ensures coordination between the schedulers.

The **`schedulerName`** field in a pod configuration file is used to specify the name of the scheduler that should be responsible for scheduling the pod. By default, pods are scheduled by the default scheduler in the Kubernetes cluster. However, if there are multiple schedulers running in the cluster, you can use the **`schedulerName`** field to indicate which specific scheduler should handle the scheduling of the pod.

You can look at the "Scheduled" entries in the event logs to verify that the pods were scheduled by the desired schedulers, by running`kubectl get events -o wide` .

You can also look at Scheduler logs in case we run into any issue, by running 
`kubectl logs scheduler_name --namespace=namespace_name` .

Read more at: https://kubernetes.io/docs/tasks/extend-kubernetes/configure-multiple-schedulers/

### LAB

```bash
# To see default scheduler in a K8s environment
kubectl get pods -n kube-system
# We can see the default scheduler deployed as a pod.

# To see the image used in the scheduler
kubectl describe pod scheduler_name -n kube-system
# Under Containers.Image option, we can see the image.

# Remember we need to have the authentication set up (service account,
# clusterRoleBinding etc) which the custom scheduler will make use of.

# We can create a ConfigMap which stores the scheduler's configuration file.
# The custom scheduler POD (or deployment) can mount the 
# custom scheduler configmap as a volume.
# Make sure we use right image in that Pod's (or Deployment's) config file.

# Now we can create any POD and use our custom scheduler to schedule it by
# mentioning the 'schedulerName' in Pod's config file.
```

## Scheduler Profiles

A scheduling Profile allows you to configure the different stages of scheduling in the [kube-scheduler](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/). Each stage is exposed in an extension point. Plugins provide scheduling behaviors by implementing one or more of these extension points.

Read more at: https://kubernetes.io/docs/reference/scheduling/config/

![Screenshot 2023-06-23 at 4.29.51 PM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c5644109-0c30-4a6f-b1c1-63bb64fb9cbb/Screenshot_2023-06-23_at_4.29.51_PM.png)

Pink Boxes are Extension Points, Blue boxes are Plugins.

# Logging & Monitoring

## Monitor Cluster Components

To monitor the:

- **`Node Level`** metrics such as the number of nodes in the cluster. How many of them are healthy as well as the performance metrics such as CPU, Memory, Network and Disk Utilization.
- As well as **`POD Level`** metrics such as the number of PODs, and performance metrics of each POD such as the CPU and Memory consumption on them.

We need a solution that will monitor these metrics store them and provide analytics around this data. We 

### Metrics Server

Metrics Server is a component in Kubernetes that collects resource metrics from various resources in the cluster, such as nodes, pods, and containers. It provides these metrics to the Kubernetes API server, which can then be used by various tools and components for monitoring and autoscaling purposes.

Note that the Metric Server is only an in memory monitoring solution and does not store the metrics on the disk and as a result you cannot see historical performance data.

### How are the metrics generated for the PODs?

- Kubernetes runs an agent on each node known as the kubelet, which is responsible for receiving instructions from the kubernetes API master server and running PODs on the nodes.
- The kubelet also contains a subcomponent known as CAdvisor or Container Advisor.
- CAdvisor is responsible for performance metrics from pods, and exposing them through the kubelet API to make the metrics available for the metrics server.

### Metrics Server deployment and commands

```bash
# Deploying Metrics Server in Minikube:
minikube addons enable metrics-server

# Deployment of Metrics Server in a K8s cluster:
# 1. Clone the metric server from github repo
git clone https://github.com/kubernetes-incubator/metrics-server.git
# 2. Deploy the metric server
kubectl create -f metric-server/deploy/1.8+/

# View the cluster performance
kubectl top node

# View performance metrics of pod
$ kubectl top pod
```

There are advanced tools available for monitoring a K8s cluster. There are number of opensource solutions available today, such as**`prometheus`**, **`Elastic Stack`**, **`Datadog`** and **`Dynatrace`**.

## Managing Application Logs

Monitoring application logs in Kubernetes is crucial for maintaining the health, performance, and security of your applications in a containerized environment.

The **`kubectl logs`** command is used to retrieve the logs of a specific container within a pod in Kubernetes.

```bash
# To view the logs of a pod:
kubectl logs -f pod_name
# -f flag enables the live trails of the logs

# The -f flag stands for "follow" and enables the streaming mode, 
# where new log entries are continuously fetched and 
# displayed as they are produced.

# If there are multiple containers in a pod:
kubectl logs -f pod_name -c container_name
# -c flag is used to specify the container name

# We use --tail to get the last log lines:
kubectl logs pod_name --tail <number>
# This will display the last <number> lines of logs.
```

### How do we run commands inside a container of a Pod?

The **`kubectl exec`** command is used to execute a command inside a running container of a pod.

```bash
kubectl exec [options] <pod_name> -- <command>
```

Here, **`<pod-name>`** refers to the name of the pod in which you want to execute the command, and **`<command>`** is the command you want to run inside the container. The **`--`** (double dash) is used to separate the **`kubectl`** options from the command to be executed inside the container.

You can also specify the container name using the **`-c`** flag if the pod has multiple containers. Additionally, you can use the **`-it`** flag for interactive mode, allowing you to enter an interactive shell session within the container.

For example:

```bash
kubectl exec -it my-pod -- /bin/bash
# This command opens an interactive shell session inside the 
# container of the my-pod pod, allowing you to run commands within it.
```