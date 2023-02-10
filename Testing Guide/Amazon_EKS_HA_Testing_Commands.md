# Amazon EKS HA Testing (Commands)

### Prerequisites

Before starting, we need to access the repository where our application image is stored.

First open Docker on your computer. Then, log in to AWS Elastic Container Registry (ECR) via the AWS Command Line Interface (CLI).

```bash
aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin 431522258999.dkr.ecr.us-west-2.amazonaws.com
```

Next, access the ECR console:

************Amazon Elastic Container Registry console > Repositories > (repository name) > Images************

Click on the image name. Under **************Details**************, copy the image **URI**. Here is the image URI we used:

```bash
431522258999.dkr.ecr.us-west-2.amazonaws.com/marlon-academy-batch-4-app:fin
```

Afterwards, update the kubeconfig file for the existing cluster. Our cluster name is ****g2****.

```bash
aws eks update-kubeconfig --region us-west-2 --name g2
```

### Create a deployment with replicas of 3.

Create a namespace to organize new resources for the cluster.

```bash
kubectl create ns <namespace>
```

Next, create the deployment (using the image URI) on this namespace.

Create deployment with replicas of 3

```bash
kubectl create deploy <deployment> --image=<image-URI> --replicas=3 -n <namespace>
```

Check if the deployment is successfully created on your namespace.

```bash
kubectl get deploy -n <namespace>
```

Add the option **-o wide** to display the nodes where these pods are running on.

Get pods, to check which node the pods are running on.

```bash
kubectl get pods -n <namespace> -o wide
```

Choose one node to **cordon** (Mark node as unschedulable) and deploy another replicas of 3, and see if the node is still receiving workloads.

### Cordon one node and test if the node is still receiving workloads.

Display all the running nodes on your cluster.

```bash
kubectl get nodes
```

Choose one healthy node to cordon. This will mark the node as **********************Unscheduled**********************, so that, even if the selected node is still running, it can no longer accept additional workloads.

```bash
kubectl cordon <node>
```

Check if the chosen node’s status has changed from ****************Running**************** to **********************SchedulingDisabled**********************.

```bash
kubectl get nodes
```

Create a new deployment to test, using the same namespace from earlier.

```bash
kubectl create deploy <test-deployment> --image=nginx --replicas=3 -n <namespace>
```

Verify that this deployment was successfully created on the specified namespace.

```bash
kubectl get deploy -n <namespace>
```

Now, on the same namespace, check the status of all running pods. View using the option **************-o wide**************, to see if the cordoned node has accepted this new workload.

```bash
kubectl get pods -n <namespace> -o wide
```

As seen from the output, the cordoned node is still running, but it no longer accepts any new workload.

The **drain** the node (Drain node in preparation for maintenance) to evict all workloads on that node.

### Drain the cordoned node.

Although scheduling has been disabled for the cordoned node, there are still processes or pods running on this node. This workload may be transferred to another worker node, while the cordoned node is still under maintenance. This is done by draining the cordoned node.

```bash
kubectl drain <node>
```

Alternatively, if the output displays ************************************************************************cannot delete Daemonset-managed Pods************************************************************************, add the following options including a grace period of 300 seconds.

```bash
kubectl drain <node> --grace-period=300 --ignore-daemonsets=true
```

You may also come up with another error that says **cannot delete Pods with local storage**.

In this case, you can add the following option to the previous command:

```bash
--delete-emptydir-data
```

Check the pods on what node did it transfer.

Now that the node has been drained, check the pods on your namespace. See if the previous workload has been rescheduled on a different node.

```bash
kubectl get pods -n <namespace> -o wide
```

The cordoned node no longer contains any running pods.

**Un-cordon** the node (Mark node as schedulable)

### Uncordon the same node and test if new workloads are now being received.

Unmark the node so it can now receive new workloads from the scheduler.

```bash
kubectl uncordon <node>
```

get nodes and check the status.

Afterwards, check the status of all running nodes. The uncordoned node should’ve changed status from ****************SchedulingDisabled**************** to **************Running**************.

```bash
kubectl get nodes
```

Again, create a new deployment on the existing namespace.

Deploy nginx with replicas of 3

- check if the nodes is now receiving any workloads

```bash
kubectl create deploy <new-deployment> --image=nginx --replicas=3 -n <namespace>
```

Display all running pods on the same namespace.

```bash
kubectl get pods -n <namespace> -o wide
```

The output will now display that one of the pods is scheduled for the newly uncordoned node.

Expose one of the deployments (apps) to the internet with load balancer as the type of service.

### Expose the deployment (app) as a LoadBalancer service.

Expose one of the deployments on your namespace as a LoadBalancer service on the Internet.

```bash
kubectl expose <existing-deployment-name> -n <namespace> --port=80 --target-port=80 --name=<service-name> --type=LoadBalancer
```

Display running services on your namespace to check if the service is successfully created.

```bash
kubectl get svc -n <namespace>
```

Get the external IP address of this service from the output of the previous command.

Dig and check the answer section.

- Access the DNS of the load balancer to any browser.

Afterwards, use ******dig****** to verify that the deployment (app) has been exposed.

```bash
dig <external-ip-address>
```

Now, check the answer section that corresponds to the external IP address. The answer section should contain three corresponding outputs, each representing the replica or pod from the deployment.

Using your Internet browser, try and access the app using the provided external IP address.

This can also be accessed through the Amazon Elastic Kubernetes Service (EKS) console

Go to: **********************Amazon EKS console > Clusters > (your cluster’s name) > Resources tab > Service and Networking (left navigation pane) > Services > (your service name)**********************

Under ********Info******** simply click on the Load Balancer URL.

Go to EC2 and terminate an instance to test.

- Check the nodes using cli
- Check the website or dig the ip address

### Try testing by terminating one of the worker node instances.

Go to your **************************EC2 Dashboard************************** and click on ******************Instances******************. Select one of the running worker node instance. Open the **********Instance state********** dropdown menu and click on ******************Terminate******************.

Then, display all the running nodes through the **AWS Command Line Interface (CLI).**

```bash
kubectl get nodes
```

One of the worker nodes’ status should change from ****************Ready**************** to ****************NotReady****************.

Once it displays ****************NotReady****************, check if the app is still accessible via the Internet. This can be done by either visiting the website (using external IP address or Load Balancer URL) or by using the dig command on the AWS CLI.

Once the new instance or node is running and ready,

- Terminate the two remaining nodes/instance to see if the structure is highly available through the app installed in the worker nodes

### Terminate the two remaining worker node instances.

Whenever a node is terminated, AWS EKS automatically spins up a new instance. Wait for the cluster to finish launching this instance before proceeding.

Again, display all running nodes on your cluster.

```bash
kubectl get nodes
```

Once the worker node’s status changes from ****************NotReady**************** to **********Ready**********, go back to your **********************EC2 Dashboard********************** and terminate the two remaining worker nodes.

Now, wait for the two worker nodes to display ****************NotReady**************** under Status.

Afterwards, check whether the app or website is still accessible.