## Terraform learnings 
tutorials from here: https://learn.hashicorp.com/collections/terraform/aws-get-started

Checkout 'fundamentals': https://learn.hashicorp.com/terraform 

might be some other good ones here: https://learn.hashicorp.com/collections/terraform/certification-associate-tutorials

Contents: 
- [What is infrastrucutre as code with Terraform?](#what-is-infrastrucutre-as-code-with-terraform)
- [Install terraform](#install-terraform)
- [Build Infrastructure](#build-infrastructure)
- [Change Infrastructure](#change-infrastructure)


---
## What is infrastrucutre as code with Terraform
Terraform providers automatically create 

To deploy infrastructure with terraform
1. define scope: identify infrastructure for your project
2. Author: write configuration to define your infrastructure
3. Initialise: Install the required Terraform providers
4. Plan: Preview the changes terraform will make 
5. Apply: to make the changes 


---
## Install terraform

`choco install terraform` and check with `choco -help`


---
## Build Infrastructure
Deploy an EC2 instance on AWS 
- use terraform config to define EC2 instance
- use the terraform CLI to provision it on AWS 

Need to create an IAM user, then set terminal using `aws configure` access key and secret access key. 

Created 'learn-terraform-aws-instance' file which contains main.tf 

The `terraform {}` contains terraform settings, including the required providers terraform will use to provision your infrastructure. You can find providers in the terraform registry: 
https://registry.terraform.io/browse/providers
(or your company may have a private registry)

````
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.16"
    }
  }

  required_version = ">= 1.2.0"
}
````

The provider block configures the specified provider, in this case, 'aws'.

````
provider "aws" {
  region  = "us-west-2"
}
````

Resource blocks define components of your infrastructure. A resource might be a physical or virtural component, i.e. an EC2 instance. 

````
resource "aws_instance" "app_server" {
  ami           = "ami-830c94e3"
  instance_type = "t2.micro"

  tags = {
    Name = "ExampleAppServerInstance"
  }
}
````

Resource blocks have two strings before the block, resource type (i.e. `aws_instance`) and resource name (i.e. `app_server`). 
The prefix of the type maps to the name of the provider. 

Together, the type and name form a unique ID for the resource. Terraform uses this to identify the resource when planning changes and to identify the resource in other parts of the configuration. 

The ID for this EC2 instance is `aws_instance.app_server`

Resource blocks contain arguments which you use to configure the resource, i.e. `instance_type = "t2.micro"`


Whenever you create a new configuration or checkout an existing configuration for version control, you need to initialise your working directory with `terraform init`.
  - initialising the config directory downloads and installs the providers defined (in this case the AWS config)

Now we run the `terraform apply` command. Before it applies changes, it prints out the execution plan which describes the actions Terraform will take in order to change your infrastructure to match the configuration. Type 'yes' to confirm
  - Terraform now waits for the ec2 instance to become available. 
  - You've now created an EC2 using terraform! visit the EC2 console in AWS to see it, make sure to check the right region. 

---
## Change Infrastructure
When using terraform in production, we recommend that you use a version control system to manage your configuration files, and store your state in a remote backend such as terraform cloud or terraform enterprise.

