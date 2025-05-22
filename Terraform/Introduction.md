# What is Terraform?

Terraform is an open-source Infrastructure as Code (IaC) tool developed by HashiCorp. It allows you to define, provision, and manage cloud infrastructure and services using declarative configuration files. Terraform supports multiple cloud providers such as Azure, AWS, and Google Cloud, enabling consistent and repeatable infrastructure deployments.

**Key Features:**
- Declarative configuration language (HCL)
- Supports multi-cloud and hybrid environments
- Enables version control of infrastructure
- Automates resource provisioning and management


   ************* Terraform uses Three faces
   >>> init ==== Terraform initialises the project and identifies the provider to be used for the target environments.
   >>> plan === Terraform drafts a plan to get to the target phase.
   >>> apply === Terraform makes necessary changes required to mak it reflect on the target phase.

                               __________________________STRUCTURAL VIEW (Configuartion File)______________________________________

         resource "aws_instance" "webserver" {                //"aws_instance" = reource type /provider  //"webserver"=reource name 
            ami = "ami-0c2f25c1f66a1ff4d"                     // arguments.
            instance_type = "t2.micro"                        // arguments.
         }


                              _________________________TERRAFORM CONSISTS OF FOUR STEPS______________________________________
         1) Write the Configuration file. 
         2) Run the Terraform INIT command.
         3) Review the excusion plan using the terraform PLAN.
         4) Apply the changes using Terraform APPLY command.

               _______COMMON TERRAFORM COMMANDS________
         *** Terraform init
         *** terraform plan
         *** terraform apply
         *** terraform destroy.


                             ___________________# Using a local set up!__________________________ 

                     terraform {
                           required_providers {
                           local = {
                           source = "hashicorp/local"
                        }
                     }
                  }

         provider "local" {}

            resource "local_sensitive_file" "pet" {
            content              = "This is a sensitive file"
            filename             = "${path.module}/files/outputfile"
            file_permission      = "0700"
            directory_permission = "0777"
         }
# This is a local file that will be created in the current directory