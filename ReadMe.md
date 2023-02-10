# G2Kube Case Study

[Problem Set](/Problem%20Set/Kubernetes__Setting_up_a_highly-available_cluster.pdf)

# Building High Availability (HA) Kubernetes Cluster

The emergence of containers has changed how we create, distribute, and manage software. Using containers, we can segregate the many services that make up an application and deploy those containers across a number of virtual and physical servers. As a result, a container orchestration tool is developed to automatically deploy, manage, scale, and make a container-based application available. Scalable container-based application deployment and management are made possible by Kubernetes.

Through the use of dynamic scheduling of containers, Kubernetes improves the stability and reliability of the distributed application built on containers. But how can you guarantee that Kubernetes itself remains operational when a part or its master nodes fail?

## Why do we need HA Kubernetes Cluster?

The goal of Kubernetes High Availability is to configure Kubernetes and all of its auxiliary components so that there is no single point of failure. While a multi-master cluster uses several master nodes, each of which has access to the same worker nodes, a single master cluster can easily fail. In a single master cluster, the crucial components, such as the API server and controller manager, are only present on the master node, and if it goes down, you are unable to add more services or pods, among other things. However, in a Kubernetes HA environment, these crucial parts are replicated over many masters, so even if one master fails, the cluster may still function thanks to the other masters.

## K8s High Availability Infrastructure

### **Minimal setup of the K8s Structure**

**Stacked ETCD cluster**

Instead of the external etcd setup, we decided to go with the stacked etcd configuration. This is to control the amount of instances running, and in turn, reduce costs incurred. The stacked etcd topology will have local etcd cluster members coupled with control planes on the same nodes. It is also simpler to set up and replicate, as compared to the external etcd setup.

**Load Balancer: Type and Algorithm**

**Type:** Elastic Load Balancer Instance or AWS Network Load Balancer

Algorithm: Round Robin

The **Round Robin** algorithm is the oldest and simplest scheduling algorithm. It is mostly used for multitasking, since it distributes the workload evenly among all available resources. This ensures no single resource is overworked, which could lead to more errors and issues down the line. Also known as round robin process scheduling.

**How it works:**
This algorithm executes processes in a cyclic manner.
Each process is first assigned a “time slice” or interval. The processes are lined up in a ready queue for the CPU to execute. During each time interval, if the process is completed or successfully executed, the process will then terminate. Else, it goes back into the ready queue and waits for the next turn of execution.

![Untitled](/images/0_MD-Structure1.png)

### Minimal setup of K8’s structure using AWS Load Balancer

Here, we used the AWS Load Balancer service to eliminate the single point of failure, which is the single load balancer instance.

![Untitled](/images/0_MD-Structure2.png)

**The inner working of the structure:**

1. The Kubernetes cluster receives workload from the client.
2. The AWS Load Balancer service detects the incoming client request. It checks the status of the master nodes and selects a healthy target. The client request is then forwarded to the selected target, in this case, one of the running master nodes.
3. The API server, located in the target or master node, receives and validates the client request. Once verified, it then starts processing the request. The master node components (controller manager, scheduler, and etcd) and the worker nodes (kubelet) are able to communicate via the API server to handle the workload.
4. The controller manager is in charge with maintaining the desired state of the cluster. It acts as a control loop that compares the actual state of the cluster with the desired state, and makes or requests changes to meet the desired specifications.
5. The scheduler monitors the status of the worker nodes. It also decides which worker node will be creating necessary resources, such as pods, for handling the workload.
6. The etcd stores critical data, including the state and configuration of the cluster. For a stacked etcd topology, the control plane and the etcd components are on the same node. The master node creates a local etcd member, which will then communicate with the API server within this particular master node only.
7. The kubelet is a node agent that runs on each of the worker nodes. It communicates via the API server, receives instructions from the scheduler, and then carries out necessary tasks. Each kubelet also sends updates about the status of its corresponding worker node.

## Amazon EKS (Amazon Elastic Kubernetes Service)

