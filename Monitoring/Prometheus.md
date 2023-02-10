# Prometheus

Reference:

[https://devopscube.com/setup-prometheus-monitoring-on-kubernetes/#:~:text=Prometheus is a high-scalable,helps with metrics and alerts](https://devopscube.com/setup-prometheus-monitoring-on-kubernetes/#:~:text=Prometheus%20is%20a%20high%2Dscalable,helps%20with%20metrics%20and%20alerts)

****Prometheus Kubernetes Manifest Files****

Clone the repo using the following command:

```bash
git clone https://github.com/techiescamp/kubernetes-prometheus
```

Execute the following command to create a new **namespace named monitoring**.

```
kubectl create namespace monitoring
```

<aside>
💡 Prometheus uses Kubernetes APIs to read all the available metrics from Nodes, Pods, Deployments, etc. For this reason, we need to create an RBAC policy with `read access`
to required API groups and bind the policy to the `monitoring` namespace.

</aside>

Create a file named `clusterRole.yaml`and copy the following RBAC role.

<aside>
💡 In the role, given below, you can see that we have added `get`, `list`, and `watch`permissions to nodes, services endpoints, pods, and ingresses. The role binding is bound to the monitoring namespace. If you have any use case to retrieve metrics from any other object, you need to add that in this cluster role.

</aside>

```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: prometheus
rules:
- apiGroups: [""]
  resources:
  - nodes
  - nodes/proxy
  - services
  - endpoints
  - pods
  verbs: ["get", "list", "watch"]
- apiGroups:
  - extensions
  resources:
  - ingresses
  verbs: ["get", "list", "watch"]
- nonResourceURLs: ["/metrics"]
  verbs: ["get"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: prometheus
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: prometheus
subjects:
- kind: ServiceAccount
  name: default
  namespace: monitoring
```

Create the role using the following command.

```bash
kubectl create -f clusterRole.yaml
```

****Create a Config Map To Externalize Prometheus Configurations****

All configurations for Prometheus are part of `prometheus.yaml` file and all the alert rules for Alertmanager are configured in `prometheus.rules`.

1. `prometheus.yaml`: This is the main Prometheus configuration which holds all the scrape configs, service discovery details, storage locations, data retention configs, etc)
2. `prometheus.rules`: This file contains all the Prometheus alerting rules

By externalizing Prometheus configs to a Kubernetes config map, you don’t have to build the Prometheus image whenever you need to add or remove a configuration. You need to update the config map and restart the Prometheus pods to apply the new configuration.

