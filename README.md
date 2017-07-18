##### WORK IN PROGRESS
### Kong - API-Gateway on Weaveworks-ECS AMI
---  


###### -- Kong an open source API Gateway that runs in front of RESTful APIs.
###### -- Built on top of battle tested Nginx webserver
###### -- Scalable - Easy horizontal scaling
###### -- Modular - Can be extended via plugins
###### -- Minimal requirements - Can run on Cloud platforms, Containers, On-premise etc
###### -- Backed by Cassandra or Postgres

<br />

#### Weave with ECS
---

![alt text](https://raw.githubusercontent.com/faizan82/ECS-kong/master/images/weave-on-ecs.png)

<br />



##### This repository
- Terraform modules and code to deploy a highly available Kong cluster in ECS
- Ansible Integration to demonstrate concepts for deploying Kong and Cassandra services
- A python utility that manages deployement on ECS rather than relying on Ansible's ECS module.
  -  Lack of ability to deploy a service across multiple cluster led to a custom utility
- Also demonstrate deploy and destroy time provisioners in Terraform
- Demostrate use of WeaveWorks networking in deploying the cluster and service discovery in ECS
- Demonstrate use of overlay network for ECS
- Demonstrate use of Cloudwatch-logs. A log group and stream is setup for log forwarding and aws logging driver is used.
- Demonstrate cloud-init with Terraform 




##### Pre-requisites
- AWS account. Obviously!
- Terraform > 0.9.5
- Ansible >= 2.3
- Python 2.7
- Boto, Botocore
- Registerred Domain and Route53 Hosted zone ( Good to have else you have to change code )
- Related ACM certificates for your domain ( Good to have else you have to change code )

<br />

#### Deployment architecture 
---
![alt text](https://raw.githubusercontent.com/faizan82/ECS-kong/master/images/kong-architecture.png)


<br />

### Deployment 
---
#### What is deployed?
1. VPC - Three private subnets and three public subnets
2. Two ECS Clusters ( Kong and Cassandra in public and private subnets respectively) 
3. A bastion node.
4. AWS Log group and log stream 
5. EBS Volumes of 50G attached to each Cassandra node using cloud-init 
6. Route53 entries based on choosen domain names and details provided to Terraform 


#### Deployment procedure
---
1. Ensure pre-requisites are met 
2. Decide a region where this needs to be deployed 
3. This guides a cluster in a region with 3 AZs. You can reduce the number in terraform.tfvars file 
2. Generate your ACM Certificates for the domain 

```shell
# Prepare your environment ( Terraform and Ansible )
# Change directory to terraform code/environments
# We are considering a sample development environment
# setup a secrets.tf file
terraform plan -var-file=secrets.tf
```


##### Note
- This is beyond the scope of Free-tier since we use two ELBs and t2.medium instances. You can reduce t2.medium to micro and replace ELBs with your own loadbalancers but you will have to update the code
- Kong limitation: A single node needs to be deployed first and then can be scaled due to limitation with https://github.com/Mashape/kong/issues/2139
- Cassandra : There is no concept of master - slave. However, for clustering it expects a pre-existing Seed Node. In order to achieve this, a Seed Node is deployed and then clients are added. Based on the choosen replication factor, there can be data replicated across all nodes
