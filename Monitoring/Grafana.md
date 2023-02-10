# Grafana

****Grafana Kubernetes Manifests****

Clone the following and use it for the setup.

```bash
git clone https://github.com/bibinwilson/kubernetes-grafana.git
```

****Deploy Grafana On Kubernetes****

Create a file named `grafana-datasource-config.yaml`

```bash
vi grafana-datasource-config.yaml
```

```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: grafana-datasources
  namespace: monitoring
data:
  prometheus.yaml: |-
    {
        "apiVersion": 1,
        "datasources": [
            {
               "access":"proxy",
                "editable": true,
                "name": "prometheus",
                "orgId": 1,
                "type": "prometheus",
                "url": "http://prometheus-service.monitoring.svc:8080",
                "version": 1
            }
        ]
    }
```

<aside>
💡 **Note:**
The following data source configuration is for Prometheus. If you have more data sources, you can add more data sources with different YAMLs under the data section.

</aside>

Create the configmap using the following command.

```bash
kubectl create -f grafana-datasource-config.yaml
```

Create a file named `deployment.yaml`

```bash
vi deployment.yaml
```

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      name: grafana
      labels:
        app: grafana
    spec:
      containers:
      - name: grafana
        image: grafana/grafana:latest
        ports:
        - name: grafana
          containerPort: 3000
        resources:
          limits:
            memory: "1Gi"
            cpu: "1000m"
          requests: 
            memory: 500M
            cpu: "500m"
        volumeMounts:
          - mountPath: /var/lib/grafana
            name: grafana-storage
          - mountPath: /etc/grafana/provisioning/datasources
            name: grafana-datasources
            readOnly: false
      volumes:
        - name: grafana-storage
          emptyDir: {}
        - name: grafana-datasources
          configMap:
              defaultMode: 420
              name: grafana-datasources
```

<aside>
💡 **Note:**
This Grafana deployment **does not use a persistent volume.** If you restart the pod all changes will be gone. Use a persistent volume if you are deploying Grafana for your project requirements. It will persist all the configs and data that Grafana uses.

</aside>

Create the deployment

```bash
kubectl create -f deployment.yaml
```

Create a service file named `service.yaml`

```bash
vi service.yaml
```

Copy the following contents. This will expose Grafana on `NodePort`32000 or a Loadbalancer based on your requirement.

```bash
apiVersion: v1
kind: Service
metadata:
  name: grafana
  namespace: monitoring
  annotations:
      prometheus.io/scrape: 'true'
      prometheus.io/port:   '3000'
spec:
  selector: 
    app: grafana
  type: <NodePort or LoadBalancer >
  ports:
    - port: 3000
      targetPort: 3000
      nodePort: 32000
```

Create the service.

```bash
kubectl create -f service.yaml
```

Now you should be able to access the Grafana dashboard **using any node IP** on port `32000`or your LoadBalancer `ip address`. Make sure the port is allowed in the firewall to be accessed from your workstation.

```bash
http://<your-node-ip>:32000
http://<your-loadbalancer-ip>
```

Use the following default username and password to log in. Once you log in with default credentials, it will prompt you to change the default password.

![Untitled](Grafana%2044a83458711940fb95e067dba0cccc6e/Untitled.png)

Now you’re in!

![Untitled](/Monitoring/images/1%20Grafana.png)