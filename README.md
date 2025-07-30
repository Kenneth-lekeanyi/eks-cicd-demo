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
18. Name:eks-cicd-demo
19. Image:CONTAINER-IMAGE
20. # IMAGE:VAMSICHUNDURU/app:v1.0.0
21. Ports
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
- So we have added our changes and our modified files to our local git. But we haven't commits it to a CodeCommit yet.
***If you see any warning here, just ignore it***
- Now we are done with the application changes in their local environment; it's now time to set up AWS CodeCommit credentials, repo and push the Code from local to AWS CodeCommit
- But we need to generate the credentials manualy to access CodeCommit.
- 
- # So let's start by first creating the IAM user CodeCommit git credentials.






So create IAM credentials to your IAM users, Which can be used to push the application code to AWS code commit
So go to your AWS console
-	go to IAM
-	then click on users on the left
in the search bar under users type and select your user Kenneth.(The intention here is that we need to push the application code to quote commit using this IAM user)
-	then click on “security credentials”
-	now click on “generate credentials” under HTTPS cut credential for AWS code commit
-	Click on “download credentials”
Note this credential is only generated once. So save it where you can remember. If you lost this credentials the only option is to click on it, delete it and create a new one. So whenever we are using a the AWS code commit we can just go and copy a code commit username and password.
Now we are done with credentials, now we are creating a code commit repository. As we are already done with our code in our local jit, we have to set these credentials so that we can push our code to our code commit. But we cannot directly push the code if we try other.
Remember to create all these in North virginia region.
The next step is to create a code commit Repo, Configure and push the application code to the court commit repository.
9)	So in your AWS console, Search for code commit and click on it
-	click on “create repository”
-	repository name: eks-cidc-demo
-	descripcion; repo to store EKS CIDC demo
-	Click now on “create”
(this gives us the ability to create a notification as well so that if a new coat drops in, it will notify us or the group. You can now see the details as it gives us ssh, HTTP and how to clone this repository.
10)	Now set up the copied URL as a remote repository for existing application code.
So add a git remote repository using this command
Git remote add origin https://git-codecommit.us-east-1.amazonaws.com/v1/repos/eks-cicd-demo
11)	Now check the status using the command Git remote-v
(You will see the newly created repository URL where you can now interact with the origin)
12)	now push the changes to the master bridge it codecommit.
Git push—set-upstream origin master. (This command means that we are interacting the master branch of our gits with the master branch of our code commit. When we press enter it demands us to put the 
Username; go and copy the username of your code commit and parts here And then you hit “enter”
Password; Go and copy the password and paste it here too using control+v
“Enter”
You will see it 
eventually object
counting object
compressing object
writing objects
Now it has copied the application code to the master of our code commit repository.( This is what developers often do)
•	now if you go back to our code commit repository and refresh, you will see our EKS- CICD demo Application code now present there.
•	We shall later see if you want to add a new change or modifications to it.
With this we have successfully pushed their code from local to remote repository of our code commit. The next step is to create IAM Role for the code pipeline, code build and setting up the CICD pipeline
In the diagram, you see the IAM role. Those IAM roles Will be used for code pipeline and code build For access management to other services. Now we have to create those IAM roles.
Note: As a DevOps engineer you will never touch the application. You only touch the build space which the app gets pushed to git, Commit and to the pipeline.
Section 5






