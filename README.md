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
***- (a) Setup IAM role, keypair and EC2 system for the management. So,***
- i) Create an EC2 AdministratorAccess role. so,
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
  - Role name: ~eks-cicd-demo-ec2-role. ***{So that the EC2 instance can interact with eks Clauster, EC2, codecommit etc}***.
-	Go down and click on “create rule” 
this rule has been created successfully

