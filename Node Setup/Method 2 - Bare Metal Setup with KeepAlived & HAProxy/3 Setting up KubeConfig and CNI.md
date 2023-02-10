# 3 Setting up KubeConfig and CNI

### ****Set Up KUBECONFIG**

Choose a master node / control plane node. Run the ff. command to configure KubeConfig file.

```bash
export KUBECONFIG=/etc/kubernetes/admin.conf
```

Run the following command:

```bash
kubectl get nodes
```

We can see here that we have a cluster created, but the status is not ready. This is because we’re missing a CNI.

### Deploy a CNI

Container Network Interface or CNI.

There are a variety of networking interfaces that could be deployed. For this simple example we’ve used Calico.

Install the operator on your cluster.

```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.24.5/manifests/tigera-operator.yaml
```

Download the custom resources necessary to configure Calico.

```bash
curl https://raw.githubusercontent.com/projectcalico/calico/v3.24.5/manifests/custom-resources.yaml -O
```

Create the manifest in order to install Calico.

```bash
kubectl create -f custom-resources.yaml
```

Then run the following command:

```bash
kubectl get pods -A
```

Here, you can see that Calico is also installed as pods.

![Untitled](/Node%20Setup/Method%202%20-%20Bare%20Metal%20Setup%20with%20KeepAlived%20%26%20HAProxy/images/3%20Setting%20up%20K8s.png)

[Back to Master and Worker Nodes Set Up](/Node%20Setup/Method%202%20-%20Bare%20Metal%20Setup%20with%20KeepAlived%20%26%20HAProxy/2%20Master%20and%20Worker%20Node%20setup.md)