**External ETCD Cluster**

![Untitled](/images/0_EKS.png)

Amazon EKS runs a single tenant Kubernetes control plane for each cluster. The control plane infrastructure isn't shared across clusters or AWS accounts. The control plane consists of at least two API server instances and three `etcd` instances that run across three Availability Zones within an AWS Region. Amazon EKS:

- Actively monitors the load on control plane instances and automatically scales them to ensure high performance.
- Automatically detects and replaces unhealthy control plane instances, restarting them across the Availability Zones within the AWS Region as needed.
- Leverages the architecture of AWS Regions in order to maintain high availability. Because of this, Amazon EKS is able to offer an SLA for API server endpoint availability.

Amazon EKS uses Amazon VPC network policies to restrict traffic between control plane components to within a single cluster. Control plane components for a cluster can't view or receive communication from other clusters or other AWS accounts, except as authorized with Kubernetes RBAC policies. This secure and highly available configuration makes Amazon EKS reliable and recommended for production workloads.

## How-To Guide - Building the HA K8s Cluster

### Prerequisites

---

[1. Setting Up Repo](/Prerequisite/1%20Setting%20Up%20Repo.md)

[2. Creating AWS IAM Role](/Prerequisite/2%20Create%20AWS%20IAM%20Role.md)

[3. Creating AWS EC2 Instance](/Prerequisite/3%20Create%20AWS%20EC2%20Instance.md)

[4. Pushing images to ECR Repo](/Prerequisite/4%20Pushing%20images%20to%20ECR%20Repo%20-%20k8s%20components%20and%20app.md)

### Nodes Setup

---

[1 Bare Metal Setup](/Node%20Setup/Method%201%20-%20Bare%20Metal%20Setup/1%20Elastic%20Load%20Balancer%20Setup.md)

[2 Bare Metal with KeepAlived and HAProxy](/Node%20Setup/Method%203%20-%20with%20AWS%20ELB/Kubernetes%20cluster%20with%202%20HA%20proxy%20and%20keepalived%20setup.md)

[3 Bare Metal with AWS ELB](/Node%20Setup/Method%202%20-%20Bare%20Metal%20Setup%20with%20KeepAlived%20%26%20HAProxy/)

[4 Amazon EKS](/Node%20Setup/Method%204%20-%20Amazon%20EKS/Setting%20up%20a%20High-Availability%20Cluster%20Using%20the%20EKS.md)

## Monitoring Setup

[Prometheus Monitoring on AWS ELB](/Monitoring/Prometheus%20Monitoring%20on%20AWS%20ELB.md)

**Amazon EKS Monitoring**
- [Kubernetes Dashboard](/Monitoring/Kubernetes%20Dashboard.md)
- [Prometheus](/Monitoring/Prometheus.md)
- [Grafana](/Monitoring/Grafana.md)
- [Splunk](/Monitoring/Splunk%20Monitoring.md)
- [AWS Cloud Watch](/Monitoring/Cloud%20Watch.md)

## Testing guidelines


[Creating a deployment](/Testing%20Guide/Creating%20a%20deployment.md)

[HA Testing for Kubeadm](/Testing%20Guide/HA%20Testing.md)

[Running nginx app](/Testing%20Guide/Running%20nginx%20app.md)

[HA Testing for EKS](/Testing%20Guide/Amazon_EKS_HA_Testing_Commands.md)

## Troubleshoot Guide



*Disclaimer: Not all troubleshoot solutions we provide in this documentation are always applicable to your configuration.*

There are some tricky places along the way, below are a few common issues you may run into when standing up your Kubernetes cluster.

<details>
<summary>AWS Instance Type </summary>

**ERROR ENCOUNTERED:**

`TLS ERROR`

- the api server crashes because the resources provided is not enough.

`Error from server (Timeout)`

- Didn’t get any response due to insufficient resources.

**Challenges:**

Big data that is pulled in

- Different AZ

