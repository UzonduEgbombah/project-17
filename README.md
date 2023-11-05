# project-17
#### AUTOMATE INFRASTRUCTURE WITH IAC USING TERRAFORM. PART 2
#### Networking
Private subnets & best practices
Create 4 private subnets keeping in mind following principles:

- Make sure you use variables or length() function to determine the number of AZs
- Use variables and cidrsubnet() function to allocate vpc_cidr for subnets
- Keep variables and resources in separate files for better code structure and readability
- Tags all the resources you have created so far. Explore how to use format() and count functions to automatically tag subnets with its respective number.
A little bit more about Tagging
Tagging is a straightforward, but a very powerful concept that helps you manage your resources much more efficiently:

- Resources are much better organized in ‘virtual’ groups
- They can be easily filtered and searched from console or programmatically
- Billing team can easily generate reports and determine how much each part of infrastructure costs how much (by department, by type, by environment, etc.)
- You can easily determine resources that are not being used and take actions accordingly
- If there are different teams in the organisation using the same account, tagging can help differentiate who owns which resources.
#### Note:
You can add multiple tags as a default set. for example, in out terraform.tfvars file we can have default tags defined.

![](https://github.com/UzonduEgbombah/project-17/assets/137091610/f9d09641-6f37-4ce6-b132-3ad41d511690)

Now you can tag all you resources using the format below
![](https://github.com/UzonduEgbombah/project-17/assets/137091610/9e6ee761-d39b-43d8-807f-00470636ed30)

The nice thing about this is – anytime we need to make a change to the tags, we simply do that in one single place (terraform.tfvars).

But, our key-value pairs are hard coded. So, go ahead and work out a fix for that. Simply create variables for each value and use var.variable_name as the value to each of the keys.
Apply the same best practices for all other resources you will create further.

#### Internet Gateways & format() function
Create an Internet Gateway in a separate Terraform file internet_gateway.tf

![](https://github.com/UzonduEgbombah/project-17/assets/137091610/1f4b195a-88fa-4092-94da-1f8413c0343d)

Did you notice how we have used format() function to dynamically generate a unique name for this resource? The first part of the %s takes the interpolated value of aws_vpc.main.id while the second %s appends a literal string IG and finally an exclamation mark is added in the end.

If any of the resources being created is either using the count function, or creating multiple resources using a loop, then a key-value pair that needs to be unique must be handled differently.

For example, each of our subnets should have a unique name in the tag section. Without the format() function, we would not be able to see uniqueness. With the format function, each private subnet’s tag will look like this.

- Name = PrvateSubnet-0
- Name = PrvateSubnet-1
- Name = PrvateSubnet-2
Lets try and see that in action.

#### NAT Gateways
Create 1 NAT Gateways and 1 Elastic IP (EIP) addresses

Now use similar approach to create the NAT Gateways in a new file called natgateway.tf.

#### Note: 
We need to create an Elastic IP for the NAT Gateway, and you can see the use of depends_on to indicate that the Internet Gateway resource must be available before this should be created. Although Terraform does a good job to manage dependencies, but in some cases, it is good to be explicit.

![](https://github.com/UzonduEgbombah/project-17/assets/137091610/1c6940e8-e5ee-43fb-9989-3181a14d3489)

#### AWS ROUTES
Create a file called route_tables.tf and use it to create routes for both public and private subnets, create the below resources. Ensure they are properly tagged.

- aws_route_table
- aws_route
- aws_route_table_association

![](https://github.com/UzonduEgbombah/project-17/assets/137091610/433b6edd-aa18-4282-a79c-43659b87012b)

![](https://github.com/UzonduEgbombah/project-17/assets/137091610/6e42fae4-b589-46a6-a609-718c271ebf09)

Now if you run terraform plan and terraform apply it will add the following resources to AWS in multi-az set up:

![](https://github.com/UzonduEgbombah/project-17/assets/137091610/ff669833-026a-42b7-a78f-56b238f0569d)

![](https://github.com/UzonduEgbombah/project-17/assets/137091610/b2c1d2f4-f5ce-46e7-a6bd-db20386c6768)

– Our main vpc
– 2 Public subnets
– 4 Private subnets
– 1 Internet Gateway
– 1 NAT Gateway
– 1 EIP
– 2 Route tables
Now, we are done with Networking part of AWS set up, let us move on to Compute and Access Control configuration automation using Terraform!

#### AWS Identity and Access Management
#### IaM and Roles
We want to pass an IAM role our EC2 instances to give them access to some specific resources, so we need to do the following:

#### Create AssumeRole
Assume Role uses Security Token Service (STS) API that returns a set of temporary security credentials that you can use to access AWS resources that you might not normally have access to. These temporary credentials consist of an access key ID, a secret access key, and a security token. Typically, you use AssumeRole within your account or for cross-account access.

#### Add the following code to a new file named roles.tf

In this code we are creating AssumeRole with AssumeRole policy. It grants to an entity, in our case it is an EC2, permissions to assume the role.

#### Create IAM policy for this role
This is where we need to define a required policy (i.e., permissions) according to our requirements. For example, allowing an IAM role to perform action describe applied to EC2 instances:

#### Attach the Policy to the IAM Role
This is where, we will be attaching the policy which we created above, to the role we created in the first step.

#### Create an Instance Profile and interpolate the IAM Role
JPEG

![](https://github.com/UzonduEgbombah/project-17/assets/137091610/2060faf3-495a-463b-b033-0d4ee5d7ff05)

We are pretty much done with Identity and Management part for now, let us move on and create other resources required.

Resources to be created
As per our architecture we need to do the following:

- Create Security Groups
- Create Target Group for Nginx, WordPress and Tooling
- Create certificate from AWS certificate manager
- Create an External Application Load Balancer and Internal Application Load Balancer.
- create launch template for Bastion, Tooling, Nginx and WordPress
- Create an Auto Scaling Group (ASG) for Bastion, Tooling, Nginx and WordPress
- Create Elastic Filesystem
- Create Relational Database (RDS)
Let us create some Terraform configuration code to accomplish these tasks.

## CREATE SECURITY GROUPS
We are going to create all the security groups in a single file, then we are going to refrence this security group within each resources that needs it.

#### IMPORTANT:

Check out the terraform documentation for security group

Check out the terraform documentation for security group rule

Create a file and name it security.tf, copy and paste the code below

![](https://github.com/UzonduEgbombah/project-17/assets/137091610/80a51f10-121a-400b-835d-6ebf823e6f1b)

#### IMPORTANT NOTE:
We used the aws_security_group_rule to refrence another security group in a security group.

#### CREATE CERTIFICATE FROM AMAZON CERTIFICATE MANAGER
Create cert.tf file and add the following code snippets to it.

NOTE: Read Through to change the domain name to your own domain name and every other name that needs to be changed.

Check out the terraform documentation for AWS Certifivate mangarer

![](https://github.com/UzonduEgbombah/project-17/assets/137091610/ec21538f-e06d-45d8-b58c-1dbb415d37ee)


#### Create an external (Internet facing) Application Load Balancer (ALB)
Create a file called alb.tf

First of all we will create the ALB, then we create the target group and lastly we will create the lsitener rule.

Useful Terraform Documentation, go through this documentation and understand the arguement needed for each resources:

ALB
ALB-target
ALB-listener
We need to create an ALB to balance the traffic between the Instances:
- To inform our ALB to where route the traffic we need to create a Target Group to point to its targets:
- Then we will need to create a Listner for this target Group

Add the following outputs to output.tf to print them on screen

- Create an Internal (Internal) Application Load Balancer (ALB)
- For the Internal Load balancer we will follow the same concepts with the external load balancer.

![](https://github.com/UzonduEgbombah/project-17/assets/137091610/68391bb1-fca8-4163-8f93-165e9210acfb)

#### CREATING AUTOSCALING GROUPS
This Section we will create the Auto Scaling Group (ASG)
Now we need to configure our ASG to be able to scale the EC2s out and in depending on the application traffic.

Before we start configuring an ASG, we need to create the launch template and the the AMI needed. For now we are going to use a random AMI from AWS, then in project 19, we will use Packerto create our ami.

Based on our Architetcture we need for Auto Scaling Groups for bastion, nginx, wordpress and tooling, so we will create two files; asg-bastion-nginx.tf will contain Launch Template and Austoscaling froup for Bastion and Nginx, then asg-wordpress-tooling.tf will contain Launch Template and Austoscaling group for wordpress and tooling.

Useful Terraform Documentation, go through this documentation and understand the arguement needed for each resources:

- SNS-topic
- SNS-notification
- Austoscaling
- Launch-template

#### Create asg-bastion-nginx.tf and paste all the code snippet below;

![](https://github.com/UzonduEgbombah/project-17/assets/137091610/88a39049-2873-45fd-b6e5-5fbb0074c3e8)


#### Autoscaling for wordpress and tooling will be created in a seperate file

Create asg-wordpress-tooling.tf and paste the following code

![](https://github.com/UzonduEgbombah/project-17/assets/137091610/d7a3983f-5955-497b-90d6-03edf209243f)

#### STORAGE AND DATABASE
Useful Terraform Documentation, go through this documentation and understand the arguement needed for each resources:

- RDS
- EFS
- KMS
#### Create Elastic File System (EFS)
In order to create an EFS you need to create a KMS key.

AWS Key Management Service (KMS) makes it easy for you to create and manage cryptographic keys and control their use across a wide range of AWS services and in your applications.

Add the following code to efs.tf

![](https://github.com/UzonduEgbombah/project-17/assets/137091610/d7ce4190-d279-42d1-9ec4-6059fc0f143f)


#### Create MySQL RDS
Let us create the RDS itself using this snippet of code in rds.tf file:

![](https://github.com/UzonduEgbombah/project-17/assets/137091610/9a7598e6-fa04-4afb-b70d-2bdf42197d4f)

Before Applying, if you take note, we gave refrence to a lot of varibales in our resources that has not been declared in the variables.tf file. Go through the entire code and spot this variables and declare them in the variables.tf file.

If you have done that well, you file should like this one below.

![](https://github.com/UzonduEgbombah/project-17/assets/137091610/b0dc13ef-33a4-4f18-a0ba-34a948a9420a)


Now, we are almost done but we need to update the last file which is terraform.tfvars file. In this file we are going to declare the values for the variables in our varibales.tf file.

Open the terraform.tfvars file and add the code below

![](https://github.com/UzonduEgbombah/project-17/assets/137091610/af5443c3-b99b-4499-9a75-70ca9fb8d3da)

At this point, you shall have pretty much all infrastructure elements ready to be deployed automatically, but before we paln and apply our code we need to take note of two things;

we have a long list of files which may looks confusing but that is not bad for a start, we are going to fix this using the concepts of modules in Project 18
Secondly, our application wont work becuase in out shell script that was passed into the launch some endpoints like the RDs and EFS point is needed in which they have not been created yet. So in project 19 we will use our Ansible knowledge to fix this.
Try to plan and apply your Terraform codes, explore the resources in AWS console and make sure you destroy them right away to avoid massive costs.




## GRAPHIC USER INTERFACE OF SOME OF THE TERRAFORM (IAC)

![](https://github.com/UzonduEgbombah/project-17/assets/137091610/4dd73130-d15e-481b-b01d-ed789fc70a94)

![](https://github.com/UzonduEgbombah/project-17/assets/137091610/18b39a50-603c-4580-a92f-79cb71d74ba5)

![](https://github.com/UzonduEgbombah/project-17/assets/137091610/a0484f4e-bf2d-4b50-97d7-f67239a15d82)

![](https://github.com/UzonduEgbombah/project-17/assets/137091610/420ac999-94b3-4acd-aec9-ed7909d1a623)

![](https://github.com/UzonduEgbombah/project-17/assets/137091610/5942121b-e0a4-419b-90f3-cf4ea9850d34)

![](https://github.com/UzonduEgbombah/project-17/assets/137091610/e6caf8a9-77b6-45ab-9ce6-2f8ee3efdaea)

![](https://github.com/UzonduEgbombah/project-17/assets/137091610/2ad1f927-11c9-455c-92f2-3279b07f12df)

![](https://github.com/UzonduEgbombah/project-17/assets/137091610/4acf9286-3c5e-4e51-be33-63c35588629a)

![](https://github.com/UzonduEgbombah/project-17/assets/137091610/d93373ee-4865-4d2e-9ce6-6dd065b0306d)

![](https://github.com/UzonduEgbombah/project-17/assets/137091610/2f096810-ad71-4020-8b84-9055ad010a7f)

![](https://github.com/UzonduEgbombah/project-17/assets/137091610/f9e576d6-f4d2-4763-a2c4-7c56b3d84a06)

![](https://github.com/UzonduEgbombah/project-17/assets/137091610/a4adcad2-ca0b-4a0b-a23e-c6a92a62038d)

![](https://github.com/UzonduEgbombah/project-17/assets/137091610/0b63f7b7-642a-4788-b1ec-094c9893bd24)

![](https://github.com/UzonduEgbombah/project-17/assets/137091610/0c4a447a-9fb1-494f-85ff-62d693f6b335)

![](https://github.com/UzonduEgbombah/project-17/assets/137091610/7c7c2ccf-793d-42fe-8d98-8400e1aaebc3)