The config map with all the [Prometheus scrape config](https://raw.githubusercontent.com/bibinwilson/kubernetes-prometheus/master/config-map.yaml) and alerting rules gets mounted to the Prometheus container in `/etc/prometheus` locationas `prometheus.yaml` and `prometheus.rules` files.

Create a file called `config-map.yaml`and copy the file contents from here.

```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-server-conf
  labels:
    name: prometheus-server-conf
  namespace: monitoring
data:
  prometheus.rules: |-
    groups:
    - name: devopscube demo alert
      rules:
      - alert: High Pod Memory
        expr: sum(container_memory_usage_bytes) > 1
        for: 1m
        labels:
          severity: slack
        annotations:
          summary: High Memory Usage
  prometheus.yml: |-
    global:
      scrape_interval: 5s
      evaluation_interval: 5s
    rule_files:
      - /etc/prometheus/prometheus.rules
    alerting:
      alertmanagers:
      - scheme: http
        static_configs:
        - targets:
          - "alertmanager.monitoring.svc:9093"

    scrape_configs:
      - job_name: 'node-exporter'
        kubernetes_sd_configs:
          - role: endpoints
        relabel_configs:
        - source_labels: [__meta_kubernetes_endpoints_name]
          regex: 'node-exporter'
          action: keep
      
      - job_name: 'kubernetes-apiservers'

        kubernetes_sd_configs:
        - role: endpoints
        scheme: https

        tls_config:
          ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token

        relabel_configs:
        - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_service_name, __meta_kubernetes_endpoint_port_name]
          action: keep
          regex: default;kubernetes;https

      - job_name: 'kubernetes-nodes'

        scheme: https

        tls_config:
          ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token

        kubernetes_sd_configs:
        - role: node

        relabel_configs:
        - action: labelmap
          regex: __meta_kubernetes_node_label_(.+)
        - target_label: __address__
          replacement: kubernetes.default.svc:443
        - source_labels: [__meta_kubernetes_node_name]
          regex: (.+)
          target_label: __metrics_path__
          replacement: /api/v1/nodes/${1}/proxy/metrics     
      
      - job_name: 'kubernetes-pods'

        kubernetes_sd_configs:
        - role: pod

        relabel_configs:
        - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
          action: keep
          regex: true
        - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
          action: replace
          target_label: __metrics_path__
          regex: (.+)
        - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
          action: replace
          regex: ([^:]+)(?::\d+)?;(\d+)
          replacement: $1:$2
          target_label: __address__
        - action: labelmap
          regex: __meta_kubernetes_pod_label_(.+)
        - source_labels: [__meta_kubernetes_namespace]
          action: replace
          target_label: kubernetes_namespace
        - source_labels: [__meta_kubernetes_pod_name]
          action: replace
          target_label: kubernetes_pod_name
      
      - job_name: 'kube-state-metrics'
        static_configs:
          - targets: ['kube-state-metrics.kube-system.svc.cluster.local:8080']

      - job_name: 'kubernetes-cadvisor'

        scheme: https

        tls_config:
          ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token

        kubernetes_sd_configs:
        - role: node

        relabel_configs:
        - action: labelmap
          regex: __meta_kubernetes_node_label_(.+)
        - target_label: __address__
          replacement: kubernetes.default.svc:443
        - source_labels: [__meta_kubernetes_node_name]
          regex: (.+)
          target_label: __metrics_path__
          replacement: /api/v1/nodes/${1}/proxy/metrics/cadvisor
      
      - job_name: 'kubernetes-service-endpoints'

        kubernetes_sd_configs:
        - role: endpoints

        relabel_configs:
        - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scrape]
          action: keep
          regex: true
        - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scheme]
          action: replace
          target_label: __scheme__
          regex: (https?)
        - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_path]
          action: replace
          target_label: __metrics_path__
          regex: (.+)
        - source_labels: [__address__, __meta_kubernetes_service_annotation_prometheus_io_port]
          action: replace
          target_label: __address__
          regex: ([^:]+)(?::\d+)?;(\d+)
          replacement: $1:$2
        - action: labelmap
          regex: __meta_kubernetes_service_label_(.+)
        - source_labels: [__meta_kubernetes_namespace]
          action: replace
          target_label: kubernetes_namespace
        - source_labels: [__meta_kubernetes_service_name]
          action: replace
          target_label: kubernetes_name
```

Execute the following command to create the config map in Kubernetes.

```bash
kubectl create -f config-map.yaml
```

Now, it creates two files inside the container.

To learn more, check this link:

[https://devopscube.com/setup-prometheus-monitoring-on-kubernetes/#:~:text=Prometheus is a high-scalable,helps with metrics and alerts](https://devopscube.com/setup-prometheus-monitoring-on-kubernetes/#:~:text=Prometheus%20is%20a%20high%2Dscalable,helps%20with%20metrics%20and%20alerts)

****Create a Prometheus Deployment****

Create a file named `prometheus-deployment.yaml` and copy the following contents onto the file. In this configuration, we are mounting the Prometheus config map as a file inside `/etc/prometheus` as explained in the previous section.

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus-deployment
  namespace: monitoring
  labels:
    app: prometheus-server
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus-server
  template:
    metadata:
      labels:
        app: prometheus-server
    spec:
      containers:
        - name: prometheus
          image: prom/prometheus
          args:
            - "--storage.tsdb.retention.time=12h"
            - "--config.file=/etc/prometheus/prometheus.yml"
            - "--storage.tsdb.path=/prometheus/"
          ports:
            - containerPort: 9090
          resources:
            requests:
              cpu: 500m
              memory: 500M
            limits:
              cpu: 1
              memory: 1Gi
          volumeMounts:
            - name: prometheus-config-volume
              mountPath: /etc/prometheus/
            - name: prometheus-storage-volume
              mountPath: /prometheus/
      volumes:
        - name: prometheus-config-volume
          configMap:
            defaultMode: 420
            name: prometheus-server-conf
  
        - name: prometheus-storage-volume
          emptyDir: {}
```

Create a deployment on monitoring namespace using the above file.

```bash
kubectl create  -f prometheus-deployment.yaml
```

You can check the created deployment using the following command.

```bash
kubectl get deployments --namespace=monitoring
```

You can also get details from the kubernetes dashboard.

****Connecting To Prometheus Dashboard****

There are a lot of ways to connect to the dashboard, we will show you 3 ways on how to achieve this.

### ****Method 1: Using Kubectl port forwarding****

Using kubectl port forwarding, you can access a pod from your local workstation using a selected port on your `localhost`. This method is primarily used for debugging purposes.

First, get the Prometheus pod name.

```bash
kubectl get pods --namespace=monitoring
```

The output will look like the following

```bash
NAME                                     READY     STATUS    RESTARTS   AGE
prometheus-monitoring-3331088907-hm5n1   1/1       Running   0          5m
```

Execute the following command with your pod name to access Prometheus from localhost port 8080.

*Replace prometheus-monitoring-3331088907-hm5n1 with your pod name.*

```bash
kubectl port-forward prometheus-monitoring-3331088907-hm5n1 8080:9090 -n monitoring
```

Now, if you access `http://localhost:8080`on your browser, you will get the Prometheus home page.

## ****Method 2: Exposing Prometheus as a Service [NodePort & LoadBalancer]****

To access the Prometheus dashboard over a `IP`or a `DNS`name, you need to expose it as a Kubernetes service.

Create a file named `prometheus-service.yaml`and copy the following contents. We will expose Prometheus on all kubernetes node IP’s on port `30000`.

```bash
apiVersion: v1
kind: Service
metadata:
  name: prometheus-service
  namespace: monitoring
  annotations:
      prometheus.io/scrape: 'true'
      prometheus.io/port:   '9090'
spec:
  selector: 
    app: prometheus-server
  type: <NodePort or LoadBalancer> 
  ports:
    - port: 8080
      targetPort: 9090 
      nodePort: 30000
```

Create the service using the following command.

```bash
kubectl create -f prometheus-service.yaml --namespace=monitoring
```

**NodePort**

Once created, you can access the Prometheus dashboard using any of the Kubernetes nodes IP on port `30000`. If you are on the cloud, make sure you have the right firewall rules to access port `30000` from your workstation.

**LoadBalancer**

Once created, run the command `kubectl get svc -n monitoring` to get the Load Balancer ip address to run in a web browser.

To fully check that all the worker nodes are around run the the command

```bash
dig <load balancer ip address>
```

Now check if your prometheus dashboard is up.

![Untitled](/Monitoring/images/2%20Prom.png)