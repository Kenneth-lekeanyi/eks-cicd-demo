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

# Section 2: Set up EKS cluster and not groups.

- There are a series of commands to run in order to set up the EKS cluster. All these commands are found in this Git Repo file. ***https://github.com/Kenneth-lekeanyi/eks-cicd-demo/blob/master/IAM%20%26%20Others/eks-cluster-nodes-setup.txt***
  
1) To create the EKS cluster without node group, Use this command. Copy the command as a block of code, and run it in your EC2 Deployer.
***To create the EKS cluster without nodegroup***
`eksctl create cluster --name=eks-cicd-dev-cluster \
                      --region=us-east-1 \
                      --zones=us-east-1a,us-east-1b \
                      --without-nodegroup`
- ***{If you now open CloudFormation, stack, you will see eksctl-eks-dev-cluster is already in the creation process. if you click on "Events", you will see all the resources that are now in the creation process. All these are happening in the backend. Click on the template to see the resources that are created}***
  
2) The next thing is to associate OIDC identity provider, this enables the flexibility to add IAM roles to EKS Cluster. Copy this command as a block of code and run it as well.
***Create & associate OIDC identity provider, this enables the flexibility to add IAM roles to EKS cluster***
`eksctl utils associate-iam-oidc-provider \
    --region us-east-1 \
    --cluster eks-cicd-dev-cluster \
    --approve`

- ***{As this EKS Cluster has to interact with a LBfor traffic distribution purposes, it will need an IAM Role}***

3) Now create the in the above created cluster through this command.
***To create the nodegroup in the above created cluster***
eksctl create nodegroup --cluster=eks-cicd-dev-cluster \
                        --region=us-east-1 \
                        --name=ng-workers \
                        --node-type=t2.medium \
                        --nodes-min=2 \
                        --nodes-max=3 \
                        --node-volume-size=10 \
                        --ssh-access \
                        --ssh-public-key=kube-demo \
                        --managed \
                        --asg-access \
                        --external-dns-access \
                        --full-ecr-access \
                        --appmesh-access \
                        --alb-ingress-access \
                        --node-private-networking

###### Display the EKS cluster to our user ######

You can verify the cluster node group in the CloudFormation Dashboard to see that its creation is complete.
CloudFormation that created the worker node can be verified in the CloudFormation console as well.
- Now both the **eks cluster** and the **worker node groups** are ready and running. The next step is to clone the existing application repo and deploy the application.

# Section 3: Cloned the Code Repository and perform manual Kubectl Deployement for understanding.
1)	Clone this code repo on this Repository: https://github.com/Kenneth-lekeanyi/eks-cicd-demo.git
- clone this repo above and take it to the EC2 instance in your terminal. i.e, copy the repor above and take it to your terminal.
This would basically just clone the demo or project.
- if you now do `ls` enter.
You will see the **bin eks-cicd-demo**

2) change the path to the repo folder. So do
`cd eks-cicd-demo`
3)	List existing report content. So do
`ls`
4)	Create or update the kubeconfig file for cluster if management server lost track of cluster. Use this command
`aws eks --region us-east-1 update-kubeconfig --name eks-cicd-dev-cluster` 
5)	Now apply the Kubectl manifests using this command 
`kubectl apply -f kube-manifests/` ***{LB WILL BE CREATED HERE}***
6)	Get the pods at this time. So do
`kubectl get pods`
7)	Get the service created as well, otherwise we can’t access it. So do
`kubectl get service`

- At this time, if you go to the load balancer and copy it domain name, then take it to a browser you will see that you will be able to access our Application.
•	Right now we have committed the code and ran the commands from codecommit to code build right to EKS.

- ***[Tomorrow we shall just push the code to Codecommit and nothing will be done at the level of Codebuild and EKS that we have done today. That process will be automated internally without us doing anything]***.

