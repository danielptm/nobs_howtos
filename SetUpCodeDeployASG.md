##Set up codedeploy/ASG to work together.
*How to setup code deploy as well as an ASG so that code deploy and the ASG
can work together to deploy an application in a CI/CD way.*

####Prep if needed:
- Delete ASG.
- Delete code deploy app.
- Delete code deploy IAM service role


**************************
####Integrating CodeDeploy with Amazon EC2 Auto Scaling
1. Create IAM profile that allows the ec2 auto scaling group to work with s3. Preferrably full access to be safe.

2. Create an ASG that specifies the IAM instance profile. Presumably from created in step 1

3. Create a code deploy service role that allows it to create a deployment group that contains the ec2 ASG.

4. Create a deployment group with codedeploy that specifies the ASG and service role.

5. use code deploy to deploy revision to the deployment group that contains the ec2 ASG.

**************************
####How to create ec2 auto scaling group:
[Check out docs](https://docs.aws.amazon.com/codedeploy/latest/userguide/tutorials-auto-scaling-group-create-auto-scaling-group.html)

1. Go here [amazon ec2 console](https://docs.aws.amazon.com/codedeploy/latest/userguide/tutorials-auto-scaling-group-create-auto-scaling-group.html)

2. In nav bar under Auto scaling choose launch configurations

3. Create launch configuration

4. Choose ubuntu

5. On Choose Instance type page leave blanks click Next: Configure details.

6. set name as globati-code-deploy-AS-config. In IAM role choose the profile created earlier.

*The AMI has to have a code deploy agent installed on it so a bash script has to be copied to User data see the website but here it is. Expand Advanced Details, and in User data, type the following.*

```
#!/bin/bash
yum -y update
yum install -y ruby
cd /home/ec2-user
curl -O https://aws-codedeploy-eu-central-1.s3.amazonaws.com/latest/install
chmod +x ./install
./install auto
```

*Notice: Bucket name is replaced already after my research.*

7. On Review choose Create launch configuration.

8. Select choose existing key pair for the one already created.

9. Choose Create an Auto Scaling group using this launch configuration.

10. On Configure Auto scaling group details page:
	Group name: globati-code-deploy-AS-config
	Set desired group size:
	In network choose launch into ec2 classic and choose default vpc
	Choose subnet for availability zone default
	Select NEXT: Configure scaling policies

11. On Configure scaling policies choose next configure notifications. Configure to receive email.

12. Skip the step for configuring notifications, and choose Review.

13. Choose Create Auto Scaling group, and then choose Close.

14. In the navigation bar, with Auto Scaling Groups selected, choose CodeDeployDemo-AS-Group, and then choose the Instances tab. Do not proceed until the value of InService appears in the Lifecycle column and the value of Healthy appears in the Health Status column.

**************************
####Deploy the application:
[More docs](https://docs.aws.amazon.com/codedeploy/latest/userguide/tutorials-auto-scaling-group-create-deployment.html)

1. Get service role arn for the code deploy service role.
2. Sign in here [code deploy](https://console.aws.amazon.com/codedeploy)
3. Expand deploy > applications
4. Choose create application
5. Choose custom application
6. In application name write globati-backend-deploy
7. Compute platform > EC2/on-premises
8. Create application
9. On deployment groups > Create deployment group
10. In deployment group name enter GlobatiDg
11. In service role choose name of service role
12. Deployment type > in place
13. Amazon EC2 auto ASG > CodeDeployDemo-AS-Group
14. In deployment configuration choose codedeploydefault.OneAtATime
15. Clear enable load balancing.
16. Create deployment group
17. in deployment group page > create deployment
18. In Revision type > My application is stored in Amazon s3
19. Revision location > enter OS and region (See above website for value #F > For Amazon Linux and RHEL Amazon EC2 instances)
20. Leave deployment description blank
21. expand advanced
22. Create deployment.

**************************
####How to create an appspec file:
1. Go [here](https://docs.aws.amazon.com/codedeploy/latest/userguide/application-revisions-appspec-file.html#add-appspec-file-server) to get a an appspec template and copy into ```appspec.yml``` file. Also [here](https://docs.aws.amazon.com/codedeploy/latest/userguide/reference-appspec-file.html) is more docs.

2. Place ```appspec.yml``` in root of application next to ```buildspec.yml```





**************************
##Issues
I had to add tags to the ASG/deployment groups. Despite them being optional they
were necessary to get the application to the server. This caught me up for almost 2 days.
