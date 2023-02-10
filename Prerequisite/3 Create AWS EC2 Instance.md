# 3. Create AWS EC2 Instance

We need two running worker nodes, and for that we need two instances as well for each node.

Launch an EC2 Instance

Name: **g2worker or g2master**

Amazon Machine Image: **Ubuntu Server 20.04**

Instance type: **t2.medium (master node)**, **t2.micro (Worker node)**

Key pair: ***< create your own keypair >*** 

Network settings:

VPC: ***used the academy vpc***

Subnet: ***academy public vpc***

Auto assign IP Address: **Enable**

Security group: **g2kube**

Advance Details:

IAM Role: **G2Kube-Role**

The click **“Launch Instance”**

![Untitled](3%20Create%20AWS%20EC2%20Instance%2048bf673f87324258b7ed99b82a5b651d/Untitled.png)

You have now created your worker and master nodes.

*Note: Since we need two worker nodes, so we created two instances. Same for the 3 master nodes = 3 instances.*

[Back to Creating AWS IAM Role](/Prerequisite/2%20Create%20AWS%20IAM%20Role.md)

[Back to Main Directory](/ReadMe.md)