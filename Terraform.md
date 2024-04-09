# Terraform Code Blocks

Creating a AWS EC2 instance with Terraform

#### Pre-requisits

1. We need to mention the required_providers in the terraform block so that terrform can set up the environment
    
like this :

```tf
terraform {
  required_providers {
      aws = {
      source = "hashicorp/aws"
      version = "5.44.0"
    }
  }
}
```

2. then in ```provider``` block we need to mention the provider with the region and secrets
```tf
provider "aws" {
    region = "us-east-1"
    access_key = ""
    secret_key = ""
}
```
#### Creating a EC2 instance in real world scenario requires three sub recourses mainly,
1. VPC (Virtual Private Cloud)
2. Subnet
3. Security Group

VPC

```tf
resource "aws_vpc" "main" {
  cidr_block       = "10.0.0.0/16"
  instance_tenancy = "default"

  tags = {
    Name = "main"
  }
}
```
Subnet

```tf
resource "aws_subnet" "main_subnet" {
  vpc_id     = aws_vpc.main.id         # reference the related VPC id here
  cidr_block = "10.0.1.0/24"
  availability_zone = "us-east-1a"     # optional

  tags = {
    Name = "main_subnet"
  }
}
```
Security Group

```tf
resource "aws_security_group" "main_security_group" {

  name = "ec2-security-group"
  description = "Security group for ec2 instances"
  vpc_id = aws_vpc.main.id     # reference the related VPC id 

  tags = {
    Name = "main_security_group"
  }

  # Allow all outbound traffic
  egress {
    from_port = 0
    to_port   = 0
    protocol = "-1"              
    cidr_blocks = ["0.0.0.0/0"]
  }

  # Allow SSH inbound traffic
  ingress {
    from_port = 22
    to_port   = 22
    protocol = "tcp"
    cidr_blocks = ["0.0.0.0/0"] # You can restrict this to specific IP addresses for better security
  }

}
```

Note: If we only Create the VPC and Security group and assign those to the EC2 it will through an error saying, that the recourses are not in the same network.

Ec2 Instance Block

```tf
resource "aws_instance" "web_server" {
  ami           = "ami-080e1f13689e07408" # Update with your desired AMI ID
  instance_type = "t2.micro"

  vpc_security_group_ids = [aws_security_group.main_security_group.id]
  subnet_id = aws_subnet.main_subnet.id  # Associate with the public subnet
  associate_public_ip_address = true  # Allocate a public IP address to the instance
  # key_name = aws_key_pair.test-terraform.key_name  # ti add jkey from referencing it, key must be generated locally and the public key must be referenced check the block below

  # Optional but good security measure
  metadata_options {
    http_tokens     = "required"  # Require the use of IMDSv2
    http_put_response_hop_limit = 1  # Ensure only one hop for HTTP PUT requests
  }

  # Add tags (optional)
  tags = {
    Name = "Web Server Instance"
  }
}
```

Pub key referencing 
```tf
resource "aws_key_pair" "test-terraform" {
  key_name   = "test-terraform"
  public_key = file("~/Documents/key-pairs/test-terraform.pub")
}
```
 


