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
-
- click `New` in `User variables for <user>` and add your access key and secret access key naming them both `AWS_ACCESS_KEY_ID` and `AWS_SECRET_KEY_ID`

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

6. Use `terraform destroy` to terminate the VM

## Creating VPC steps

![full_IaC_dia.png](file%2Ffull_IaC_dia.png)

1. Within our `main.tf` file we need to add code to create our VPC.

### Add provider:
```commandline
provider "aws" {
	region = "eu-west-1"
}
```
### Create a VPC on AWS:
```
# Create a VPC on AWS
resource "aws_vpc" "main" {
 cidr_block = "10.0.0.0/16"

 tags = {
   Name = "mateusz_tech221_VPC_terraform"
 }
}
```
### Create Subnets (public/private)
```
#Create Subnets
resource "aws_subnet" "public_subnets" {
 vpc_id     = aws_vpc.main.id
 cidr_block = var.public_subnet_cidrs

 tags = {
   Name = "mateusz_tech221_public_subnet"
 }
}

resource "aws_subnet" "private_subnets" {
 vpc_id     = aws_vpc.main.id
 cidr_block = var.private_subnet_cidrs

 tags = {
   Name = "mateusz_tech221_private_subnet"
 }
}
```
### Set-up internet Gate-way
```
# Set-up internet Gate-way
resource "aws_internet_gateway" "gw" {
 vpc_id = aws_vpc.main.id
 
 tags = {
   Name = "mateusz_tech221_VPC_IG_Terraform"
 }
}
```
### Create Route Table
```commandline
# Create Route Table
resource "aws_route_table" "public_rt" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.gw.id
  }
  
  route {  
    ipv6_cidr_block = "::/0"
    gateway_id      = aws_internet_gateway.gw.id
  }
    
  tags = {
    Name = "Public Route Table"
  }
}
```
### Make it accessable over internet
```commandline
# Make it accessable over internet
    
resource "aws_route_table_association" "public_1_rt_a" {
  subnet_id      = aws_subnet.public_subnets.id
  route_table_id = aws_route_table.public_rt.id
}
```
### Create Security Groups for HTTP and SSH access
```
# Create SG

resource "aws_security_group" "web_sg" {
  name   = "HTTP and SSH" 
  vpc_id = aws_vpc.main.id
        
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```
### Now we need to create service on AWS (EC2)
```
# EC2 instance
resource "aws_instance" "app_instance" {
        # which AMI to use
        ami = var.ami_id
        instance_type = "t2.micro"
        # do you need the public IP
        associate_public_ip_address = true
        key_name = "tech221"
    
        subnet_id                   = aws_subnet.public_subnets.id
        vpc_security_group_ids      = [aws_security_group.web_sg.id]
  
        tags = {
          Name = "tech221_mateusz_terraform_app"
    
        }
}  
```

2. The second step is to make sure that our private information are secured.
- create `variable.tf` file and within add this code:

```
variable "ami_id" {
    default = "..." <- here paste your AMI_id
}

variable "public_subnet_cidrs" {
 type        = string
 description = "Public Subnet CIDR values"
 default     = "10.0.5.0/24"
}
 
variable "private_subnet_cidrs" {
 type        = string
 description = "Private Subnet CIDR values"
 default     = "10.0.51.0/24"
}
```

3. Last step is to check if everything works properly. Navigate to your terraform file in GitBash and use command: `terraform plan`
4. If everything works properly in the same directory use: `terraform apply`

![terminal.png](file%2Fterminal.png)

### VPC created

![vpc_created.png](file%2Fvpc_created.png)

### Instance launched (with proper ports)

![instance_ports.png](file%2Finstance_ports.png)

### To save costs use: `terraform destroy` to terminate the EC2 instance 





