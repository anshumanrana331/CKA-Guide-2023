# Logging & Monitoring

## Monitor Cluster Components

To monitor:

- **`Node Level`** metrics such as the number of nodes in the cluster. How many of them are healthy as well as the performance metrics such as CPU, Memory, Network and Disk Utilization.
- As well as **`POD Level`** metrics such as the number of PODs, and performance metrics of each POD such as the CPU and Memory consumption on them.

We need a solution that will monitor these metrics store them and provide analytics around this data.

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

There are advanced tools available for monitoring a K8s cluster. There are number of opensource solutions available today, such as **`prometheus`**, **`Elastic Stack`**, **`Datadog`** and **`Dynatrace`**.

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