**Now let's quickly delete the manifest.**
-	Do `refresh`
-	then you do `sudo su -`
-	then be in the folder first by doing `cd eks-cicd-demo/`
-	Then do `kubctl delete -f kube-manifests/`
- Now services is deleted.
- 
With this the manual process of deployment is completed.
-
But since the Cluster and Nodes are still running and costly, we need to delete them. In order to delete them, run this command.
•	Delete node group using this command
`eksctl delete nodegroup --cluster=eks-dev-cluster --name=ng-workers --region=us-east-1`
•	Now we delete the EKS cluster as well,using this command
`eksctl delete cluster eks-cicd-dev-cluster --region=us-east-1`
- This will dismantle everything in the CloudFormation stack.
-
- ***{Note that, this is just a Manual Deployment. It is not used by most companies. This is just to show us how Deployment works}***
-
- Note also that, we have placed our EKS CLUSTER in a private subnet, reason why we cannot acces it. So inorder to access it, we only take the Domain name of the LB and use it in the browser to access the EKS Cluster.
-
- ***What we have done so far is that, we have clone the repository (assuming this repository) as that of a developer.
Now as a DevOps engineer, we have cloned that repository; now, after cloning it into EC2, We are using the EC2 as our management EC2.(Because we are using it to connect all the services. Because some of these have macos and others have windows, linux etc. We decide to use Amazon EC2. Then in this EC2, we have installed AWS CLI and we have also installed Kubectl And also we have installed eksctl  and lastly we have installed git.
- So using all these CLIs and softwares, We have executive commands such as `kubectl` commands, eksctl commands
- And we have created clusters, we have created nodegroups and then we have deployed pods ontop of those node groups. Which means that we have deploy this node groups on top of eks clusters.
- In order to do all these, we have executed eksctl commands to create a cluster, and then we have used IAM to integrate the cluster although we shall later still use IAM role again.
- Then we have also created node groups using commands in live 13-26
- wherein cluster will be harboring the node groups and whenever we launch containers, those containers will be launched inside node groups.
- So we have launched an application by using Kubectl (that is Kubectl apply -f kube manifest/) Manifest file.
- If you see the manifest file in the application code, you will see that repicas is 2 meaning it has created 2 pods
- And what exactly do we have inside this pot? Inside this pods, we have a Docker image. See line 20 of the deployement file inside the manifest file which is a docter image that we have created called Vamsichunduru app v.1.0.0. And as we - have mentioned that we have two replicas, it means we have another vamsichunduru app v.1.0.0 Inside another and/ or container in node group 2.
- Then we have mentioned that we have created a service. This service means an ELB, So that whatever any user is accessing the URL, he will be able to catch up the URL that will hit the LB and the LB will then distribute traffic to those applications within the node Group 1 or node group 2.
- In our environment this LB will  just show up as a DNS (Domain name) search as amazon.com, facebook.com ETC. So that the user don't know them anything like ELB. All they know and see is the domain name like Netflix, or amazon.com or facebook.
- This is how traffic is distributed.
- This is what we have done so far as the first part.
- 
- # We are now going to create the CodeCommit, To store the Application Code: we shall also create the CodeBuild  and CodePipeline. Wherein, we shall also make use of IAM to access order services.
- 
# Section 4: Set up the codebase for the CICD
- (Note that everything has to be done together we cannot keep our pipeline in a broken state.)

- Connect back to our EC2 instance "mgnt-eks-demo Instance" and do `sudo su -` ***inorder to be a real super sudo user.*** 
- If you remember we have done the repository **eks-cicd-demo**. So if you do `ls` enter, You should be able to see the repository deployed as **bin eks-cicd-demo**

- Currently our Application code is in Github. And we need to push it to CodeCommit so that ColdPipeline will integrate it to CodeBuild and everything gets going to deployement at the EKS Cluster.
- So first change your path to eks-cicd-demo. so do
- `cd eks-cicd-demo/` enter
- Now do `ls` to see all the files there.
- 
1)	Now do `git remote -v` enter
- You will see the address of this repository appear as
- https://github.com/Kenneth-lekeanyi/eks-cicd-demo.git (push). This is the URL that we had cloned earlier.
-	Now we want to remove this code in GitHub so that in future we shall use it in CodeCommit.
2)	Now remove the existing git remote using this command. so do
`git remote rm origin`
- if you now do `git remote -v` enter, you will see that nothing exists there anymore. This is because the repo has been removed.
- Note that in the manifest file, our container image “vamsichunduru is "had-coded" so that nothing should ever change there. ***(But this is not the best practice because as we move the Application Code from GitHub to CodeBuild a new docker image will be generated to go to EKS. So we should be deploying V1, V2, V3, V4 etc. So whenever we are deploying the docker image, we can be sure that we are employing the latest version of that image; That was pushed by a developer out of EKS)***. 
- -----Dev-----CodeBuild------docker-----EKS_V2.
- So in order to do that, We shall uncommend the container image as seen in line 19 of the manifest file.
- So in order to uncomment that we have to first of all be in the eks-cicd-demo. But since we are already in this directory, If you do `ls` you will see the files there amongst which we see kube-manifests file.
3)	Now change the path to Kube-manifests. So do
- `cd kube-manifests/`
- So from EKS-CICD-demo, we are now in "Kube-manifests". 
- Now when you do `ls`, you should see "app-deployment.yml" and "app.nodeport-service.yml"
- Now our intention is to edit app "deployment.yml"
- 4)	So do `cat app-deployment.yml`
- You will see the file here have container image commented. Now You understand that, this is the local repository that we are using.
- 5)	Now inorder to edit the file, do
`nano app-deployment.yml`
- Now move down and comment the line dealing with image: vamsichundunu then you uncommend the line above dealing with image: container-image. {As you know this is a yml file so make sure your identity is perfect so let it appear as follows.
Containers:
  - Name:eks-cicd-demo
  - Image:CONTAINER-IMAGE
  - # IMAGE:VAMSICHUNDURU/app:v1.0.0
  - Ports
- Then exit using command +x (or control +x) to same the file
- Save modified buffer? Press Y then hit “enter”.
- Now if you do `cat app-deployment.yml` again you will see the items you have commentted and that which you have uncommitted.
6)	Now do `cd ..` to go back to eks-cidc-demo
- Then do `git status` to check the status of git you will see it say that manifest file has been modified.
7)	Add all the changed files to git by doing
- `git add.` ***{This will add the modified files to Kube manifest/app-deployment.yml)***
8)	Now commit the change to local git by doing
- `git commit -m “modified the container image variable”`
9) Do `git status` you will see that "nothing to commit and making tree clean".
-
- {So we have added our changes and our modified files to our local git. But we dont have to do **git commit** now inorder to commit it to CodeCommit yet. Because we have to set it up first, create a CodeCommit User, because we need it credentials inorder to access CodeCommit, and generated the CodeCommit URL before we can use it to push the Application source code from gitHub repo to our CodCommit.}
- 
***If you see any warning here, just ignore it***
- Now we are done with the application changes in their local environment; it's now time to set up AWS CodeCommit credentials repo and push the Code from local to AWS CodeCommit
- But we need to generate the credentials manualy to access CodeCommit.
- 
- # So let's start by first creating the IAM user CodeCommit git credentials.
-
- So create IAM credentials to your IAM users, Which can be used to push the application code to AWS CodeCommit
- So go to your AWS console
-	go to IAM
-	then click on "users" on the left
- in the search bar under users type and select your user Kenneth.(The intention here is that we need to push the application code to CodeCommit using this IAM user). ***(Make sure that the user has "AdministratorAccess", otherwise, it will fail.
-	then click on “security credentials”
-	now click on “generate credentials” under HTTPS credential for AWS CodeCommit. (If you want to avoid being asked each time you want to push a change to CodeCommit and it keeps asking for **username** and **Password**, rather than clicking on "Generate Credentials under https", instead click on **"Upload SSH Public Key"**, and just download the public key into your computer, so that whenever you want to push any changes to CodeCommit, it will just use the public key to push those changes to codeCommit without asking you for any Username and Password)
-	Click on “download credentials”. ***{Note that, this credential is only generated once. So save it where you can remember. If you lost this credentials the only option is to click on it, delete it and create a new one. So whenever we are using AWS CodeCommit, we can just go and copy a CodeCommit username and password.}***
- Now we are done with credentials, now we are creating a CodeCommit Repository. As we are already done with our code in our local Git, we have to set these credentials so that we can push our code to our CodeCommit. But we cannot directly push the code. Because Codecommit is not yet linked to our local git repo.
- ***{Remember to create all these in North virginia region}***.
The next step is to create a CodeCommit Repository, Configure and push the Application Code to the CodeCommit Repository.

- So,
- In your AWS console, Search for Codecommit and click on it
-	click on “create repository”
-	repository name: **eks-cidc-demo**
-	descripcion; **repository to store EKS CIDC demo Application code**
-	Click now on “create”
- (this gives us the ability to create a notification as well, so that if a new code drops in, it will notify us or the group. You can now see the details as it gives us ssh, HTTP and how to clone this repository.

- Now, copy the URL as a remote repository for our existing application code.
- So, in our Terminal, add a git remote repository using this command
- `git remote add origin https://git-codecommit.us-east-1.amazonaws.com/v1/repos/eks-cicd-demo` enter
- Now check the status using the command `git remote-v`
- (You will see the newly created repository URL where you can now interact with the origin)
- Now push the changes to the master branch of codecommit. So do
- `git push --set-upstream origin master`. ***(This command means that we are interacting the master branch of our git with the master branch of our CodeCommit)***. When we hit enter it demands us to put the 
- Username; go and copy the username of your CodeCommit and paste it here And then you hit “enter”
- Password; Go and copy the password and paste it here too. (Note, Password will not show) “Enter”
-
- Challenge Here: ***{when you do `git push` to get the code from the local git repo to the master branch of CodeCommit Repository, it keeps reporting 403 error}***
- To fix this, make sure that the user that is habouring the codecommit have Administrative Access. So if you have created that user without an Administrator Access, go and delete it and create a new user with Administrative Access.

- You will see it saying:
  - enumerating objects
  - counting objects
  - compressing objects
  - writing objects
- Now it has copied the application code to the master Branch of our CodeCommit Repository.
•	now if you go back to our CodeCommit Repository and refresh, you will see our EKS- CICD demo Application code now present there.
•	We shall later see if you want to add a new change or modifications to it.
- With this we have successfully pushed our Application Source Code from local to a remote repository of our CodeCommit.
-
-The next step is to create IAM Role for the CodePipeline, CodeBuild and setting up the CICD pipeline.

- In the diagram, you see the IAM role. Those IAM roles Will be used by CodePipeline and CodeBuild For access management to other services.
- Now we have to create those IAM roles.
- (Note: As a DevOps engineer you will never touch the Application Source Code. You will only touch the "Build-spec" which the app gets pushed to git and to CodeCommit, and to the Pipeline.

# Section 5: set up required IAM roles for the CICD deployments.

- As the application code or code base is pushed to CodeCommit Repository, the next steps is to set up the IAM roles for the pipeline and to create required AWS service for (ELB) for the pipeline. ***(So, this Role will be attached to CodeBuild and CodePipeline.)***
- 
- 1) ***Set up policy for EKS cluster interactions.***. So,
- Create an IAM policy with the name: **iam-eks-describe-policy** By following the steps below
- (if you already deleted the cluster and node groups, then use the first command to set up the cluster and node groups here. So that there will be up and running by the time we finish the next session). ***{ To create the cluster directly using Commands, run the first command on line 1. inside ~eks-cluster-nodes-setup.txt~. When it gets created, run the second command on line 7. to attach or associate IAM Role. Then, when it gets done, creates the worker nodegroups using the 3rd command on line 14-29.}***
- 
- so open your IAM console
  - go to IAM
  - click on "policies"
  - click on “create policy”
  - then click on "Jason"
  - just copy the policy in ***(iam-eks-describe-policy.json)*** in this link and paste it in the Jason editor. https://github.com/Kenneth-lekeanyi/eks-cicd-demo/blob/master/IAM%20%26%20Others/iam-eks-describe-policy.json
  - 
  - When you copy it, go to Jason editor page, select everything in there and first delete it.
  - Then paste the policy you just copied from the link above in there.
  - click on “next : tags”
  - Next review
  - Name: **iam-eks-describe-policy** {Please do not change the name. copy it as it is because down the line, we shall use this name}
  - Descriptions: policy to describe EKS cluster
  - click now on "create policy”

- **now we are done with the policy: the next task is to create a role and attach this policy to the role. Follow these steps to create a custom role with the name as eks-cicd-kubectl-role**. So go to IAM role and
  - click on "Roles"
  - click on “create role”
  - click on “custom trust policy”
  - Now copy the policy that you will find in this link below inside ***(custom-trust-policy.json)*** https://github.com/Kenneth-lekeanyi/eks-cicd-demo/blob/master/IAM%20%26%20Others/custom-trust-policy.json
  - When you copy it, go to the custom trust policy editor, delete whatever you find there
  - then paste the new policy that you just copied here.
  - ***{We are going to attach this role to the EKS cluster node. So, On line 7 we see a <AWS account ID>. so, replace this with your AWS account ID. Copy the ID at the top right of your account beside your name and paste it here to appear as follows. “AWS:”am:aws:iam:;746210165048:root”(Be careful with the spaces) }***
  - then click on “next”
  - now it will prompt us to select a policy that we have earlier created.
  - So search in the permissions policy search bar for ***Iam-eks-describe-policy***
  - Select this ***iam-eks-describe-policy***
  - Click on “next”
  - Role name: ***eks-cicd-kubectl-role***
  - Descripcionption: ***EKS Kubectl role for interacting with EKS cluster***
  - Scroll down to click on “create role”
-So, as the first step we have created an IAM policy. Then we have created a role and attach this policy to the Role that we created.

- Again, we still need to create one more policy and one more role for the code build.
- 
2)	***Set up the policy for AWS CodeBuild Role.***
- Just click on this link below to copy the big policy inside ***(eks-cicd-codebuild-poilicy.json)*** https://github.com/Kenneth-lekeanyi/eks-cicd-demo/blob/master/IAM%20%26%20Others/eks-cicd-codebuild-policy.json
- Copy this policy in this link and paste in any text editor.
- Replace the AWS_Accounts_ID with your own ID number. There are three places that you need to replace this account ID. So,
- (copy your ID on the top right and replace it in those three places)
- ***{so as you can see this policy will make CodeBuild to interact with services that you see in this code as s3. Cloudmatch logs,.. When you finish replacing your account ID in those three places.{Make sure that there are no spaces between your ID and the semi colon}.***
- Now, Copy the whole code
- Go back to your IAM console
- click on "policies"
- click on "create policy"
-click on "json"
* Delete whatever you see existing in there
* Paste the new long policy in here
* Click now on "next stage"
-	click on "next review”
- name: **eks-cicd-codebuild-policy**
- Description: **policy for codebuild project of eks cicd demo**
-	Scroll down to click on “create policy”.
-	
3)	***Now set up the final role for ColdBuild***
- create a new role with the name **eks-cicd-codebuild-role** by Following the below steps and use the policy that we have created above. So,
- go to IAM console
-	now as we are done with policy creation, click now on "Roles"
-	click on "create role”
-	Select "AWS service"
-	on the box on [choose a service to view use case]. In this box, type in this **[CodeBuild]**
-	click the circle against CodeBuilds to check that box on CodeBuild
-	Then, click on “next”
-	permission policies
- Select in here the [eks-cicd-codebuild-policy] That we just created above
- Click to check the box on [Eks-cicd-codebuild-policy]
- Then click on “next” 
- Role name: **eks-cicd-codebuild-role**
- Click now on “create role”  ***{since we want to attach this role to codebuild, so that CodeBuild will be able to perform some services, we would attach this role to CodeBuild}***
- 
- So, what we have done is that, we have created 2 policies and 2 Roles. Wherein, one is for the EKS Cluster and the other one is for the CodeBuild. Before proceeding to the next step, make sure that both your EKS Cluster and Worker nodes are up and running.
- 
4)	***Configure the AWS-auth configmap with eks-cicd-kubect-role created earlier.***
  - (This means that, here we are integrating AWS IAM Which we have created above with Kubenetes cluster.)
  - ( Remember; we had earlier on created IAM Roles associated with OIDC Identity provider. This enables flexibility to add our new IAM Roles to EKS Cluster.
  i) Now, lets execute this command so that we can intergrate the new Roles that we have created to EKS Cluster.
  - 
  - Before that verify the existing AWS-auth configuration for the cluster, through this command.
