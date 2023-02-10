# Why use “kubeadm” and not “minikube” cluster?

**minikube** is designed for learning, development and testing. It’s a fast and simple solution for deploying a single node cluster. 

**kubeadm** builds a minimum viable, production-ready Kubernetes cluster that conforms to best practices.

In a single master setup, the master node manages the etcd database, API server, controller manager and scheduler, along with the worker nodes. However, if that single master node fails, all the worker node fail as well and entire cluster will be lost.

In a multi-master setup, by contrast, multi-master provides high availability for a single cluster and improves network performance because all the masters behave like a unified data center.

A multi-master setup protects against a wide range of failure modes, from a loss of single worker node to the failure of the master node’s etcd service. By providing redundancy, a multi-master cluster serves a highly available system for your end users.