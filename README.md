#### This folder contains terraform configuration files to create various resources on aws to deploy multiple  ecs services that are necessary.

#### We need an additional file tf.tfvars which is a variables file to run these scripts. tf.tfvars file is not present in this repository. It needs to have values w.r.t client's needs.
#### To create tf.tfvars file please refer to the variables.tf file present in the root directory in this folder. Also given sample values of tf.tfvars file in this  document later. 

#### Commands to deploy infrastructure
###### To initiate modules and  installs plugins for required providers
`terraform init` 
###### To show any syntax errors
`terraform validate` 
###### To shows what resources are going to be created/destroyed/modified in the cloud 
`terraform plan -var-file="tf.tfvars"` 
###### To create resources in the cloud depending upon the input values provided in the tf.tfvars file
`terraform apply -var-file="tf.tfvars"`  
###### To destroy the resources if no longer required.
`terraform destroy -var-file="tf.tfvars"` 


## Prerequisites for executing the commands

* Make sures `s3_bucket_name` used here for storing env file must have been created before applying 

* Right now ECR has to be setup manually and the image corresponding to the service has to be uploaded to ECR from local for the first time. So you can provide image urls in the input variables to reduce some manual steps if it is already created. Else you can use some dummy name for now and can change it manually later.

* If you have the `environment files` related to services ready with you, then you can keep the files with valid names in the env directory at root level in this folder then those files will be uploaded to s3.(Name of envfile that code expects is defined within the code and you can refer to any of the ecs modules to understand the naming convention.) Please note that if file-name is not exactly mentioned as it requires then it will upload a dummy env file to the s3 bucket which is in the root module but inside s3 you can see a file uploaded with proper name which you need to replace it manually later on. (The name of env file to be uploaded depends on other variables: For e.g. if game_name="abc", environment="dev", service_name[i]="contest" then corresponding env should be having the name: abc-dev-contest.env )

* ensure that `ecsrole` mentioned has proper permissions to read env files. Service may fail to update if it does not have enough permission to read envs stored at s3.

* Sample tf.tfvars file should have valid values and mentioned below the required sample values. Values mentioned here are for reference only and not be used as it is while running in the scripts.


---
***

##### ***Overall application related variables***

---

game_name          = "test-client"

service_name       = ["contest", "cricket", "football", "leaderboard", "payment", "player-analytics", "tenant", "socket", "revenue", "game-analytics"]

services_to_launch = [true,        false,      false,         true,    false,       true,           false,    false,     false,      false]

environment        = "dev"

aws_region           = "us-east-1"

---

##### ***vpc related variables***

---

vpc_cidr_block       = "192.168.0.0/16"

subnets_cidr_public  = ["192.168.0.0/24", "192.168.1.0/24", "192.168.2.0/24"]

subnets_cidr_private = ["192.168.3.0/24", "192.168.4.0/24", "192.168.5.0/24"]

---

##### ***load-balancer,ecs and env-file related variables***

---

contest_image      = "nginx:latest"

cricket_image      = "nginx"

football_image     = "nginx"

payment_image      = "nginx"

pl-analytics_image = "nginx"

leaderboard_image  = "606880623734.dkr.ecr.ap-south-1.amazonaws.com/leaderboard-service:latest"

tenant_image       = "606880623734.dkr.ecr.ap-south-1.amazonaws.com/tenant:latest"

target_port        = 3000

certarn            = "arn:aws:acm:us-east-1:606880623734:certificate/a99a9aad-d80c-4d0e-bafc-97192b94a3b9"

ecsrole            = "arn:aws:iam::606880623734:role/ecsTaskExecutionRole-b2b-tenant"

dns_name           = "api-dev.gamio-services.net"

s3_bucket_name     = "test-fargate-cluster-configurations"

---


---
***



### Resources that will be created ?
##### We create long list of resources. Broadly VPC related, Loadbalaner related, cloudwatch related, s3 related(upload envs only) and ecs related services.
---

* VPC related!

    ##### - Creates VPC, three public and three private subnets.
    ##### - Creates Route tables for public and private subnets. Associates them to corresponding subnets
    ##### - Creates internet gateway 
    ##### - Creates NAT Gateway
    ##### - Creates EIP for NAT Gateway
---

* Loadbalancer Related

##### - Creates loadbalancer
##### - Creates HTTP and HTTPS listeners
##### - Create listener rules for  whatever service that we are creating for client mentioned in the input files. In some of the services more than one listener rule is created for forwarding to target groups
---

* Cloudwatch related

##### - Creates cloudwatch group for each service that we want to launch
---
* s3 related

##### - we upload the env files to the already created s3 bucket which is present in `ap-south-1` region
---

* ecs related

##### - Creates ecs cluster for the client.
##### - Creates resources for only the services mentioned in the input file.
##### - Creates task definition for the services mentioned
##### - Targetgroup for the service for the services mentioned
##### - Creates ecs service for the services mentioned
---
