# Kubernetes CICD Demo with AWS CodeCommit, ECR, CodeBuild, CodePipeline & EKS.
<img width="1521" height="809" alt="Screenshot 2025-07-23 at 10 55 49 PM" src="https://github.com/user-attachments/assets/5c8885b0-5f0c-42ba-b5a4-c825d722ef2c" />

CloudFormation Stack. Copy the template from this url: https://s3.amazonaws.com/stelligent-public/cloudformation-templates/github/labs/codebuild/codebuild-cpl-cd-cc.json

# Steps to Install the required software and to set up the Pipeline.
1) Install required AWS CLI.
2) Set up EKS Cluster & Node groups.
3) Clone the code repo and perform manual deployments
4) Set up the codebase for CICD in the deployment file
5) Setup required AWS Services for the EKS CICD Pipeline
6) Setup required IAM Roles for the CICD Deployment.
7) Verify the Deployment.
8) Run the complete CICD flow for new deployments. (i.e run the complete cicd pipeline end-to-end)

**1) Section One: Install required CLI's**

**- (a) Setup IAM role, keypair and EC2 system for the management.**

**a) Create an EC2 AdministratorAccess role.** so,
- Go to the IAM Dashboard
- Then you click on 'Roles' at the left
- We don’t have to create administration access policy because as one knows, AWS will create such administration access policy by default. So now in order to create a new policy role;
  - Click on “create Role” at the top right. ***{This role that we are creating, the intention is to attach it to an EC2 instance. So go to use case down and select "EC2"}***.
  - Select or click the box on EC2. ***{Next time we shall create our role and we shall attach our role to codebuild. So we shall come to bar down and search for code build. Because we intend codebuild to access other services.}***
  - Trusted entity type is “AWS service”
  - Click on “next” ***{Like we said, AWS by default has administration access,}***
  - so under the box dealing with permission policies type “admin” and press enter. ***{If there was no default policy, we should have just typed in the box the type of policy we want.}***
  - “Administration access” will pop up right at the end down.
  - so click the box against it to check the box on  ***Administrator access***
  - click now on “next”
  - Role name: **eks-cicd-demo-ec2-role**. ***{So that the EC2 instance can interact with eks Clauster, EC2, codecommit etc. So this role is aimed to be attached to EC2, so that the EC2 can perform all the functions or actions}***.
  - Go down and click on “create rule” 
  - this rule has been created successfully.

**b)	The next thing is to create a key pair.**

To create a key pair, go to EC2 console, or EC2 dashboard and locate keypair on the left and click on it.
-	Then click on “create keypair”
-	name it: ***Kube-demo***
-	then click on “create keypair”.

**c) Next, launch an EC2 Linux instance using the role and key pair that we created above. While launching, add the "eks-cicd-demo-ec2 Role" that we have created above.**
- So go to instances on the left and click on it
-	click on “launch instances”
-	select "Amazon linux2 AMI"
-	Selected t2 micro(because we are not instally anything and also we want to minimize the cost)
-	click on “next”: configure instance details”
- VPC= ***default***
- Auto-assign public IP: ***Enable***
- Default subnet
-	IAM Role: ***select the "eks-cicd-demo-ec2-role" that we created***
-	Click on “next app storage”
-	next: “add tag”
-	next: "configure security group”
- ***{At work here when you take SSH, go under **source** and put only the **IP addresses of the Laptop**. so that u just you laptop to access this instance. So it will look like this:}***
**type**.        **Protocol**.     **port range**.       **Source**
ssh.               TCP.               22.                 Customs.  0.0.0.0/0
-	Review and launch
-	Launch
- choose an existing key pair
- then select our “kubee-demo” keypair
-	Launch instances
- name the instance as: **mgnt-eks-demo-ec2-instance**

**d) Now connect to the instance.** {You can use SSH to connect as well as you can use EC2 instance connect}.
- If you choose to use EC2 instance connect,
  - Click on "EC2 instance connect"
  - Then click on "connect" {this will directly connect with the shell to that EC2 instance Without necessary using terminal.}
-	First of all, update the instance. ***{For Ubuntu, Debian, do = sudo apt update. you can as well upgrade it by doing = sudo apt upgrade. for CentOs/RHEL/Fedora, do sudo yum update}***
- Then, Verify the version of AWS CLI. So do ***aws—version**. {You see that AWS CLI is already installed by default with version 1.18.147.} Python already installed as well. This is another reason why we choose Linux 2
-	Now, change from EC2 user to a root user. So do ***"sudo su"***

-	Now install Kubectl: Use this link to go to AWS documentation. https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html
- When the link opens, you will see macos/ linux/ windows. {Since we are dealing with Linux here, click on Linux}
  - this will show you the commands you will use to install Kubecth. Latest version on the OS (Linux) that we are currently using.
1.	Download the latest Kubectl 1.21 using the first command.
Now do **ls** to check if Kubecth is there.
3.	Let's now go to execution right: Go back to the documentation and Scroll down to #3 dealing with apply execute permissions to binary. Copy the command there and run it. ***{We do this because the Kubectl is a software folder And so we need to have permission to run or execute the software}***.
***{If you do pwd, you will realize that we are in the home path as EC2 user. So we must ensure that whatever commands we want to run, it should be executive not only in the home Directory, but anywhere}*** So go to #4
4.	Still from the documentation, copy the command on number on #4 dealing with **mkdir……..**
5.	Copy this command on #5 to export the path and run it
6.	Now, verify the Kubectl version by running this command on #6. ***{You will see the latest version of Kubecth. If there is no kubectl install this command ran, It would just throw out the command}***
- Now, We are done with the installation of Kubectl.
-
- **Let's now proceed to the installation of eksctl**. Use this link of AWS documentation to get the commands. https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html
When this link opens, you will still see **macos/linux/Windows**. Since we are dealing with Linux ensure to select Linux
1.	download the latest version of eksctl using this command on #1
2.	Move to the location or path of bin using the command here in #2
3.	Now check the eksctl version by runing this command on #3
***{when is says “command not found" and arrow out, clear your screen and stand by doing Sudo Su- enter}***
•	Now we are done with the installation of eksctl. 
Let's now proceed to install Git.

- Proceed to install Git.
***{Since git is already available or installed in the yum repository, We just have to run this command}***
- `Sudo yum install git -y`
- Now if we do or type `git` enter,
- You should be able to see some git commands
- with that we have installed all those softwares in our machine.




