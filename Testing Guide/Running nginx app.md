# Running nginx app

For this section, we will run an application to simulate a real life scenario of HA testing while the application is running in one of your worker nodes.

1. Log in to AWS ECR to pull images from it.

```
aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin 431522258999.dkr.ecr.us-west-2.amazonaws.com
```

2. Create a namespace for your nginx app to deploy on.

```
kubectl create ns test
```

3. Create an nginx deployment inside the test namespace with 3 replicas using your custom image. 

```
kubectl create deploy nginx -n test --image=431522258999.dkr.ecr.us-west-2.amazonaws.com/marlon-academy-batch-4-app:fin --replicas=3 
```

4. Run a `kubectl get pods` to see the pods in creating status.

```
kubectl get pods -n test
```

![Screenshot 2022-12-09 at 10.30.05 AM.png](/Testing%20Guide/images/Screenshot_2022-12-09_at_10.30.05_AM.png)

![Screenshot 2022-12-09 at 10.30.33 AM.png](/Testing%20Guide/images/Screenshot_2022-12-09_at_10.30.33_AM.png)

5. Expose the deployment so it can be viewed in a browser.

```
kubectl expose deploy nginx -n test --port=80 --target-port=80 --type=LoadBalancer
```

6. Get the external ip by running the `kubectl get svc` command.

```
kubectl get svc -n test
```

![Screenshot 2022-12-09 at 10.43.28 AM.png](/Testing%20Guide/images/Screenshot_2022-12-09_at_10.43.28_AM.png)

7. (Optional) you can run a `dig` command to check if that dns is accessible to the internet and all the IP associated with it.

```
dig abddfc4bc67f94a8bbc5e29cf126c05d-586642328.us-west-2.elb.amazonaws.com
```

![Screenshot 2022-12-09 at 10.44.39 AM.png](/Testing%20Guide/images/Screenshot_2022-12-09_at_10.44.39_AM.png)

8. Copy that dns and paste it on the browser to access it.
    
    ![Screenshot 2022-12-09 at 10.46.41 AM.png](/Testing%20Guide/images/Screenshot_2022-12-09_at_10.46.41_AM.png)