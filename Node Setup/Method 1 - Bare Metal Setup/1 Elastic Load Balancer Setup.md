# 1. Elastic Load Balancer Setup

Install HAProxy

```bash
apt update && apt install -y haproxy
```

- HAProxy (High Availability Proxy) is an **open source proxy and load balancing server software.** It provides high availability at the network (TCP) and application (HTTP/S) layers, improving speed and performance by distributing workload across multiple servers.

Configure HAProxy

Append the following line to **/etc/haproxy/haproxy.cfg**

```bash
frontend kubernetes-frontend
    bind <elb ip>:6443
    mode tcp
    option tcplog
    default_backend kubernetes-backend

backend kubernetes-backend
    mode tcp
    option tcp-check
    balance roundrobin
    server kmaster1 <master1 private ip>:6443 check fall 3 rise 2
    server kmaster2 <master2 private ip>:6443 check fall 3 rise 2
    server kmaster3 <master3 private ip>:6443 check fall 3 rise 2
```

Restart HAProxy service

```bash
systemctl restart haproxy
```

[Back to the main directory](/ReadMe.md)

[Next Step: Master Node Set Up](/Node%20Setup/Method%201%20-%20Bare%20Metal%20Setup/2%20Master%20Nodes%20Set%20Up.md)