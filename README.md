## Terraform learnings 

AWS documentation here: 
https://registry.terraform.io/providers/hashicorp/aws/latest/docs


tutorials from here: https://learn.hashicorp.com/collections/terraform/aws-get-started


Do these: 
- https://learn.hashicorp.com/collections/terraform/certification-associate-tutorials
- https://learn.hashicorp.com/tutorials/terraform/lambda-api-gateway
- https://learn.hashicorp.com/collections/terraform/automation (do some of these)
- https://learn.hashicorp.com/collections/terraform/recommended-patterns (do whats interesting)

Contents: 
- [What is infrastrucutre as code with Terraform?](#what-is-infrastrucutre-as-code-with-terraform)
- [Install terraform](#install-terraform)
- [Build Infrastructure](#build-infrastructure)
- [Change Infrastructure](#change-infrastructure)
- [Destroy Infrastructure](#destroy-infrastructure)
- [Define Input Variables](#define-input-variables)
- [Query Data with Outputs](#query-data-with-outputs)
- [Store Remote State (terraform cloud)](#store-remote-state-terraform-cloud)
- [Lock and Upgrade Provider versions](#lock-and-upgrade-provider-versions)
- [Modules Overview](#modules-overview)


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

We make a change to the AMI: `ami           = "ami-08d70e59c07c61a3a"` and then run `terraform apply` again. 

When there is a -/+ prefix, it means terraform will destroy and recreate the instance rather than update it in place. type `yes` and it will destroy it and recreate it in its place

you can then use `terraform show` to see the new values associated with the instance.


---
## Destroy Infrastructure

Once you no longer need infrastructure, you may want to delete it to save security and cost. 

`terraform destroy` destroys resources defined in your terraform configuration, it does not destroy resources running elsewhere that are not managed by your configuration. 

running `terraform destroy` prints out an execution plan telling you what it will do. the "-" prefix indicates the instance will be destroyed. 


---
## Define Input Variables

Terraform can use variables to make your configuration more dynamic and flexible. 

makes a variables.tf file:
````
variable "instance_name" {
  description = "Value of the Name tag for the EC2 instance"
  type        = string
  default     = "ExampleAppServerInstance"
}
````

then in main.tf references it 
````
resource "aws_instance" "app_server" {
  ami           = "ami-08d70e59c07c61a3a"
  instance_type = "t2.micro"

  tags = {
    Name = var.instance_name // references it here
  }
}
````

use `terraform apply`

it is created with the same name, as we replaced the variable with the same name.
can now run `terraform apply -var 'instance_name=YetAnotherName'`
using the command line like above will not save the new variable. 
terraform has many ways to use and set varaibles so you can avoid having to set them each time. 

---
## Query Data with Outputs

Use output values to organise data to be easily queried and shown back to the user. 

create an 'outputs.tf' file like below:
````
output "instance_id" {
  description = "ID of the EC2 instance"
  value       = aws_instance.app_server.id
}

output "instance_public_ip" {
  description = "Public IP address of the EC2 instance"
  value       = aws_instance.app_server.public_ip
}
````

in the above case the "aws_instance.app_server" is the resource ID that we can reference
=> we can then run `terraform apply` and the output will be shown. 
=> we can also run `terraform output` and the output will be shown.


---
## Store Remote State (terraform cloud)

you have built, changed, and destroyed infrastructure from your local machine. This is great for testing and development, but in production environments you should keep your state secure and encrypted, where your teammates can access it to collaborate on infrastructure. The best way to do this is by running Terraform in a remote environment with shared access to state.
i.e. by using terraform cloud 

sign up to terraform cloud: https://app.terraform.io/public/signup/account

in main.tf add this code: 
````
terraform {
  backend "remote" { // this 'backend' block is what we added
    organization = "mark-learning"
    workspaces {
      name = "Example-workspace"
    }
  }
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.16"
    }
  }

  required_version = ">= 1.2.0"
}
````

login to terraform using the CLI `terraform login`
now you have configured the remote backend, run `terraform init` to re-initialize the config and migrate the state to the cloud.

now terraform has migrated your state to the cloud, delete the local state file (terraform.tfstate)

the `terraform init` command greated the example-workspace in your terraform organisation. 
your workspace needs to be configured with your AWS credentials to authenticate the AWS provider. 
- naviggate to your workspace in the terraform cloud and select the variables cloud 
  - note: something like this: `https://app.terraform.io/app/<organization name>/workspaces/<workspace name>`
- go to the variables tab, click add variable, select environment variables and add aws access key and value as "AWS_ACCESS_KEY_ID" and the aws secret key as "AWS_SECRET_ACCESS_KEY"

now run `terraform apply` to trigger a run in terraform cloud

terraform is now storing your state remotely in terraform cloud. remote state keeps working as a team easier and allows you to store your secrets in one location. 

---
## Lock and Upgrade Provider versions 

you can set your versions using various options, i.e.
````
terraform {
  required_providers {
    random = {
      source  = "hashicorp/random"
      version = "3.0.0" // will always ask for exactly 3.0.0
    }

    aws = {
      source  = "hashicorp/aws"
      version = ">= 2.0.0" // will always ask for a version greater than 2.0.0
    }
  }

  required_version = ">= 1.1"
}
````

when you `terraform init` the above config, terraform will look for the terraform.lock.hcl file (all terraform 1.1 or higher use this) and if the terraform lock file (similar to package.lock) has a specific version saved, i.e. 2.5 for hasicorp/aws, it will use that even if there is a later version i.e. 4.0.0. We should always commit the terraform.lock.hcl file.

to get the latest version (and see it updated in the lock file), `terraform init -upgrade`

its generally better to lock a version in the terraform config rather than use something like ">=". 

---
## Modules Overview