# 2. Master Nodes Set Up

Update and upgrade the master nodes.

```bash
sudo apt update
```

```bash
sudo apt upgrade
```

Disable firewall

```bash
ufw disable
```

Disable Swap

```bash
swapoff -a; sed -i '/swap/d' /etc/fstab
```

Update sysctl settings for Kubernetes networking

```bash
cat >>/etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```

Install Docker Engine

```bash
{
  apt install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
  add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
  apt update && apt install -y docker-ce=5:19.03.10~3-0~ubuntu-focal containerd.io
}
```

**KUBERNETES SET UP**

Add apt repository

```bash
{
  curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
  echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list
}
```

Install Kubernetes components

```bash
apt update && apt install -y kubeadm=1.19.2-00 kubelet=1.19.2-00 kubectl=1.19.2-00
```

### On any of the kubernetes cluster (pick one master node)

Initialize K8s cluster

```bash
kubeadm init --control-plane-endpoint="<load balancer ip>:6443" --upload-certs --pod-network-cidr=192.168.0.0/16
```

- A command will appear to join the other master nodes and worker nodes
- Copy the command specified to the master and worker nodes

Deploy Calico network

- Install Calico to provide both networking and network policy for self-managed on-premises deployments.

```bash
kubectl --kubeconfig=/etc/kubernetes/admin.conf create -f https://docs.projectcalico.org/v3.15/manifests/calico.yaml
```

To check if it’s installed, run the command 

```bash
kubectl get pods
```

[Back to the Elastic Load Balancer Setup](/Node%20Setup/Method%201%20-%20Bare%20Metal%20Setup/1%20Elastic%20Load%20Balancer%20Setup.md)

[Next Step: Worker Node Setup](/Node%20Setup/Method%201%20-%20Bare%20Metal%20Setup/3%20Worker%20Nodes%20Setup.md)


[Back to the main directory](/ReadMe.md)
