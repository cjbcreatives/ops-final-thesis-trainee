# Kubernetes cluster with 2 HA proxy and keepalived setup

Why is it impotant to have 2 load balancers? eventhough 1 load balancer is already have high availability? well what if 1 of your load balancer crash or shutdown? so you need a back up right? so here’s the process of setting up a 2 load balancer using HA proxy and keepalived setup.

 Virtual IP (172.16.16.100)

 Run these commands in your 2 load balancers

**Install Keepalived & Haproxy**

```jsx
 apt update && apt install -y keepalived haproxy
```

**Configure KeepAlived**

On both nodes create the health check script /etc/keepalived/check_apiserver.sh

 

```jsx
cat >> /etc/keepalived/check_apiserver.sh <<EOF
#!/bin/sh
```

```jsx
errorExit() {
echo "*** $@" 1>&2
exit 1
}
```

```jsx
curl --silent --max-time 2 --insecure [https://localhost:6443/](https://localhost:6443/) -o /dev/null || errorExit "Error GET [https://localhost:6443/](https://localhost:6443/)"
if ip addr | grep -q 172.16.16.100; then
curl --silent --max-time 2 --insecure [https://172.16.16.100:6443/](https://172.16.16.100:6443/) -o /dev/null || errorExit "Error GET [https://172.16.16.100:6443/](https://172.16.16.100:6443/)"
fi
EOF
```

```jsx
chmod +x /etc/keepalived/check_apiserver.sh
```

Create KeepAlived config /etc/keepalived/keepalived.conf

```jsx
cat >> /etc/keepalived/keepalived.conf <<EOF
vrrp_script check_apiserver {
script "/etc/keepalived/check_apiserver.sh"
interval 3
timeout 10
fall 5
rise 2
weight -2
}
```

```jsx
vrrp_instance VI_1 {
state BACKUP
interface eth1
virtual_router_id 1
priority 100
advert_int 5
authentication {
auth_type PASS
auth_pass mysecret
}
virtual_ipaddress {
172.16.16.100
}
track_script {
check_apiserver
}
}
EOF

```

**Enable & start KeepAlived service**

```jsx
systemctl enable --now keepalived
```

**Configure HAProxy**
Update **/etc/haproxy/haproxy.cfg**

```jsx
cat >> /etc/haproxy/haproxy.cfg <<EOF
```

```jsx
frontend kubernetes-frontend
bind *:6443
mode tcp
option tcplog
default_backend kubernetes-backend
```

```jsx
backend kubernetes-backend
option httpchk GET /healthz
http-check expect status 200
mode tcp
option ssl-hello-chk
balance roundrobin
server kmaster1 172.16.16.101:6443 check fall 3 rise 2
server kmaster2 172.16.16.102:6443 check fall 3 rise 2
server kmaster3 172.16.16.103:6443 check fall 3 rise 2
```

```jsx
EOF
```

**Enable & restart haproxy service**

```jsx
systemctl enable haproxy && systemctl restart haproxy
```

**Pre-requisites on all kubernetes nodes (masters & workers)**
**Disable swap**

```jsx
swapoff -a; sed -i '/swap/d' /etc/fstab
```

**Disable Firewall**

```jsx
systemctl disable --now ufw
```

**Enable and Load Kernel modules**

```jsx
{
cat >> /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF
```

```jsx
modprobe overlay
modprobe br_netfilter
}
```

**Add Kernel settings**

```jsx
{
cat >>/etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
EOF
```

```jsx
sysctl --system
}
```

**Install containerd runtime**

```jsx
{
apt update
apt install -y containerd apt-transport-https
mkdir /etc/containerd
containerd config default > /etc/containerd/config.toml
systemctl restart containerd
systemctl enable containerd
}
```

**Add apt repo for kubernetes**

```jsx
{
curl -s [https://packages.cloud.google.com/apt/doc/apt-key.gpg](https://packages.cloud.google.com/apt/doc/apt-key.gpg) | apt-key add -
apt-add-repository "deb [http://apt.kubernetes.io/](http://apt.kubernetes.io/) kubernetes-xenial main"
}
```

**Install Kubernetes components**

```jsx
{
apt update
apt install -y kubeadm=1.22.0-00 kubelet=1.22.0-00 kubectl=1.22.0-00
}
```

**On Master node 1**

```jsx
kubeadm init --control-plane-endpoint="172.16.16.100:6443" --upload-certs --apiserver-advertise-address=172.16.16.101 --pod-network-cidr=192.168.0.0/16
```

Copy the commands to join other master nodes and worker nodes

**Deploy Calico network**

```jsx
kubectl --kubeconfig=/etc/kubernetes/admin.conf create -f [https://docs.projectcalico.org/v3.18/manifests/calico.yaml](https://docs.projectcalico.org/v3.18/manifests/calico.yaml)
```

**Join other master nodes to the cluster**
- Use the respective kubeadm join commands you copied from the output of kubeadm init command on the first master.

**Join worker nodes to the cluster**
- Use the kubeadm join command you copied from the output of kubeadm init command on the first master****