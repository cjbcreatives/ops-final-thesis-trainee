# HA Testing

- Shutdown the master node 2
    - reason in shutting down: if there’s a faulty hardware, change of os, etc.
    - Bring up the master node 2 back
        - systemctl status kube-apiserver
        - systemctl is-enabled kube-apiserver
- Shutdown the master node 1
    - before…master 1 has created a new deployment,  and master 2 is now in synced with the other master nodes…
    - Now master 2 has no idea about the new deployment
- Shutdown 2 master nodes
    - cannot be 2 shutdown master node:1 master node up
        - will not get pods
    - Note: See documentation about “best practices for replicating masters for HA cluster”
- Shutdown worker node 1, then 2
- Shutdown the api-server
- Shutdown the kubelet
- Shutdown app = nginx web server
- Shutdown callico

### Create Deployments

Create deployments

```bash
kubectl create deploy nginx --image=nginx --replicas=3 -n test
```

Check deployments

```bash
kubectl get deploy -A
```

![Untitled](/Testing%20Guide/images/1%20HA%20testing.png)

 Check the pods

```bash
kubectl get pods -n test
kubectl get pods -n test -o wide
```

![Untitled](/Testing%20Guide/images/2%20HA%20testing.png)

Check the nodes

```bash
kubectl get nodes
```

![Untitled](/Testing%20Guide/images/3%20HA%20testing.png)

### ********************Shutting down the worker node********************

- [ ]  Stop one of the worker node service
    - [ ]  Create another set of deployments
        - Testing if the worker node is still accepting workloads
    
    Once checked that it isn’t accepting any workloads…
    
    - [ ]  Drain the worker node
    - [ ]  Check if the pods and apps running on that node they’re still on that worker node or they’re transferred.

### Start up the worker node again

- [ ]  Bring up the worker node again
- [ ]  Check the worker node status
- [ ]  To test if that worker node can receive workloads, create deployments.
- [ ]  Get pods to check pods to see if that worker node is receiving workloads

### Stop the master node and its components

- [ ]  Stop one of the master node service by stopping the docker and kubelet service
- [ ]  Get nodes and check the status of your master node (NodeNotReady)
- [ ]  Check if you’re cluster is still functioning and can take up workloads even though one of the master nodes or control plane is down.
    - [ ]  Create deployments to add workloads to your cluster
    - [ ]  Check your pods if the cluster is still functioning
- [ ]  Delete the master node to make sure that the k8s components inside the node, can still function without it

### Deleting a worker node

- [ ]  Delete worker node, to see if the apps inside our cluster are running

Try to access one of the apps if it’s working

- [ ]  Expose the app deployment to the internet
- [ ]  Access the web app by using the ip in any of the master nodes