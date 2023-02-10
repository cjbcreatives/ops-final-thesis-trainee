# 4. Pushing images to ECR Repo -  k8s components and app

You can push your container images to an Amazon ECR repository with the **docker push** command. Amazon ECR also supports creating and pushing Docker manifest lists, which are used for multi-architecture images. Each image referenced in a manifest list must already be pushed to your repository. For more information, see [Pushing a multi-architecture image](https://docs.aws.amazon.com/AmazonECR/latest/userguide/docker-push-multi-architecture-image.html).

### **To push a Docker image to an Amazon ECR repository**

Before you push the image, the Amazon ECR repository must already be there. See Creating a private repository for further details.

1. Ensure that your Docker client is authorized to access the Amazon ECR registry where you want to push your image. For each registry utilized, authentication tokens that are good for 12 hours are required. See Private registry authentication for further details. Use the aws ecr get-login-password command to authenticate Docker to an Amazon ECR registry. Use AWS as the username and specify the Amazon ECR registry URI you wish to authenticate to when delivering the authentication token to the docker login command. You must issue the command again for each register if you are authenticating to more than one registry.

```
aws ecr get-login-password --region region | docker login --username AWS --password-stdin aws_account_id.dkr.ecr.region.amazonaws.com
```

1. If your image repository doesn't exist in the registry you intend to push to yet, create it. For more information, see [Creating a private repository](https://docs.aws.amazon.com/AmazonECR/latest/userguide/repository-create.html).

1. Identify the local image to push. Run the **docker images** command to list the container images on your system.

```
docker images
```

1. Pull the image using the **docker pull** command. The image name format should be *`registry*/*repository*[:*tag*]` to pull by tag, or *`registry*/*repository*[@*digest*]` to pull by digest.

```
docker pull aws_account_id.dkr.ecr.us-west-2.amazonaws.com/amazonlinux:latest
```

1. Tag your image with the Amazon ECR registry, repository, and optional image tag name combination to use. The registry format is *`aws_account_id*.dkr.ecr.*region*.amazonaws.com`. The repository name should match the repository that you created for your image. If you omit the image tag, we assume that the tag is `latest`.
    
    The following example tags a local image with the ID *`e9ae3c220b23`* as *`aws_account_id*.dkr.ecr.*region*.amazonaws.com/my-repository:tag`.
    
    ```
    docker tag e9ae3c220b23aws_account_id.dkr.ecr.region.amazonaws.com/my-repository:tag
    ```
    
2. Push the image using the **docker push** command:
    
    ```
    docker push aws_account_id.dkr.ecr.region.amazonaws.com/my-repository:tag
    ```
    
3. Lastly check your ECR to assure you have successfully pushed the image.

![Screenshot 2022-12-09 at 9.54.16 AM.png](/Prerequisite/images/4.png)