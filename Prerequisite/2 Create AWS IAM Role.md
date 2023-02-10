# 2. Create AWS IAM Role

**Glossary:**

IAM Role

- An IAM role is **an IAM identity that you can create in your account that has specific permissions.** An IAM role is similar to an IAM user, in that it is an AWS identity with permission policies that determine what the identity can and cannot do in AWS.

Before we create an EC2 instance, let’s first create an IAM Role.

On your AWS Console go to:

IAM > Roles > Create Role

**Select trusted entity**

Trust entity type: **AWS service**

Common use cases: **EC2**

![Untitled](/Prerequisite/images/2%20Create%20AWS%20IAM%20Role%20-%201.png)

Click **Next** to proceed…

Add permissions

![Untitled](/Prerequisite/images/2%20Create%20AWS%20IAM%20Role%20-%202.png)

![Untitled](/Prerequisite/images/2%20Create%20AWS%20IAM%20Role%20-%203.png)

![Untitled](/Prerequisite/images/2%20Create%20AWS%20IAM%20Role%20-%204.png)

[Back to Setting Up Repo](/Prerequisite/1%20Setting%20Up%20Repo.md)
[Next: Create AWS EC2 Instance](/Prerequisite/3%20Create%20AWS%20EC2%20Instance.md)

[Back to Main Directory](/ReadMe.md)