- from Load Balancer instance (HA Proxy) to upgrading AWS Network Load Balancer

************************Solution:************************  Upgraded the instance type.

</details>
<details>
<summary>AWS Testing different availability zones. </summary>

To test high availability, we experimented to put the master and worker nodes in different high availability zones.

- What happened before we changed to different kinds of availability zones
    - the nodes are in sync.
    - No problems occured to take note of.
- After assigning the instances to different kinds of AZ.
    - The data that is pulled from different AZ caused the Load balancer to crash, due to insufficient resource (instance type = t2.medium)

**Solution**: we upscaled the instance type to t2.large instead.

</details>
<details>
<summary>Different AMI images</summary>

**AMI: Ubuntu 20.04 LTS**

**Answer:**

Compatibility. We tried using Ubuntu 22.04 LTS and when running kubeadm init on one of the master nodes, we got the following error:

error execution phase addon/kube-proxy: error when creating kube-proxy service account: unable to create serviceaccount: Post "https: //[g2-elb-8a82a6e32d33306.elb.us-west-2.amazonaws.com](http://g2-elb-8a82a6e32d33306.elb.us-west-2.amazonaws.com/):6443/api/vi/namespaces/kube-system/serviceaccounts?timeout=10s": net/http: request canceled(Client. Timeout exceeded while awaiting headers)

</details>

<details>
<summary> “print-command” to join nodes got lost </summary>

To print a join command for a new worker node use:

```bash
kubeadm token create --print-join-command
```

But if you need to join a new control plane node, you need to recreate a new key for the control plane join command. This can be done with three simple steps:

- Re upload certificates in the already working master node with kubeadm init phase upload-certs --upload-certs. That will generate a new certificate key.
- Print join command in the already working master node with kubeadm token create --print-join-command.
- Join a new control plane node with
    
    <*****kubeadm join command>***** --control-plane --certificate-key <**********token from step 1>**********
    
    ```bash
    kubeadm join g2-elb-8a82a6e32d33306f.elb.us-west-2.amazonaws.com:6443 --token svfqy2.kxknmv0qge3j1j43 --discovery-token-ca-cert-hash sha256:309b624f59c963a910b647add2cf24fcc91c85ca0ae5dd97a211ceb9dd204c02
    ```
</details>

<details>
<summary>Missing EC2 Cluster Tags</summary>

If you’ve forgotten to add the kubernetes.io/cluster/<CLUSTERNAME> tag to your EC2 instances, kubeadm will fail stating issues with the kubelet.

