# Prometheus Monitoring on AWS ELB

**Prometheus and Grafana setup**

Add the Helm repo for Prometheus and Grafana.

```jsx
 helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

```jsx
helm repo add grafana https://grafana.github.io/helm-charts
```

```jsx
helm repo update

```

Create a [namespace](https://www.containiq.com/post/kubernetes-namespaces) for monitoring.

```jsx
kubectl create namespace monitoring

```

Install the Prometheus chart in the monitoring namespace.

```jsx
helm install prometheus prometheus-community/prometheus \
--namespace monitoring \
--set alertmanager.persistentVolume.storageClass="default" \
--set server.persistentVolume.storageClass="default"
```

Specify the data source and values in the [YAML](https://www.containiq.com/post/yaml-kubernetes-objects-and-configurations) because when Grafana starts up, it needs to automatically find the data source, which, in this case, is the Prometheus endpoint. It looks like this:

```jsx
datasources:
datasources.yaml:
apiVersion: 1
datasources:
- name: Prometheus
type: prometheus
url: http://<prometheus-server>.<monitoring_namespace>.<svc.cluster.local>
access: proxy
isDefault: true
```

Install the Grafana chart with the arguments below. Please change the admin password for the login.

```jsx
helm install grafana grafana/grafana \
--namespace monitoring \
--set persistence.storageClassName="default" \
--set persistence.enabled=true \
--set adminPassword='Yourfavpassword' \
--values grafana.yaml \
--set service.
```

```jsx
**type**
```

```jsx
=LoadBalancer
```

![Untitled](/Monitoring/images/Untitled.png)

![Untitled](/Monitoring/images/Untitled%201.png)

**ROAD BLOCK:**

- We’ve tried a lot of process or ways to set up prometheus in our cluster but this one’s the closest, it got connected to our cluster but the state metrics is down eventhough all the nodes are healthy.
- we tried all the possible troubleshoots, like CNI and cidr but still it don’t works.
- our conclusion is that a lot of conflict on a kubeadm cluster setup.
- we believe we can achieve this setup but we need more time.