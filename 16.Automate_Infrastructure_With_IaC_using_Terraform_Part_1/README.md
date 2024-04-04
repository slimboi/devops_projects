# Automating Infrastructure Creation With IAC (Terraform)


- Create an IAM user, name it terraform (ensure that the user has only programatic access to your AWS account) and grant this user AdministratorAccess permissions.

- Copy the secret access key and access key ID. Save them in a notepad temporarily.

If you have the AWS CLI installed, then you can use the aws configure command to configure your credentials file:

```
aws configure
```

# VPC | SUBNETS | SECURITY GROUPS

Let us create a directory structure

Open your Visual Studio Code and:

- Create a folder called `PBL`
- Create a file in the folder, name it `main.tf`

# Provider and VPC resource section

- Set up Terraform CLI.

- Add `AWS` as a provider, and a resource to create a VPC in the `main.tf` file.

- Provider block informs Terraform that we intend to build infrastructure within AWS.

- Resource block will create a VPC.

```
provider "aws" {
  region = "eu-west-2"
}

# Create VPC
resource "aws_vpc" "main" {
  cidr_block                     = "172.16.0.0/16"
  enable_dns_support             = "true"
  enable_dns_hostnames           = "true"
}
```

- The next thing we need to do, is to download necessary plugins for Terraform to work. These plugins are used by `providers` and `provisioners`. At this stage, we only have `provider` in our `main.tf` file. So, Terraform will just download plugin for AWS provider.

- Run the below command
```
terraform init
```

![inititialize terraform](./img/1.terraform_init.png)

- Notice that a new directory has been created: .`terraform\...`. This is where Terraform keeps plugins. Generally, it is safe to delete this folder. It just means that you must execute terraform init again, to download them.
- Moving on, let us create the only resource we just defined `aws_vpc`. But before we do that, we should check to see what terraform intends to create before we tell it to go ahead and create it.

- Run `terraform plan`
- Then, if you are happy with changes planned, execute `terraform apply`

![vpc_creation](./img/2.vpc_creation.png)

1. A new file is created `terraform.tfstate`. This is how Terraform keeps itself up to date with the exact state of the infrastructure. It reads this file to know what already exists, what should be added, or destroyed based on the entire terraform code that is being developed.
2. If you also observed closely, you would realise that another file gets created during planning and apply. But this file gets deleted immediately `terraform.tfstate.lock.info` .This is what Terraform uses to track, who is running its code against the infrastructure at any point in time. This is very important for teams working on the same Terraform repository at the same time. The lock prevents a user from executing Terraform configuration against the same infrastructure when another user is doing the same - it allows to avoid duplicates and conflicts.

## Subnets resource section
#

According to our architectural design, we require 6 subnets:

- 2 public
- 2 private for webservers
- 2 private for data layer
  
Let us create the first 2 public subnets.

Add below configuration to the main.tf file:

```
# Create public subnet1
    resource "aws_subnet" "public1" {
    vpc_id                     = aws_vpc.main.id
    cidr_block                 = "172.16.0.0/24"
    map_public_ip_on_launch    = true
    availability_zone          = "eu-west-2a"

}

# Create public subnet2
    resource "aws_subnet" "public2" {
    vpc_id                     = aws_vpc.main.id
    cidr_block                 = "172.16.1.0/24"
    map_public_ip_on_launch    = true
    availability_zone          = "eu-west-2b"
}
```

- We are creating 2 subnets, therefore declaring 2 resource blocks - one for each of the subnets.
- We are using the `vpc_id` argument to interpolate the value of the VPC id by setting it to aws_vpc.main.id. This way, Terraform knows inside which VPC to create the subnet.

Run `terraform plan` and `terraform apply`.

**Observations:**

- `Hard coded values`: Remember our best practice hint from the beginning? Both the `availability_zone` and `cidr_block` arguments are hard coded. We should always endeavour to make our work dynamic.
- `Multiple Resource Blocks`: Notice that we have declared multiple resource blocks for each subnet in the code. This is bad coding practice. We need to create a single resource block that can dynamically create resources without specifying multiple blocks. Imagine if we wanted to create 10 subnets, our code would look very clumsy. So, we need to optimize this by introducing a `count argument`.

Now let us improve our code by refactoring it.

First, destroy the current infrastructure. Since we are still in development, this is totally fine. Otherwise, `DO NOT DESTROY` an infrastructure that has been deployed to production.

To destroy whatever has been created run `terraform destroy` command, and type `yes` after evaluating the plan.