![https://i2.wp.com/theithollow.com/wp-content/uploads/2020/01/no-ec2-tag-1.png?resize=1024%2C277&ssl=1](https://i2.wp.com/theithollow.com/wp-content/uploads/2020/01/no-ec2-tag-1.png?resize=1024%2C277&ssl=1)

The kubelet logs will include something like the following:

> Tag “KubernetesCluster” nor “kubernetes.io/cluster/…” not found; Kubernetes may behave unexpectedly.… failed to run Kubelet: could not init cloud provider “aws”: AWS cloud failed to find ClusterID
> 
</details>

<details>
<summary>Missing Subnet Cluster Tags</summary>

If you’ve forgotten to add the kubernetes.io/cluster/<clustername> tag to your subnets, then any LoadBalancer resources will be stuck in a “pending” state.

![https://i2.wp.com/theithollow.com/wp-content/uploads/2020/01/image-26.png?resize=897%2C67&ssl=1](https://i2.wp.com/theithollow.com/wp-content/uploads/2020/01/image-26.png?resize=897%2C67&ssl=1)

The controller managers will throw errors about a missing tags on the subnets.

> failed to ensure load balancer: could not find any suitable subnets for creating the ELB
</details>

<details>
<summary>Node Names Don’t Match Private DNS Names</summary>

If your hostname and private dns names don’t match, you’ll see error messages during the kubeadm init phase.

> Error writing Crisocket information for the control-plane node: timed out waiting for the condition.
> 

The kubelet will show error messages about the node not being found.

![https://i0.wp.com/theithollow.com/wp-content/uploads/2020/01/image-35.png?resize=575%2C45&ssl=1](https://i0.wp.com/theithollow.com/wp-content/uploads/2020/01/image-35.png?resize=575%2C45&ssl=1)

To fix this, update the nodes hostnames so that they match the private dns names.

1. two pods are still in pending state:
    - prometheus-server
    - prometheus-alertmanager
2. I manually created persistent volume for both.

```bash
[root@k8smaster1 ~]$ kubectl get pod -n monitoring
NAME                                             READY   STATUS    RESTARTS   AGE
prometheus-alertmanager-7757d759b8-x6bd7         0/2     Pending   0          44m
prometheus-kube-state-metrics-7f85b5d86c-cq9kr   1/1     Running   0          44m
prometheus-node-exporter-5rz2k                   1/1     Running   0          44m
prometheus-pushgateway-5b8465d455-672d2          1/1     Running   0          44m
prometheus-server-7f8b5fc64b-w626v               0/2     Pending   0          44m

```

```bash
[root@k8smaster1 ~]$ kubectl get pv
prometheus-alertmanager   3Gi        RWX            Retain           Available                                                                       22m
prometheus-server         12Gi       RWX            Retain           Available                                                                       30m
```

```bash
[root@k8smaster1 ~]$ kubectl get pvc -n monitoring
NAME                      STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
prometheus-alertmanager   Pending                                                     20m
prometheus-server         Pending                                                     20m
```

```bash
[root@k8smaster1 ~]$ kubectl describe pvc prometheus-alertmanager -n monitoring
Name:          prometheus-alertmanager
Namespace:     monitoring
StorageClass:
Status:        Pending
Volume:
Labels:        app=prometheus
               chart=prometheus-8.15.0
               component=alertmanager
               heritage=Tiller
               release=prometheus
Annotations:   <none>
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:
Access Modes:
VolumeMode:    Filesystem
Events:
  Type       Reason         Age                  From                         Message
  ----       ------         ----                 ----                         -------
  Normal     FailedBinding  116s (x83 over 22m)  persistentvolume-controller  no persistent volumes available for this claim and no storage class is set
Mounted By:  prometheus-alertmanager-7757d759b8-x6bd7

```

I am expecting the pods to get into running state

**!!!UPDATE!!!**

```bash
NAME                      STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS    AGE
prometheus-alertmanager   Pending                                      local-storage   4m29s
prometheus-server         Pending                                      local-storage   4m29s

```

```bash
[root@k8smaster1 prometheus_pv_storage]$ kubectl describe pvc prometheus-server -n monitoring
Name:          prometheus-server
Namespace:     monitoring
StorageClass:  local-storage
Status:        Pending
Volume:
Labels:        app=prometheus
               chart=prometheus-8.15.0
               component=server
               heritage=Tiller
               release=prometheus
Annotations:   <none>
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:
Access Modes:
VolumeMode:    Filesystem
Events:
  Type       Reason                Age                   From                         Message
  ----       ------                ----                  ----                         -------
  Normal     WaitForFirstConsumer  11s (x22 over 4m59s)  persistentvolume-controller  waiting for first consumer to be created before binding
Mounted By:  prometheus-server-7f8b5fc64b-bqf42

```

**!!UPDATE-2!!**

```bash
[root@k8smaster1 ~]$ kubectl get pods prometheus-server-7f8b5fc64b-bqf42 -n monitoring  -o yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2019-08-18T16:10:54Z"
  generateName: prometheus-server-7f8b5fc64b-
  labels:
    app: prometheus
    chart: prometheus-8.15.0
    component: server
    heritage: Tiller
    pod-template-hash: 7f8b5fc64b
    release: prometheus
  name: prometheus-server-7f8b5fc64b-bqf42
  namespace: monitoring
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: prometheus-server-7f8b5fc64b
    uid: c1979bcb-c1d2-11e9-819d-fa163ebb8452
  resourceVersion: "2461054"
  selfLink: /api/v1/namespaces/monitoring/pods/prometheus-server-7f8b5fc64b-bqf42
  uid: c19890d1-c1d2-11e9-819d-fa163ebb8452
spec:
  containers:
  - args:
    - --volume-dir=/etc/config
    - --webhook-url=http://127.0.0.1:9090/-/reload
    image: jimmidyson/configmap-reload:v0.2.2
    imagePullPolicy: IfNotPresent
    name: prometheus-server-configmap-reload
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /etc/config
      name: config-volume
      readOnly: true
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: prometheus-server-token-7h2df
      readOnly: true
  - args:
    - --storage.tsdb.retention.time=15d
    - --config.file=/etc/config/prometheus.yml
    - --storage.tsdb.path=/data
    - --web.console.libraries=/etc/prometheus/console_libraries
    - --web.console.templates=/etc/prometheus/consoles
    - --web.enable-lifecycle
    image: prom/prometheus:v2.11.1
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 3
      httpGet:
        path: /-/healthy
        port: 9090
        scheme: HTTP
      initialDelaySeconds: 30
      periodSeconds: 10
      successThreshold: 1
      timeoutSeconds: 30
    name: prometheus-server
    ports:
    - containerPort: 9090
      protocol: TCP
    readinessProbe:
      failureThreshold: 3
      httpGet:
        path: /-/ready
        port: 9090
        scheme: HTTP
      initialDelaySeconds: 30
      periodSeconds: 10
      successThreshold: 1
      timeoutSeconds: 30
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /etc/config
      name: config-volume
    - mountPath: /data
      name: storage-volume
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: prometheus-server-token-7h2df
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext:
    fsGroup: 65534
    runAsGroup: 65534
    runAsNonRoot: true
    runAsUser: 65534
  serviceAccount: prometheus-server
  serviceAccountName: prometheus-server
  terminationGracePeriodSeconds: 300
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - configMap:
      defaultMode: 420
      name: prometheus-server
    name: config-volume
  - name: storage-volume
    persistentVolumeClaim:
      claimName: prometheus-server
  - name: prometheus-server-token-7h2df
    secret:
      defaultMode: 420
      secretName: prometheus-server-token-7h2df
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2019-08-18T16:10:54Z"
    message: '0/2 nodes are available: 1 node(s) didn''t find available persistent
      volumes to bind, 1 node(s) had taints that the pod didn''t tolerate.'
    reason: Unschedulable
    status: "False"
    type: PodScheduled
  phase: Pending
  qosClass: BestEffort

```

Also I have the volumes created and assigned to local storage

```bash
[root@k8smaster1 prometheus_pv]$ kubectl get pv -n monitoring

prometheus-alertmanager   3Gi        RWX            Retain           Available                                               local-storage            2d19h
prometheus-server         12Gi       RWX            Retain           Available                                               local-storage            2d19h
```
</details>

<details>
<summary>To fix this pending state.</summary>

If you are in EKS, your node need to have the next permission

```
arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy
```

and the **Amazon EBS CSI Driver** Add-on.

**But in our case. our IAM user doesnt have the permission to add this policy to our IAM roles.** 

</details>

<details>
<summary>Calico CNI not working</summary>

Calico is an open source networking and network security solution for containers, virtual machines, and native host-based workloads. Calico supports a broad range of platforms including Kubernetes, OpenShift, Mirantis Kubernetes Engine (MKE), OpenStack, and bare metal services.

Container Network Interface and Classless Inter-Domain Routing Requirements of Calico to connect with Prometheus monitoring is not working due to compatibility issues.

**Solution**
version compatibility outdated —> new version
Always check the newest version in the calico website.

</details>

## FAQs

<details>
<summary>What does high-availability mean?</summary>
High-availability is a characteristic of a system to A highly available infrastructure aims for minimal downtime. In order to achieve this, every single point of failure (SPOF) within the cluster must be fixed or eliminated, such that, in the event where one of the component goes down, another backup component takes over to keep the infrastructure fully operational.
</details>

<details>
<summary> Why use “kubeadm” instead of “minikube” for the HA cluster?</summary>

**Minikube** is best used for educational purposes, but may also apply in development and testing. It basically implements a local single-node Kubernetes cluster.

**Kubeadm** builds a minimum viable, production-ready Kubernetes cluster that conforms to best practices. It is used to implement multi-node clusters with single or multiple masters.

Since minikube only deals with local single-node Kubernetes clusters, this hinders our goal in providing high availability with the multi-master Kubernetes architecture, which will then be hosted on cloud (AWS). The kubeadm tool, on the other hand, allows us to configure and add more nodes easily to an existing cluster, and thus, better supports the multi-master setup.

</details>

<details>
<summary>What is the use of keepalived and haproxy for the load balancers? </summary>

The keepalived is a service used to monitor load-balanced infrastructure in order to ensure high availability. It implements health checkers to manage and maintain servers and processes. The HAProxy, on the other hand, provides a load balancer and proxying solution for HTTP and TCP-based applications.
</details>

<details>
<summary> Can a multi-master setup be automatically considered as highly available? </summary>

Not necessarily. There’s a difference between high availability and multi-master. If you, for example, have three masters and only one Nginx instance in front load balancing to those masters, you have a multi-master cluster but not a highly available one because your Nginx load balancer instance can still go down at any time.

However, we have also used the AWS Elastic Load Balancer that can auto-scale upon demand, depending on how much traffic there is. This makes our load balancer highly available.
</details>

<details>
<summary> Using the design, is it possible for your Kubernetes cluster to have absolutely zero downtime?</summary>

No, because a high-availability (HA) cluster isn't completely fault-tolerant. If we're going to create a fault-tolerant (FT) system, we'd need to replicate physical servers/components as backup, and that would result in more expenses. Whereas FT aims to prevent any critical process/application from experiencing downtime, HA focuses on delivering high performance through minimal downtime.
</details>

<details>
<summary> Single-master config vs Multi-master config</summary>

In a single-master setup, the master node manages the etcd database, API server, controller manager and scheduler, as well as the worker nodes. However, if that single master node fails, all the worker nodes will fail as well, and the entire cluster will be lost.

In a multi-master setup, by contrast, having multiple master nodes provides high availability for a single cluster and improves network performance because all the masters behave like a unified data center.

A multi-master setup protects against a wide range of failure modes, from a loss of single worker node to the failure of the master node’s etcd service. By providing redundancy, a multi-master cluster serves a highly available system for your end users.
</details>

<details>
<summary>Elements of a Highly-Available Infrastructure </summary>
A highly available infrastructure is almost always operational - it minimizes potential service disruption for businesses and clients over a given period of time.

Here are the elements of a HA infrastructure:
[https://www.cisco.com/c/en/us/solutions/hybrid-work/what-is-high-availability.html#~infrastructure-elements](https://www.cisco.com/c/en/us/solutions/hybrid-work/what-is-high-availability.html#~infrastructure-elements)

**Fault-tolerance:**

A system's ability to keep operating even when software/hardware components fail. Aims for zero downtime.

**Failover:**

The process performed by a failing component can be moved to a backup component, one that's recommended to be off premises. (This is where the Availability Zone option comes in).

**Replication:**

Data must always be replicated and shared between nodes in the cluster, so that in the event that a data center fails, the backup data can be used to rebuild.

**Redundancy:**

The components in the HA cluster can perform the same tasks.
</details>

## Conclusion

High Availability is a crucial component of reliability engineering. It aims to strengthen system reliability and eliminate any potential single points of failure across the board. Although the initial installation may appear to be very complicated, HA provides stability and reliability for systems that need such benefits. One of the most important elements of creating a reliable infrastructure is using a highly available cluster.
