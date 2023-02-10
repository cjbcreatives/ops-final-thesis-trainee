# 3. Worker Nodes Setup

Once you can access to your worker instance, run the following first:

```bash
sudo apt update
```

```bash
sudo apt upgrade
```

The **update command** only gets the information about the latest version of packages available for your system. It doesn’t download or install any package.

It is the **apt upgrade** command that actually downloads and upgrades the package to the new version.

On kubernetes worker nodes, run the following commands:

Note: run as the root user

Disable firewall

```bash
ufw disable
```

Disable swap

Why is it necessary to “disable swap” ?

The idea of kubernetes is to tightly pack instances to as close to 100% utilized as possible. All deployments should be pinned with CPU/memory limits. So if the scheduler sends a pod to a machine it should never use swap at all. You don’t want to swap since it’ll slow things down.

**Its mainly for performance.**

```bash
swapoff -a; sed -i '/swap/d' /etc/fstab
```

to double check, if the command has no result, it means swapoff is working…

```bash
swapon -s
```

Update sysctl settings for K8s networking

```bash
cat >>/etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```

Install docker engine

```bash
{
  apt install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
  add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
  apt update && apt install -y docker-ce=5:19.03.10~3-0~ubuntu-focal containerd.io
}
```

Check the docker system status

```bash
systemctl status docker
```

### Setting up the Kubernetes Cluster

Add the apt repository

```bash
{
  curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
  echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list
}
```

Install the K8s components

```bash
apt update && apt install -y kubeadm=1.19.2-00 kubelet=1.19.2-00 kubectl=1.19.2-00
```

Before connecting to the master node, let’s change our hostname

- changed hostname to avoid confusion.

```bash
hostnamectl g2worker1
```

To join the worker node to the master node

Ask for the master node for the command/token, in order for the worker node to connect to the master node.

```bash
kubeadm join <master-node-ip>:6443 --token 78rkei.b5dy3oxmgf0m27xc \--discovery-token-ca-cert-hash sha256:eafda5f0542ecb0bc781984378464900bf3b59233e1d4bea1d5a0885fe0dd8b
```

[Back to Master Node Setup](/Node%20Setup/Method%201%20-%20Bare%20Metal%20Setup/2%20Master%20Nodes%20Set%20Up.md)

[Last Step: Add kube config to your local machine](/Node%20Setup/Method%201%20-%20Bare%20Metal%20Setup/4%20Download%20kube%20config%20to%20your%20local%20machine.md)

[Back to the main directory](/ReadMe.md)