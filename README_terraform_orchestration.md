IaC Terraform
-

![IaC.png](file%2FIaC.png)

## How to set up Terraform?

1. If we don't have Terraform installed we can always follow the steps:

https://developer.hashicorp.com/terraform/downloads

- open `Windows PowerShell` as admin
- download `Chocolatey` by entering this command:
```commandline
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```
- download Terraform using `choco`: ` choco install terraform`
- to check if it works we can run this command in our fresh GitBash terminal: `terraform`

2. Now we need to configure AWS access keys in environmental variable.

- on your local machine type: `Edit the system environmental variables`
- click `Environmental Variables`
![env_var.png](file%2Fenv_var.png)
- click New and add you access key and secret access key naming them both `AWS_ACCESS_KEY_ID` and `AWS_SECRET_KEY_ID`

## Creating an AWS Instance with Terraform.

1. Create new directory for terraform files: `mkdir tech221-terraform`
2. First we need to create `main.tf` file: `nano main.tf`
3. Enter the following:
```commandline
# Write a ascript to launch resources on the cloud 
# codify terraform - syntax - {key = value}
provider "aws" {
    region = "eu-west-1"
}

# let's create a service on AWS
# add resource
# specify which service
resource "aws_instance" "app_instance"{
    # choose ami
    ami = "ami-0c5a2f4a8680af2d2"
    instance_type = "t2.micro"
    associate_public_ip_address = true
    tags = {
        Name = "tech221-fatima-terraform-app"
    }
}

```
4. Now enter the command `terraform plan` - this will check if syntax is correct
5. To launch the instance from AMI according to how you have configured our file use: `terraform apply`

![instance_running.png](file%2Finstance_running.png)

6. 6.Use `terraform destroy` to terminate the VM


















