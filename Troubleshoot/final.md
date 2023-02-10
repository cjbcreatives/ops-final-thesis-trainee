# final

There are some tricky places along the way, below are a few common issues you may run into when standing up your Kubernetes cluster.

### AWS Instance Type

************************ERROR ENCOUNTERED:************************

`TLS ERROR`

- the api server crashes because the resources provided is not enough.

`Error from server (Timeout)`

- Didn’t get any response due to insufficient resources.

**Challenges:**

Big data that is pulled in

- Different AZ

- from Load Balancer instance (HA Proxy) to upgrading AWS Network Load Balancer

************************Solution:************************  Upgraded the instance type.

---

## AWS Testing different availability zones.

To test high availability, we experimented to put the master and worker nodes in different high availability zones.

- What happened before we changed to different kinds of availability zones
    - the nodes are in sync.
    - No problems occured to take note of.
- After assigning the instances to different kinds of AZ.
    - The data that is pulled from different AZ caused the Load balancer to crash, due to insufficient resource (instance type = t2.medium)

**Solution**: we upscaled the instance type to t2.large instead.

---

### Different AMI images

**AMI: Ubuntu 20.04 LTS**

**Answer:**

Compatibility. We tried using Ubuntu 22.04 LTS and when running kubeadm init on one of the master nodes, we got the following error:

error execution phase addon/kube-proxy: error when creating kube-proxy service account: unable to create serviceaccount: Post "https: //[g2-elb-8a82a6e32d33306.elb.us-west-2.amazonaws.com](http://g2-elb-8a82a6e32d33306.elb.us-west-2.amazonaws.com/):6443/api/vi/namespaces/kube-system/serviceaccounts?timeout=10s": net/http: request canceled(Client. Timeout exceeded while awaiting headers)

---

### “print-command” to join nodes got lost

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
    

---

### **Missing EC2 Cluster Tags**

If you’ve forgotten to add the kubernetes.io/cluster/<CLUSTERNAME> tag to your EC2 instances, kubeadm will fail stating issues with the kubelet.

![https://i2.wp.com/theithollow.com/wp-content/uploads/2020/01/no-ec2-tag-1.png?resize=1024%2C277&ssl=1](https://i2.wp.com/theithollow.com/wp-content/uploads/2020/01/no-ec2-tag-1.png?resize=1024%2C277&ssl=1)

The kubelet logs will include something like the following:

> Tag “KubernetesCluster” nor “kubernetes.io/cluster/…” not found; Kubernetes may behave unexpectedly.… failed to run Kubelet: could not init cloud provider “aws”: AWS cloud failed to find ClusterID
> 

---

### **Missing Subnet Cluster Tags**

If you’ve forgotten to add the kubernetes.io/cluster/<clustername> tag to your subnets, then any LoadBalancer resources will be stuck in a “pending” state.

![https://i2.wp.com/theithollow.com/wp-content/uploads/2020/01/image-26.png?resize=897%2C67&ssl=1](https://i2.wp.com/theithollow.com/wp-content/uploads/2020/01/image-26.png?resize=897%2C67&ssl=1)

The controller managers will throw errors about a missing tags on the subnets.

> failed to ensure load balancer: could not find any suitable subnets for creating the ELB
> 

---

### **Node Names Don’t Match Private DNS Names**

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

---

### To fix this pending state.

If you are in EKS, your node need to have the next permission

```
arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy
```

and the **Amazon EBS CSI Driver** Add-on.

**But in our case. our IAM user doesnt have the permission to add this policy to our IAM roles.** 

---

### Calico

Calico is an open source networking and network security solution for containers, virtual machines, and native host-based workloads. Calico supports a broad range of platforms including Kubernetes, OpenShift, Mirantis Kubernetes Engine (MKE), OpenStack, and bare metal services.

outdated version compatibility —> new version

These two reasons why we did not achieve monitoring on our kubeadm setup

Container Network Interface and Classless Inter-Domain Routing Requirements of Calico to connect with Prometheus monitoring is not working due to compatibility issues.

We tried to adjust the scrape so that the exporter  will not take long to respond due to networking/firewall issues but it is still not working even though we are running curl with exactly the same URL that prometheus is running but it is slow to return data and after a while the cluster got crash and don’t return data at all.
