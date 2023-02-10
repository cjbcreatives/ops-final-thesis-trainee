# Setting up a High-Availability Cluster Using the Elastic Kubernetes Service (EKS)

## **Outline**

### Prerequisites

[Step 1 - Create the Amazon EKS cluster](#step-1-create-the-amazon-eks-cluster)

[Step 2 - Enable your computer to communicate with the cluster](#step-2-enable-your-computer-to-communicate-with-the-cluster)

[Step 3 - Create nodes under a managed node group](#step-3-create-nodes-under-a-managed-node-group)

[Step 4 - View resources in your Kubernetes cluster](#step-4-view-resources-in-your-kubernetes-cluster)

## Prerequisites

Before creating the cluster with EKS, install the following command line tools on your device first:

### kubectl

```bash
#Check if kubectl is already installed on your device
kubectl version | grep Client | cut -d : -f 5

#Install or update kubectl (for amd64) using the latest version
curl -o kubectl https://s3.us-west-2.amazonaws.com/amazon-eks/1.24.7/2022-10-31/bin/linux/amd64/kubectl

#Apply execute permissions to the binary
chmod +x ./kubectl

#Copy the binary to a folder in your PATH
mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin

#Verify kubectl version after installing/updating
kubectl version --short --client
```

### eksctl

```bash
#Check if eksctl is already installed on your device
eksctl version

#Download and extract the latest release of eksctl
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp

#Move the extracted binary to /usr/local/bin
sudo mv /tmp/eksctl /usr/local/bin

#Verify eksctl version after installing/updating
eksctl version
```

## Step 1. Create the Amazon EKS cluster

### VPC

First, create an Amazon VPC with public and private subnets that meets Amazon EKS requirements. In our case, the **********************academy-vpc********************** was already provided by The Academy.

![Untitled](/Node%20Setup/Method%204%20-%20Amazon%20EKS/images/Untitled.png)

### IAM Role

Next, create the IAM role for the EKS cluster.

Open the **Identity and Access Management (IAM) Console**. Then, select ************Roles************ from the options under ************************************Access management************************************, as shown here on the left side of the panel.

![Untitled](/Node%20Setup/Method%204%20-%20Amazon%20EKS/images/Untitled%201.png)

Click on ************************Create role************************ to create a new IAM role for your cluster.

![Untitled](/Node%20Setup/Method%204%20-%20Amazon%20EKS/images/Untitled%202.png)

Attach the required Amazon EKS IAM managed policy to this IAM Role.

```bash
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "eks.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

Alternatively, you can edit the trust policy configuration when creating the cluster’s IAM role.

For ****************Step 1****************, simply select **********************AWS service********************** as the trusted entity type. Afterwards, open the dropdown menu for ****************Use case**************** and click on ******EKS******. Then, select ********************EKS - Cluster********************.

![Untitled](/Node%20Setup/Method%204%20-%20Amazon%20EKS/images/Untitled%203.png)

For ********Step 2********, proceed to add permissions policies for the IAM role. The ********************************************AmazonEKSClusterPolicy******************************************** will be automatically added, since this is required by default.

![Untitled](/Node%20Setup/Method%204%20-%20Amazon%20EKS/images/Untitled%204.png)

For ************Step 3************, enter a role name. You can also include a short description.

IAM role name (EKS cluster): **************************g2clusterrole**************************

![Untitled](/Node%20Setup/Method%204%20-%20Amazon%20EKS/images/Untitled%205.png)

Click on ******************Create role****************** to finish the setup.

Go to **IAM Console > Roles** to verify that the role was successfully created.

![Untitled](/Node%20Setup/Method%204%20-%20Amazon%20EKS/images/Untitled%206.png)

### Amazon EKS Console

Go to the **Amazon Elastic Kubernetes Service (EKS) Console** and select ********************************Clusters******************************** on the left side of the navigation pane.

![Untitled](/Node%20Setup/Method%204%20-%20Amazon%20EKS/images/Untitled%207.png)

Choose the AWS region where the cluster will be created.

Selected region: ******************us-west-2******************

![Untitled](/Node%20Setup/Method%204%20-%20Amazon%20EKS/images/Untitled%208.png)

Click on **********************Add cluster********************** and select ************Create************ to configure a new cluster.

![Untitled](/Node%20Setup/Method%204%20-%20Amazon%20EKS/images/Untitled%209.png)

For **************Step 1**************, enter the name of your cluster.

Choose the latest Kubernetes version and then add the recently created IAM role to serve as the cluster service role. Leave the remaining setting values as default and click ******Next******.

Cluster name: ****g2****

Kubernetes version: ********1.24********

Cluster service role: **************************g2clusterrole**************************

![Untitled](/Node%20Setup/Method%204%20-%20Amazon%20EKS/images/Untitled%2010.png)

For ************Step 2************, edit the networking properties for your cluster.

Select the previously created VPC (**********************academy-vpc**********************) to be used for the cluster resources.

Next, select all corresponding public subnets under **********************academy-vpc**********************.

Afterwards, apply the corresponding security groups.

VPC:

**********************academy-vpc**********************

Subnets:

**academy-vpc-public-us-west-2a**

**academy-vpc-public-2b**

**academy-vpc-public-2c**

Security groups:

********************g2kube********************

![Untitled](/Node%20Setup/Method%204%20-%20Amazon%20EKS/images/Untitled%2011.png)

For **************Steps 3 to 5**************, leave the default values as is and then choose **********Next.**********

For ******Step 6******, review your configuration settings.

![Untitled](/Node%20Setup/Method%204%20-%20Amazon%20EKS/images/Untitled%2012.png)

Click on ************Create************ to finish the cluster setup.

Go to **************************EKS > Clusters************************** to check if the cluster was successfully created and is currently ************Active************.

![Untitled](/Node%20Setup/Method%204%20-%20Amazon%20EKS/images/Untitled%2013.png)

You may also click on the cluster name (******g2******) to view additional details.

![Untitled](/Node%20Setup/Method%204%20-%20Amazon%20EKS/images/Untitled%2014.png)

## Step 2. Enable your computer to communicate with the cluster.

Make sure you can access the AWS Command Line Interface (CLI) on your computer.

Wait for the cluster to finish creating, before proceeding with the following steps.

Apply the AWSPowerUserAccess credentials via the terminal.

```bash
export AWS_ACCESS_KEY_ID="ASIAWI6F3EA3RPKZGSHA"
export AWS_SECRET_ACCESS_KEY="SfX1ESqUrjWT2F7hS/oepbzSwXx0emBRSpgWGiCs"
export AWS_SESSION_TOKEN="IQoJb3JpZ2luX2VjEND//////////wEaCXVzLXdlc3QtMiJHMEUCIBsCthZUOExgpa3O0kFzfs0YdBWSe4TIA5vkAN+nGA55AiEA//EcK7c+xiAPt21ytpdF1R0puUfgfSR8TVj54BpAEL4qowMI6f//////////ARABGgw0MzE1MjIyNTg5OTkiDNmcOhsx0fSWcfmWiir3Ak/jGXMpQltEhYXH4W7lypzK26FUpt3v4WWKqW6MgIPXwXnYLi5OC0k6Qh2Ed1Rprh2GCZ3OafsK2xbfRe9/zz5T/Te09cxQDbtCehjjBn/UCIjKoBQaUueX1WxTgrjotJViY4D3TKfBrVWzbU6wSlF43Tqvh/PM7lL+6IP0VQSj6V+e6VCqd9kvqQthudxiijmp4efiPj+I4BO33foJhiay+PMD0xPFtz6FmjJHzjHotKwxlFYYWADDBBpRMsPhOCM5BYCn1tRbNPcedXx/ZskdFRIIHu1JCZ4h3JvEW4nVxx0/AcY2YX+ZxZCcQpFbNl3rzSsTkZrLulQyZbS2ofViHqxUtz3NaTI7a+M3Jsi+jNKvqBQvp0ok/vXPgcGK1MM1K6RhWIP0cVbWTPZxTqj5VVmvaCS4mID7KuGoqVvGQ1WmjngVUeLSl9f49q9wZr3tIwuGBCiJxO6VG6R1DnOL8ttz3YelvTHMUdkc6SKQZ3Oh1xaQADDBu7acBjqmAYuCwhiBwrXd79IVD43W9dvczkB1DZGbwRkXDDEoPc5svMqhPzb/SU+N/9Oqs/ZW16cC8S6jUgleoyKQLnwsdyCbvXK852FXnGzxQGkYZGq8mpHJJPHICtW7KjvW8ZpT1gxudjrJsSTef9A7x2YhsrtZ4KTp3y5D/Tro7pneTSBzdxd9ahZn6KI2ZMHSfK95c4LXFMgHEQDqgQxFUd0PdLf1YpXXYNI="
```

Check to see if you’re currently logged in.

```bash
aws sts get-caller-identity
```

Next, to enable communication between kubectl CLI and your cluster, create a ********************kubeconfig******************** file.

AWS region code: ******************us-west-2******************

Cluster name: ****g2****

```bash
aws eks update-kubeconfig --region region-code --name my-cluster
```

This can also be verified by reading the contents of the **************config************** file located in ~/.kube

```bash
cat ~/.kube/config
```

To test the configuration, display currently running services.

```bash
kubectl get svc
```

## Step 3. Create nodes under a managed node group.

There are different node types that can be used when creating a cluster.

For our setup, we decided to use **************************Managed Nodes************************** for Linux.

### IAM Role

Open the **Identity and Access Management (IAM) Console** again. Then, select ************Roles************ from the options under ************************************Access management************************************, as shown here on the left side of the panel.

![Untitled](/Node%20Setup/Method%204%20-%20Amazon%20EKS/images/Untitled%2015.png)

Click on **************************Create role.**************************

For **********Step 1**********, select ************************AWS service************************ as trusted entity type and ******EC2****** as the use case. Maintain default values for other settings.

![Untitled](/Node%20Setup/Method%204%20-%20Amazon%20EKS/images/Untitled%2016.png)

For ************Step 2************, add the following permissions to the node IAM role:

******************AmazonEKSWorkerNodePolicy******************

**************AmazonEKS_CNI_Policy**************

******************************AmazonEC2ContainerRegistryReadOnly******************************

![Untitled](/Node%20Setup/Method%204%20-%20Amazon%20EKS/images/Untitled%2017.png)

For ************Step 3************, add a role name and short description for the node IAM role. Keep the default values for other settings and then click on **********Create**********.

Role name: g2-eks-node

![Untitled](/Node%20Setup/Method%204%20-%20Amazon%20EKS/images/Untitled%2018.png)

Check if the node IAM role was successfully created by going to **********************IAM > Roles**********************.

![Untitled](/Node%20Setup/Method%204%20-%20Amazon%20EKS/images/Untitled%2019.png)

### Amazon EKS Console

Once the IAM role is successfully created, go to your previously created cluster.

Go to the **********************************************************************************************Amazon Elastic Kubernetes Service (EKS) Console**********************************************************************************************. Select ****************Clusters****************.

![Untitled](/Node%20Setup/Method%204%20-%20Amazon%20EKS/images/Untitled%2020.png)

Open the **************Compute************** tab and then click on ****************************Add node group****************************.

![Untitled](/Node%20Setup/Method%204%20-%20Amazon%20EKS/images/Untitled%2021.png)

For ************Step 1************, add a node group name. Also, select your recently created node IAM role.

Keep the other settings at their default values.

Node group name: ********************************g2-eks-nodegroup********************************

Node IAM role: **********************g2-eks-node**********************

![Untitled](/Node%20Setup/Method%204%20-%20Amazon%20EKS/images/Untitled%2022.png)

For **************Step 2**************, keep all default settings except for the ************************************************Node group scaling configuration.************************************************

![Untitled](/Node%20Setup/Method%204%20-%20Amazon%20EKS/images/Untitled%2023.png)

For ************Node group scaling configuration************, set the desired node size, minimum node size, and maximum node size as follows:

![Untitled](/Node%20Setup/Method%204%20-%20Amazon%20EKS/images/Untitled%2024.png)

For ************Step 3************, select all public subnets under the **********************academy-vpc**********************.

![Untitled](/Node%20Setup/Method%204%20-%20Amazon%20EKS/images/Untitled%2025.png)

For **************Step 4**************, review all configuration settings and then click on ************Create************.

![Untitled](/Node%20Setup/Method%204%20-%20Amazon%20EKS/images/Untitled%2026.png)

Go back to the ********Amazon EKS******** console. Select ******Clusters****** and click on your created cluster: ****g2.****

Wait for the node group status to change from ******************Creating****************** to ************Active************.

![Untitled](/Node%20Setup/Method%204%20-%20Amazon%20EKS/images/Untitled%2027.png)

## Step 4. View resources in your Kubernetes cluster.

There are two ways you can view the resources created in your Kubernetes cluster. You can either display these resources via the AWS Command Line Interface (CLI) or directly on the Amazon EKS console.

### AWS Command Line Interface (CLI)

Once the node group status turns **************Active**************, go to your AWS CLI. Verify if the node group was successfully created by displaying all currently running nodes.

![Untitled](/Node%20Setup/Method%204%20-%20Amazon%20EKS/images/Untitled%2028.png)

### Amazon EKS Console

In the left navigation pane, choose ****************Clusters****************. Click on your created cluster ****g2**** from the list.

Under the ****************Compute**************** tab, you’ll see the list of **Nodes** that were deployed for the cluster. You can also choose the name of a node to see more information about it.

Under the ******************Resources****************** tab, you’ll see all of the Kubernetes resources that are deployed by default to an Amazon EKS cluster. Select any resource type in the console to learn more about it.
