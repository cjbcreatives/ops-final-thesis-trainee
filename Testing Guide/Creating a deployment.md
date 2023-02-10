# Creating a deployment

Create a new namespace

```bash
kubectl create namespace <namespace-name>
```

To change the default namespace with your preferred namespace, do the following command:

```bash
kubectl config set-context $(kubectl config current-context) --namespace=<namespace-name>
```

Create a deployment

```bash
kubectl create nginx --image=nginx --replicas=4 --namespace=namespace-name
```

Check the pods created after the deployment

```bash
kubectl get pods -o wide -n namespace-name
```

### Shutting down node

```bash
kubectl cordon g2worker 
```

Once you check the nodes by running `kubectl get nodes` you’ll see the status of the worker node.

Result: The status of the worker node will be “Ready, SchedulingDisabled”

The node is still running but won’t accept any workload anymore.

```bash
kubectl drain g2worker
```

- only drain the running pods or deployments under the node.

![Untitled](/Testing%20Guide/images/1.png)

Instead run the command…

```bash
kubectl drain g2worker --grace-period=300 --ignore-daemonsets=true
```

Result: forcefully stop running pods in that node

- evicted the pods under the node
- pods under that node will be transferred to another master node

To bring back the node

```bash
kubectl uncordon g2worker
```

Note: It doesn’t mean that once the node comes up, the pods that were previously running on that node will also return. You have to do another deployment to place pods under the recovered node.