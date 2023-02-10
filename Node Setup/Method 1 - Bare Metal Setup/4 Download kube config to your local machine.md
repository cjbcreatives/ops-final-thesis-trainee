# 4. Download kube config to your local machine

On your ELB / Host machine

```bash
mkdir ~/.kube
scp root@<master private ip address>:/etc/kubernetes/admin.conf ~/.kube/config
```

- make directory for your k8s
- scp -  copied from master node to your ELB machine
    - in order top copy via ssh, connect the two instances through SSH first.

Install kubeadm on your ELB instance

```bash
apt update && apt install -y kubeadm=1.19.2-00
```

After downloading, verify your clusters if kubeadm is installed. 

```bash
kubectl pods -A
```

[Back to Worker Node Set Up](/Node%20Setup/Method%201%20-%20Bare%20Metal%20Setup/3%20Worker%20Nodes%20Setup.md)
[Back to the main directory](/ReadMe.md)