Kubectl get configmap aws-auth-o yaml-n kube-system. (This command with diploid the config role that kubecth currently has. But our intention is to add another rule so that it will be used to deployments
b)	Now configure the AWS-uath configmap using the commands in line 34 from this link inside (eks-cluster-nodes-setup.txt). https://github.com/Kenneth-lekeanyi/eks-cicd-demo/blob/master/IAM%20%26%20Others/eks-cluster-nodes-setup.txt
- Here are the commands in this link: https://github.com/Kenneth-lekeanyi/eks-cicd-demo/blob/master/IAM%20%26%20Others/aws-auth%20config.txt
-  # Verify what is present in aws-auth configmap before change. So run `kubectl get configmap aws-auth -o yaml -n kube-system`
- {This command will display the config role that kubectl currently has. But our intention is to add another Role so that it will be used for deployments}.
- Now, export your Account ID through this command `export ACCOUNT_ID=464599248654`
- {Copy this command and put it in a text editor. Then replace this id with your Account ID}. Then copy the command and run it in your EC2 terminal.}
- Now let's execute the second command which is to set role value.
- # Set ROLE value
- `ROLE="    - rolearn: arn:aws:iam::$ACCOUNT_ID:role/eks-cicd-kubectl-role\n      username: build\n      groups:\n        - system:masters"`
- {Replace this 'Account_ID' with your own AWS account ID}.
- {This command is simply saying that "go into my account where you will see the eks-cicd kubectl and attach that role}.
- At this time, echo role to see the value. Use this command echo and role
- # echo Role to see the value
- `echo $ROLE`
- Now get the current AWS auth configmap data and attach new role info to it. Use this command.
- # Get current aws-auth configMap data and attach new role info to it
- `kubectl get -n kube-system configmap/aws-auth -o yaml | awk "/mapRoles: \|/{print;print \"$ROLE\";next}1" > /tmp/aws-auth-patch.yml`
- Now, patch or update the AWS-auth configmap with a new role , Using this command
- # Patch the aws-auth configmap with new role
- `kubectl patch configmap/aws-auth -n kube-system --patch "$(cat /tmp/aws-auth-patch.yml)"`
- Finally verify what is updated in this AWS-auth config map After change. Do this using this command
- # Verify what is updated in aws-auth configmap after change
- `kubectl get configmap aws-auth -o yaml -n kube-system`
- You will see 
  - rolearm:
  - username: build
  - groups:
  - When you see this, know you are done.
- With that, we have set up all the permissions and Roles required for the demo. It's now time for us to create AWS Devops services for the cicd Demo.
- { “Note that we are using EKS but the software is kubenetes. And this kubenetes is interacting with AWS and it is creating resources like Load Balanacer."}
- { "This is a reason why we earlier attached a custom rule to our kubenetes. So we use some third party software “AWS-config-map to attach to kubenetes so that it can perform some actions."

- # Section 6: Set up required AWS services for the EKS CICD demo
- ***i) Create an ECR repo to store the container images.***
- During the CICD pipeline execution, Pipeline will automatically create and push a Docker image every time to the ECR Repo.
- Now let's go to ECR console and create this repository. So
  - So on the search bar of your AWS account, type and search for ECR (Elastic container registory)
  - Click on it
  - Now, click on “create repository”
  - visibility settings; click to check the box on "Private".
  - Name: **eks-cicd-demo**
  - scroll down to click on “create repository”
- Now we are done with the creation of our ECR repository as well as done with IAM Roles as well. Now let's proceed to create CodePipeline and CodeBuild for the pipeline.
- 
- ***ii) Create CodePipeline and CodeBuilt for the pipeline.***
- So,
- Go to your AWS console  and use the search bar to type and search for CodePipeline.
  - Click on "CodePipeline"
  - click on “create pipeline”
  - Pipeline name: **eks-cicd-demo**
  - Select to check the box of "new services role"
  - Role name: **AWSCodePipelineServiceRole-us-east-1-eks-cicd-demo**
  - Click to check this box on [**allow AWS CodePipeline to create a service role so it can be used with this new pipeline**].
  - Then click on “advanced settings”
  - Under Artifact store; Check these boxes on [Default location] and on [encrypt AWS managed key]
  - Click on “next”
  - Source provider; in the box, use the drop down to search for [AWS CodeCommit].
  - Repository name: "**eks-cicd-demo**"
  - Branch name: "Master"
  - Now, check the boxes on [Amazon Cloudmatch Events (recommended)] and [Codepipeline default]
  - Click on “next”
  - Build: ***{This is where our build happens}***
  - In the first box, type in [AWS CodeBuild] ***{Here, this is where if you want to intergrate Jenkins, you can do it here. You can use the drop down to select "jenkins" as well as other tools that you might want to use. But for our case, we are using AWS CodeBuilt. Note that the Jenkins file must be set up and running some where. So you use this just to integrate it}***
  - Region: **Us East (N.VIRGINIA)**
  - Project name:
  - Click on “create project” on the right
  - Project name: **eks-cicd-demo-dev**  {A new portal will pop up for you to use and create a new project}
  - 
  - ***{So, we put dev with CodeBuild because we want to get the Application Code to go to the EKS-cicd-Dev Environment. If we want the code to go to another Acc or Prod, we can put it to CodeBuild as well. And if we want it to go only to Prod, we can add Prod to CodeBuild. And we can also come and create a "Manual Approval" stage after the CodeBuild so that such manual Approval will be done before the code moves to Prod}***
  - Description: **Codebuild to build eks cicd demo and deploy on Dev cluster**.
- Environment:
  - Click and check this box on "managed image"
-	Operating system:
  - Amazon linux2 {So AWS will use Amazon Linux for our CodeBuild internally}.
-	Runtime:
  - Standard
-	Image:
  - Use the drop down in the box to select [aws/codebuild/amazonlinux2-x86_64-standard3.0] {**inshort, select the latest version, which is always down at the bottom**}
-	Image version:
  - In the box, let it display like this [Always use the latest image for this runtime version]
-	Environment type:
  - Select [Linux]
-	Priviledge:
  - Check the box that says [Enable this flag if you want to build docker images..]
-	Service role:
  - check the box to select [“existing service role”]
-	Role ARN:
  - use thr drop down to select the role that we have earlier created. it will now appear as [Arn:aws:iam……….role/eks-cicd-codebuild-role]
-	Now, check the box on [Allow AWS codebuild to modify this service role so it can]
- Now, Click on “additional configuration”
- Leave all these boxes as they are
-	Certificate:
- **Do not install any certificate**
-	VPC: ***{For a demo we are not selecting any VPC. so leave this box blank. But at work make sure you select a VPC that is private and not open to the public. And thus that VPC must have a NATGATWAY Since we are not passing through the Internet. So if you select a VPC here, know that your codebuild will be placed in that VPC that you selected here.}***
-	Compute:
  - Check the box on [3GB memory,2 VCPUS]



-	Environment variables
use this link to copy your environment variables
https://github.com/cvamsikrishna11/eks-cicd-deo/blob/master/IAM%20%26%20others/code-build%20env%20variables.txt
Name
EKS-KUBECTL-ROLE-ARN
EKS-CLUSTER-NAME
Value
Arn:aws:iam:746210165048:role/eks…
Eks-cicd-dev-cluster
(first copy the arm {arn:aws:iam: :…………..role} take it to a text editor then replace the ID that you see there with your account ID. Then you then copy the edited arm and paste it in the box under value.
Type
-plarintext
Plarintext
Repository-URI.   copy the URL of your ECR and paste here.     Plain text
-	Click on “add environment variable
Build space
Use a buildspec file
Build name-optional
Buildspec.yml
•	Click on “continue to codepipeline”
Leave the remaining sections as it is and go to code pipelines console back. This will save and close the existing code build pop and redirect to the code pipeline page.
So on the code pipeline console, you will see a message with a small tick saying that successfully created eks-cicd-demo-dev in codebuild
•	If you want to add another environment variable you can do it here.
-	Click on next
-	click on skip deploy stage
-	your pipeline will not include a diploment stage. Are you sure you want to skip this stage?
-	Click on skip
The review page will be visible to verify the settings And click on “create pipeline”
this will start the initial build. 
Then is the code pipeline console, As soon as you click on create pipeline on that other page, yeah is the code pipeline console it will report that “SUCCESS”
Congratulations the pipeline eks-cicd-demo has been created.
You will see that it pulled the code from the source. Then once it clone the code from the scm it automatically start the build. So-code pipe will integrate all these stages.
On the console if you go to the left and click on “build project” you will see the name of the pipeline and its ID beside it.
•	If you click on the details under the build, click on open in a new tab, it will open in a new tab where you can see the pipeline name and it's ID. And if you Scroll down you will see all the details of what is happening. (e.g You will see where it is trying to push the image to the repository) etc
-	You will see that it is “success”
meaning that the build has been succeeded or successful
-	click on “phase details” to see how they build succeeded and more details
-	click again on “build details” to see more details on the logs.
Now we need to verify that we have completed the build. And in order to verify that,
•	go back to your terminal or EC2 instance there and do
-	Sudo su
-	Cd into the folder: cd eks-cicd-demo
-	Then run this command to get the pod. Kubectl get pods Kubectl get pods.
It will show you how the deployment is running as such:
Name.                                                      ready.          Status.   Restart.      Age
Eks-cicd-demo-deployment         1/1.            Running.    0.                4min
Eks-cicd-demo-deployment.         1/1.            Running.     0.             4min
-	Run this command to get Kubernetes service
Kubetes get service
You will see the service that has been created such as the load balancer that has been created.
-	If you go too EC2 and go to the load balancer, you will see the load balancer that has been created
-	now you can see that the scheme of it says ……Internet facing……..
That is why if we copy the DNS of this load balancer (don”t copy (A record) And take it to the browser and enter it you will see our demo application has now been diployed and it is up and running without executing any comments.
This is how we deploye an EKS setup.
Now as you can see the UI of the EKS we shall change the version and some few things And we shall see how it will deploy.
And we shall see how build is doing this thing and how it is getting this thing automated. Let's see how it happens
as we know how to edit site, go to your EC2 terminal again
-	Do ls. To see buildspec.yml,dockerfile,IAM and others, kubemanifests, READ.md
-	Now do cd app
-	Now do ls again
-	Now do cat index.html to see the html file
-	Now do nano index.html to edit it.
Add a line below contact us.html<p>hey,we have dona a great job!!!!!!! <\p>
Now do command +x
Press y
Click now on “enter”
-	If you now do cat index.html, you will see the change we have made.
-	Now do git status
You will see it says index.html file has been modified.
-	Now do git add.
-	Now do git commit-m deployment a new version v1.2.0”
Now again do git status again. It says(use git push tompublish your local commits) This means that we have done everything in our local git and therefore have Nothing to commit. Waking the clean.
And for we should use git push to push the code to code commit
-	now do git push
it will prompt you to enter the username and password (go to the page where you saved your code commit credentials copy the username and password and bring them here)
username:…………..enter
password……………..enter
•	So it has pasted the version to code commit
•	Before going to code commit, go to code pipeline and see how this has interacted.
You will see under source  down after refreshing it says source: deploying a new version v1.2.0
-	Succeeded-just now
2520d88 (Click here on this hyperlink you will see the change and how it has moved from the existing version to a new version)
•	go back to code pipeline. Under build, click on details to see the logs
•	Go to our Amazon container service (AC5) or (ECR) console, we shall see that a new image has been created.
•	If we now go to the browser page where we pasted the DNS of the load balancer and entered to see “hi there”
And we refresh this page, we shall now see what we added to appear as
“hey we have done a great job!!!! With a version -1.2.0
so we haven't done anything. Basically what we have just done is that we just did the application code change and push it. Everything is taken care of until it gets to ekso
          Q $ A
What we have just done is possible in two ways. First you diploy your code or application code to code commit. Then code build and code pipeline will do some stuffs internally And there will store images such as v1, v2, v3, in the ECR.
And after deploying version 3 which is the latest version to EKS, Let's assume that this version gets broken and everything is not going well, you have to revert to version v2.
There are 2 steps to do it or ways to do it.
1)	As a DevOps engineer, you can revert to the last code change ask Git or in your local git and do what is known as git revert. If you do this, it will revert and use all code changes (i.e v2+ v3) And get it to code build. These are the level of build will come up with a new code change called version v4 (because it is a new code change). And this V4 will be deployed into the EKS cluster. Where as this V4 will have the changes of V2.
(This is initiated from the console or git, where we would be prompted to provide the username and password for us to get the old v2 from our local gIt to code commit. And when we provide the credentials everything is then taken care of and the changes will appear in EKS cluster as V4.)
2)	Another way is that we can create another or separate code build or cold pipeline project. When in search code build is not interacting with code commit. But it is only interacting with ECR where we have this version V1,V2,V3.
So this code build will take manual input from us the DevOps engineer (where we can use environment variables to get it there).
However before starting the manual input, you have to provide the version image e.g V2 that we want to introduce, we have to provide it and get it in as our input environment parameter.
Once this is done, It has to take the image in build space. Since we already have the docker image present in cold build, what is done is that it will directly take the docker image and it would diploy the docker image on top of EKS. This is the faster way as we already have an image of V3 that has broken in the code build so by introducing version 2, it just take it at the code build and deploy it to EKS cluster






To implement this method, go to the code pipeline console, click on “build projects” on the left.
-	 Select the project that you want to edit (In our case we are editing eks-cicd-demo-dev)
O eks-cicd-demo-dev AWS Codepipeline…..
-	Click on edit
then select “environment”
then click on “add environment variable”
DOCKER_IMAGE_VERSION.        V2.       Plaintext
-	Then click on “update environment”
-	then go to the top right and click on “release charged”
however before doing that you must have the new version that is V2. If you don't have it, then those parameters that you passed in the environment variable is of no use. See how to get the new version at hand!!! And see how to input it in environment variable.
Whenever there is a break in the code, all the team is put to work to find out where the problem is. The infrastructure team will check to see
-	if the network is OK
-	if the storage is OK
-	if the pods are OK
(by reading the locks you will understand where the problem is coming from. That is why monitoring is very important)
Note that: EKS or Kubectl is managed by AWS. Yes. Bear we have to run all the commands to set up the cluster,, commands to set up the node group, Commands to set up fargate as well if applicable.














