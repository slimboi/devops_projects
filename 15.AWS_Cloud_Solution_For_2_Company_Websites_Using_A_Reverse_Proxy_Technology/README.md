# AWS CLOUD SOLUTION FOR 2 COMPANY WEBSITES USING A REVERSE PROXY TECHNOLOGY

In this project, we will build a secure infrastructure inside AWS VPC (Virtual Private Cloud) network for a fictitious company named `ofagbule` that uses WordPress CMS for its main business website, and a Tooling Website (https://github.com/slimboi/tooling) for their DevOps team. As part of the company’s desire for improved security and performance, a decision has been made to use a reverse proxy technology from NGINX to achieve this.

`Cost, Security, and Scalability are the major requirements for this project. Hence, implementing the architecture designed below, ensure that infrastructure for both websites, WordPress and Tooling, is resilient to Web Server’s failures, can accomodate to increased traffic and, at the same time, has reasonable cost.`

## Project Design Architecture Diagram

![architecture](./img/1.architecture.PNG)

## Starting Off AWS Project

1. Properly configure the AWS account and Organization Unit [Watch How To Do This Here](https://youtu.be/9PQYCc_20-Q)
   
- Create an AWS Master account. (Also known as Root Account)
- Within the Root account, create a sub-account and name it `DevOps`. (A seperate email address is needed to complete this)
- Within the Root account, create an AWS Organization Unit (OU). Name it Dev (Dev resources will be launched in here) and move the DevOps account into the Dev OU.
- Login to the newly created AWS account using the new email address.
- Create a free domain name for ofagbule company. (I used freenom)

- Create a hosted zone in AWS, and map it to your free domain from Freenom. [Watch how to do that here](https://youtu.be/IjcHp94Hq8A)

![hosted_zone](./img/2.hosted_zone.png)

**NOTE**: As you proceed with configuration, ensure that all resources are appropriately tagged, for example:

- Project: <ofagbule>
- Environment: <dev>
- Automated: <No> (If you create a recource using an automation tool, it would be <Yes>)

## Setting Up Infrastucture

1. Create a VPC

![vpc_created](./img/3.vpc_creation.png)

2. Create subnets as shown in the architecture

![subnets](./img/4.subnets_creation.png)

3. Create a route table and associate it with public subnets

![pub_rtb_assoc](./img/6.priv_rtb_assoc.png)

4. Create a route table and associate it with private subnets

![priv_rtb_assoc](./img/7.pub_rtb_assoc.png)

5. Create an Internet Gateway

![igw](./img/8.internet_gateway.png)

6. Edit a route in public route table, and associate it with the Internet Gateway. (This allows the public subnet to be accessible from the internet)
   
![](./img/9.pub_with_igw.png)

7. Create an Elastic IP

![](./img/7.nat_eip.png)

8. Create a Nat Gateway and assign one of the Elastic IPs (*The other 2 will be used by Bastion hosts)

9. Create a Security Group for:

- Nginx Servers: Access to Nginx should only be allowed from a `Application Load balancer (ALB)`. At this point, we have not created a load balancer, therefore we will update the rules later. For now, just create it and put some dummy records as a place holder.
  
- Bastion Servers: Access to the Bastion servers should be allowed only from workstations that need to SSH into the bastion servers. Hence, you can use your workstation public IP address. To get this information, simply go to your terminal and type curl [www.whatismyipaddress.com](https://www.whatismyipaddress.com/)
  
- Application Load Balancer: ALB will be available from the Internet
Webservers: Access to Webservers should only be allowed from the Nginx servers. Since we do not have the servers created yet, just put some dummy records as a place holder, we will update it later.

- Data Layer: Access to the Data layer, which is comprised of Amazon Relational Database Service (RDS) and Amazon Elastic File System (EFS) must be carefully desinged – only webservers should be able to connect to RDS, while Nginx and Webservers will have access to EFS Mountpoint.

![security_groups](./img/10.security_groups.png)

## Proceed With Compute Resources

You will need to set up and configure compute resources inside your VPC. The recources related to compute are:

- EC2 Instances
- Launch Templates
- Target Groups
- Autoscaling Groups
- TLS Certificates
- Application Load Balancers (ALB)
  
### TLS Certificates From Amazon Certificate Manager (ACM)
#

You will need TLS certificates to handle secured connectivity to your Application Load Balancers (ALB).

- Navigate to AWS ACM
- Request a public wildcard certificate for the domain name you registered in Freenom
- Use DNS to validate the domain name
- Tag the resource
- Bind the ACM to the route53 hosted zone created earlier

### Setup EFS
#
Amazon Elastic File System (Amazon EFS) provides a simple, scalable, fully managed elastic Network File System (NFS) for use with AWS Cloud services and on-premises resources. In this project, we will utulize EFS service and mount filesystems on both Nginx and Webservers to store data.

- Create an EFS filesystem
- Create an EFS mount target per AZ in the VPC, associate it with both subnets dedicated for data layer
- Associate the Security groups created earlier for data layer.
Create an EFS access point. (Give it a name and leave all other settings as default)

![nfs_create](./img/11.nfs_creation.png)

- On the EFS setup, create two access points for both `tooling` and `wordpress` applications

![access_points](./img/12.access_points.png)

## Setup RDS
#

**Pre-requisite:** Create a KMS key from Key Management Service (KMS) to be used to encrypt the database instance.

`Amazon Relational Database Service (Amazon RDS) is a managed distributed relational database service by Amazon Web Services. This web service running in the cloud is designed to simplify setup, operations, maintenance & scaling of relational databases. Without RDS, Database Administrators (DBA) have more work to do, but due to RDS, some DBAs have become jobless`

To ensure that your databases are highly available and also have failover support in case one availability zone fails, we will configure a multi-AZ set up of RDS MySQL database instance. In our case, since we are only using 2 AZs, we can only failover to one, but the same concept applies to 3 Availability Zones. We will not consider possible failure of the whole Region, but for this AWS also has a solution – this is a more advanced concept that will be discussed in following projects.

To configure RDS, follow steps below:

- Create a subnet group and add 2 private subnets (data Layer)
- Create an RDS Instance for mysql 8.*.*
- To satisfy our architectural diagram, you will need to select either Dev/Test or Production Sample Template. But to minimize AWS cost, you can select the Do not create a standby instance option under Availability & durability sample template (The production template will enable Multi-AZ deployment)
- Configure other settings accordingly (For test purposes, most of the default settings are good to go). In the real world, you will need to size the database appropriately. You will need to get some information about the usage. If it is a highly transactional database that grows at 10GB weekly, you must bear that in mind while configuring the initial storage allocation, storage autoscaling, and maximum storage threshold.
- Configure VPC and security (ensure the database is not available from the Internet)
- Configure backups and retention
- Encrypt the database using the KMS key created earlier

![kms_creation](./img/14.rds_kms.png)

- Enable CloudWatch monitoring and export Error and Slow Query logs (for production, also include Audit)

![database_creation](./img/13.db_creation.png)

## Creating AMIs for Launch Templates

- To create Launch templates and target groups later on, we will need to setup AMI containing configurations to be done on this respective servers.

