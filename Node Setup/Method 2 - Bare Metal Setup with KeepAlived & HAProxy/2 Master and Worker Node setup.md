# 2 Master and Worker Node setup

### ****Kubernetes EC2 Instance Setup****

Installing Kubernetes components to your EC2 Instances

Disable swap

**Note:** *sed command requires to run as root or use sudo command, else Permission Denied.*

```bash
swapoff -a
sed -i.bak -r 's/(.+ swap .+)/#\1/' /etc/fstab
```

Install Kubernetes components: Kubelet, Kubeadm, and Kubectl.

```bash
sudo apt-get update && sudo apt-get install -y apt-transport-https curl

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

sudo apt-get update

sudo apt-get install -y kubelet kubeadm kubectl

sudo apt-mark hold kubelet kubeadm kubectl
```

Install Docker and change the cgroup driver to systemd.

```bash
sudo apt install docker.io -y

cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF

sudo systemctl restart docker
sudo systemctl enable docker
```

************FOR MASTER Only:************

### ****Create a Kubeadm Config File****

Now that the AWS infrastructure is ready to go, we’re ready to start working on the Kubernetes pieces. 

The first section is dedicated to setting up a kubeadm.conf file. 

This file has instructions on how to setup the control plane components when we use kubeadm to bootstrap them. There are a ton of options that can be configured here, but we’ll use a simple example that has AWS cloud provider configs included.

Create a kubeadm.conf file based on the example below, using your own environment information.

```bash
---
apiServer:
  extraArgs:
    cloud-provider: aws
apiServerCertSANs:
- g2-elb-8a82a6e32d33306f.elb.us-west-2.amazonaws.com # ELB DNS Name
apiServerExtraArgs:
  endpoint-reconciler-type: lease
apiVersion: kubeadm.k8s.io/v1beta1
clusterName: g2 # Cluster Name
controlPlaneEndpoint: cp.theithollowlab.com # VIP DNS name
controllerManager:
  extraArgs:
    cloud-provider: aws
    configure-cloud-routes: 'false'
kind: ClusterConfiguration
kubernetesVersion: 1.25.4 # K8s version
networking:
  dnsDomain: cluster.local
  podSubnet: 192.168.0.0/16 # Pod subnet matching your CNI config
nodeRegistration:
  kubeletExtraArgs:
    cloud-provider: aws
```

**Note:**

*For the kubeadm.conf file, Change the VIP DNS name to the AWS network load balancer’s DNS name: [g2-elb-8a82a6e32d33306f.elb.us-west-2.amazonaws.com](http://g2-elb-8a82a6e32d33306f.elb.us-west-2.amazonaws.com/) (just like the apiServerCertSANs)*

*Change apiVersion to: [kubeadm.k8s.io/v1beta3](http://kubeadm.k8s.io/v1beta3)
There’s an error when running older version of kube components (v1beta1)*

Update the kubelet service configuration so that it knows about the AWS environment as well. Edit the `/etc/systemd/system/kubelet.service.d/10-kubeadm.conf` 
and add an additional configuration option.

```bash
--cloud-provider=aws
```

Once the configuration has been made, reset the kubelet daemon.

```
systemctl daemon-reload
```

********************************FOR MASTER Only:********************************

The time has come to set up the cluster. Log in to one of your control plane nodes which will become the first master in the cluster. We’ll run the kubeadm initialization with the kubeadm.conf file that we created earlier and placed in the /etc/kubernetes directory.

```
kubeadm init --config /etc/kubernetes/kubeadm.conf --upload-certs
```

It may take a bit of time for the process to complete. 

Kubeadm init is ensuring that our api-server, controller-manager, and etcd container images are downloaded,as well as creating certificates, which you should find in the /etc/kubernetes/pki directory.

When the process is done, you should receive instructions on how to add additional control plane nodes and worker nodes.

**Note:** *The results you will receive may be different from what is shown in this documentation.*

****************************Worker Nodes:****************************

******************************************************************To join worker nodes to the cluster******************************************************************

```bash
kubeadm join 192.168.1.97:6443 --token nb23zs.03358v68zvcbsy8c \
    --discovery-token-ca-cert-hash sha256:d865d644b071421a46e4a1c1f2dd87d771547f2ce8a756db7bf796b3ce8d98a3
```

******Master Nodes:******

********To add more master nodes / control plane nodes to the cluster********

```bash
kubeadm join 192.168.1.97:6443 --token nb23zs.03358v68zvcbsy8c \
    --discovery-token-ca-cert-hash sha256:d865d644b071421a46e4a1c1f2dd87d771547f2ce8a756db7bf796b3ce8d98a3 \
    --control-plane --certificate-key f659f2ab32a31f9721404b386c5817e9e4b6579f157eb33e6bca81ec6c7cb05f
```

![Untitled](/Node%20Setup/Method%202%20-%20Bare%20Metal%20Setup%20with%20KeepAlived%20%26%20HAProxy/images/2%20master%20and%20worker.png)

[Back to AWS Load Balancer](/Node%20Setup/Method%202%20-%20Bare%20Metal%20Setup%20with%20KeepAlived%20%26%20HAProxy/1%20AWS%20Load%20Balancer.md)

[Next: Setting up KubeConfig](/Node%20Setup/Method%202%20-%20Bare%20Metal%20Setup%20with%20KeepAlived%20%26%20HAProxy/3%20Setting%20up%20KubeConfig%20and%20CNI.md)