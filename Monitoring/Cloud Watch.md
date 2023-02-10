# Cloud Watch

****Quick Start with the CloudWatch agent and Fluentd****

To deploy the CloudWatch agent and Fluentd using the quick start, use the following command. The following setup contains a community supported FluentD container image. You can replace the image with your own FluentD image as long as it meets the FluentD image requirements.

```bash
curl https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/quickstart/cwagent-fluentd-quickstart.yaml | sed "s/{{cluster_name}}/cluster-name/;s/{{region_name}}/cluster-region/" | kubectl apply -f -
```

Add CloudWatch permissions to your Worker Cluster IAM Role

Click [`CloudWatchAgentServerPolicy`](https://us-east-1.console.aws.amazon.com/iam/home#/policies/arn:aws:iam::aws:policy/CloudWatchAgentServerPolicy) to your permissions.

![Untitled](/Monitoring/images/4%20ACW.png)

Go to AWS Cloud Watch and check the Insights you have in there…