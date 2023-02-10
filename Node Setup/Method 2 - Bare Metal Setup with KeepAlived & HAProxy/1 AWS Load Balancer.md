# 1. AWS Load Balancer

## AWS Prerequisites

- 3 Master Nodes AWS Instance
- 3 Worker Nodes AWS Instance

### EC2 instances hostname must match the private DNS name assigned to it.

You can copy the private DNS name of your instance in the dashboard or run the ff. code in the CLI:

```bash
curl http://169.254.169.254/latest/meta-data/local-hostname
```

Use this command to set the hostname

```bash
hostnamectl set-hostname <private dns name>
```

### Create an Instance policy to your IAM Role

****************For Master Nodes / Control Plane:****************

To ensure the Control plane instances have access to AWS to spin up Load Balancers, EBS volumes, etc. we must apply an Instance Policy to the Control Plane nodes with the following IAM policy:

```bash
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "autoscaling:DescribeAutoScalingGroups",
        "autoscaling:DescribeLaunchConfigurations",
        "autoscaling:DescribeTags",
        "ec2:DescribeInstances",
        "ec2:DescribeRegions",
        "ec2:DescribeRouteTables",
        "ec2:DescribeSecurityGroups",
        "ec2:DescribeSubnets",
        "ec2:DescribeVolumes",
        "ec2:CreateSecurityGroup",
        "ec2:CreateTags",
        "ec2:CreateVolume",
        "ec2:ModifyInstanceAttribute",
        "ec2:ModifyVolume",
        "ec2:AttachVolume",
        "ec2:AuthorizeSecurityGroupIngress",
        "ec2:CreateRoute",
        "ec2:DeleteRoute",
        "ec2:DeleteSecurityGroup",
        "ec2:DeleteVolume",
        "ec2:DetachVolume",
        "ec2:RevokeSecurityGroupIngress",
        "ec2:DescribeVpcs",
        "elasticloadbalancing:AddTags",
        "elasticloadbalancing:AttachLoadBalancerToSubnets",
        "elasticloadbalancing:ApplySecurityGroupsToLoadBalancer",
        "elasticloadbalancing:CreateLoadBalancer",
        "elasticloadbalancing:CreateLoadBalancerPolicy",
        "elasticloadbalancing:CreateLoadBalancerListeners",
        "elasticloadbalancing:ConfigureHealthCheck",
        "elasticloadbalancing:DeleteLoadBalancer",
        "elasticloadbalancing:DeleteLoadBalancerListeners",
        "elasticloadbalancing:DescribeLoadBalancers",
        "elasticloadbalancing:DescribeLoadBalancerAttributes",
        "elasticloadbalancing:DetachLoadBalancerFromSubnets",
        "elasticloadbalancing:DeregisterInstancesFromLoadBalancer",
        "elasticloadbalancing:ModifyLoadBalancerAttributes",
        "elasticloadbalancing:RegisterInstancesWithLoadBalancer",
        "elasticloadbalancing:SetLoadBalancerPoliciesForBackendServer",
        "elasticloadbalancing:AddTags",
        "elasticloadbalancing:CreateListener",
        "elasticloadbalancing:CreateTargetGroup",
        "elasticloadbalancing:DeleteListener",
        "elasticloadbalancing:DeleteTargetGroup",
        "elasticloadbalancing:DescribeListeners",
        "elasticloadbalancing:DescribeLoadBalancerPolicies",
        "elasticloadbalancing:DescribeTargetGroups",
        "elasticloadbalancing:DescribeTargetHealth",
        "elasticloadbalancing:ModifyListener",
        "elasticloadbalancing:ModifyTargetGroup",
        "elasticloadbalancing:RegisterTargets",
        "elasticloadbalancing:DeregisterTargets",
        "elasticloadbalancing:SetLoadBalancerPoliciesOfListener",
        "iam:CreateServiceLinkedRole",
        "kms:DescribeKey"
      ],
      "Resource": [
        "*"
      ]
    }
  ]
}
```

**********************************For Worker Nodes:**********************************

Worker node EC2 instances also need an AWS Instance Profile assigned to them with permissions to the AWS control plane.

```bash
{
  "Version": "2012-10-17",
  "Statement": [
      {
          "Effect": "Allow",
          "Action": [
              "ec2:DescribeInstances",
              "ec2:DescribeRegions",
              "ecr:GetAuthorizationToken",
              "ecr:BatchCheckLayerAvailability",
              "ecr:GetDownloadUrlForLayer",
              "ecr:GetRepositoryPolicy",
              "ecr:DescribeRepositories",
              "ecr:ListImages",
              "ecr:BatchGetImage"
          ],
          "Resource": "*"
      } 
  ]
}
```

**************************************************Adding tags**************************************************

All EC2 instances (Worker and Control Plane nodes) must have a tag named `kubernetes.io/cluster/clustername` where <clustername> is a name you’ll give your cluster.

![Untitled](/Node%20Setup/Method%202%20-%20Bare%20Metal%20Setup%20with%20KeepAlived%20%26%20HAProxy/images/Untitled.png)

Subnets must have a tag named `kubernetes.io/cluster/<CLUSTERNAME>`, where CLUSTERNAME is the name you’ll give your cluster. This is used when new Load Balancers are attached to Availability Zones.

![Untitled](/Node%20Setup/Method%202%20-%20Bare%20Metal%20Setup%20with%20KeepAlived%20%26%20HAProxy/images/Untitled%201.png)

![Untitled](/Node%20Setup/Method%202%20-%20Bare%20Metal%20Setup%20with%20KeepAlived%20%26%20HAProxy/images/Untitled%202.png)

First go to EC2 > Target groups. Create a new target group.
Then add instances to this target group. Only include the master node instances.
Afterwards, go to EC2 > Load balancers, then create a new listener on this ELB.
Add the new target group to this listener.

![Untitled](/Node%20Setup/Method%202%20-%20Bare%20Metal%20Setup%20with%20KeepAlived%20%26%20HAProxy/images/Untitled%203.png)

[Back to the main directory](/ReadMe.md)

[Next: Master and Worker Nodes Set Up](/Node%20Setup/Method%202%20-%20Bare%20Metal%20Setup%20with%20KeepAlived%20%26%20HAProxy/2%20Master%20and%20Worker%20Node%20setup.md)