Challenge #1 

A 3 tier environment is a common setup. Use a tool of your choosing/familiarity create these resources. Please remember we will not be judged on the outcome but more focusing on the approach, style and reproducibility. 

============================================================================ 

Overview 

This document describes on how to provision network and resources common setup for a new account in AWS. 

Follows AWS Resources  best practices such as: - 

Analysis 

Design  

Review the design Document  

Coding and Testing  

Finally, Jenkins Job run  

Verify the results: -  

  

3-Tier Architecture 

Appropriate Routing in Route Table etc. 

what are the resources needed (VPC/CIDR/Autoscaling/Security groups /S3/DynamoDB tables etc.) 

IAM Permitions  

Jenkins  

Terraform 

BitBucket/GIT/GitHub(separate reops for the resources line terraform Layer code and terraform Module code) 

  

  

for provisioning using IAC - terraform  

  

Pre-Requisites 

Need to create S3-Bucket and DynamoDB Table in "Central-iam" Account for saving terraform state file 

                     Note: we are following below naming convention for s3 bucket and DynamoDB table creation 

  

                                      S3 Bucket name: s3-<account_name>-terraform-state  

                                      DynamoDB table name:  <account_name>-terraform-state-lock 

                                       

If any shared service account using the following standard procedure. 

  

Jenkins user role "iam_role_jenkins_user" needs to have permissions to create "service-linked-role"(transitgateway.amazonaws.com) for Transit Gateway Attachment 

Create a BitBucket repo for Layer Code (<account>-terraform-network (exp:- terraform-network)), this is for specifying account specific variable and inputs for global network module 

If you are using any Shared-Services "SSH-Key" should be configured in BitBucket Repository Created above. This is required for Jenkins to clone the repo while running/executing the job Repositories 

  Steps: 

Create repositories for layer code 

Update terraform code for new account network provisioning 

Create Jenkins Job (1_<Account>_Network) in shared service Jenkins server 

Manual creation of S3-Bucket and DynamoDB Lock table to save terraform state in "<aacount-name>-<Dev/test/Prod>" to save terraform states 

Run Jenkins Job to provision VPC and its resources in Target Account. 

 

  Updating Layer Code: 

Copy the files for Layer code from any existing repo. 

Update Layer code files to reflect your account specific details. 

create softlink for "backend.tf" and "global_variables.tf" from common folder of layer code. 

command: "ln -s ../../common/global_variables.tf global_variables.tf" 

command: "ln -s ../../common/backend.tf backend.tf" 

Update "providers.tf" to reflect alias name for provider 

Sample:  

provider "aws" { 

alias = "<accountAlias>"  —> the same name should be used in "global_variables.tf" and "main.tf" files 

region = "${var.region}" 

allowed_account_ids = ["${lookup(var.target_account_id, terraform.workspace)}"] 


assume_role { 

role_arn = "arn:aws:iam::${lookup(var.target_account_id, terraform.workspace)}:role/uki_iam_role_jenkins_user" 

} 

} 

  
Modify "variables.tf" and update the values for below mentioned variable names: 


vpc_cidr : CIDR Range that will be allowed by IMSD Group during Account Creation. 

availability_zones: Based in requirement specify if you need 2 AZ or 3 AZ. This is also dependent on the region where Account is created. 

subnets: We follow 3-Tier Architecture, we will have to plan the subnet bifurcation based on CIDR range.  

subnet names: Please follow standard name, 

 privdmz_cidrs_routes: CIDR/IP Ranges that needs to be added in Private Route Table. 

common_tags: Update "Project Name" details. 
 

final Steps:- 

======================== 

Jenkins Configuration and Execution 

Get the new Jenkins Job created in Network View of Shared Services Jenkins server. 

Make sure appropriate git credentials are being configured on the same. 

Run Terraform Plan to check for errors. 

Run Terraform Apply Once Plan is successful. 

Once Terraform Apply is successful. Validate by login to the account and verify if the resources are provisioned properly. 

=========================== 

 

 

  
  
  
  
