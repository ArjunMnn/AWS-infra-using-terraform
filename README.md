In this tutorial, we will use Terraform to set up a basic AWS infrastructure, including a VPC, public and private subnets, an Internet Gateway, a route table, a security group, and an EC2 instance hosting a simple website. The tutorial will guide you through each step, providing explanations for each Terraform block.

[](https://arjunmenon.hashnode.dev/day-66-terraform-hands-on-project#heading-prerequisites "Permalink")
-------------------------------------------------------------------------------------------------------

Step 1: Set Up Terraform Project
--------------------------------

Create a new directory for your Terraform project and navigate into it:

```
mkdir terraform_project
cd terraform_project

```

Initialize the Terraform project:

```
terraform init

```

[](https://arjunmenon.hashnode.dev/day-66-terraform-hands-on-project#heading-step-2-define-aws-provider-and-vpc "Permalink")
----------------------------------------------------------------------------------------------------------------------------

Step 2: Define AWS Provider and VPC
-----------------------------------

Create a file named [`main.tf`](http://main.tf) and add the initial code to define the AWS provider and create a Virtual Private Cloud (VPC):

```
provider "aws" {
  region = "your_region"
}

resource "aws_vpc" "my_vpc" {
  cidr_block = "10.0.0.0/16"
}

```

Replace `"your_region"` with your desired AWS region.

[](https://arjunmenon.hashnode.dev/day-66-terraform-hands-on-project#heading-step-3-add-public-and-private-subnets "Permalink")
-------------------------------------------------------------------------------------------------------------------------------

Step 3: Add Public and Private Subnets
--------------------------------------

Extend the [`main.tf`](http://main.tf) file to include the creation of public and private subnets within the VPC:

```
resource "aws_subnet" "public_subnet" {
  vpc_id                  = aws_vpc.my_vpc.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "your_az" # replace with your desired availability zone
  map_public_ip_on_launch = true
}

resource "aws_subnet" "private_subnet" {
  vpc_id     = aws_vpc.my_vpc.id
  cidr_block = "10.0.2.0/24"
  availability_zone = "your_az" # replace with your desired availability zone
}

```

Replace `"your_az"` with your desired availability zone.

[](https://arjunmenon.hashnode.dev/day-66-terraform-hands-on-project#heading-step-4-add-internet-gateway-and-route-table "Permalink")
-------------------------------------------------------------------------------------------------------------------------------------

Step 4: Add Internet Gateway and Route Table
--------------------------------------------

Extend the [`main.tf`](http://main.tf) file to include the creation of an Internet Gateway and a route table for the public subnet:

```
resource "aws_internet_gateway" "my_igw" {
  vpc_id = aws_vpc.my_vpc.id
}

resource "aws_route_table" "public_route_table" {
  vpc_id = aws_vpc.my_vpc.id
}

resource "aws_route" "public_route" {
  route_table_id         = aws_route_table.public_route_table.id
  destination_cidr_block = "0.0.0.0/0"
  gateway_id             = aws_internet_gateway.my_igw.id
}

```

[](https://arjunmenon.hashnode.dev/day-66-terraform-hands-on-project#heading-step-5-associate-route-table-with-public-subnet "Permalink")
-----------------------------------------------------------------------------------------------------------------------------------------

Step 5: Associate Route Table with Public Subnet
------------------------------------------------

Extend the [`main.tf`](http://main.tf) file to associate the route table with the public subnet:

```
resource "aws_route_table_association" "public_association" {
  subnet_id      = aws_subnet.public_subnet.id
  route_table_id = aws_route_table.public_route_table.id
}

```

[](https://arjunmenon.hashnode.dev/day-66-terraform-hands-on-project#heading-step-6-create-security-group "Permalink")
----------------------------------------------------------------------------------------------------------------------

Step 6: Create Security Group
-----------------------------

Extend the [`main.tf`](http://main.tf) file to create a security group allowing SSH and HTTP traffic:

```
resource "aws_security_group" "web_sg" {
  name        = "web_sg"
  description = "Security group for the web instance"
  vpc_id      = aws_vpc.my_vpc.id

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

```

[](https://arjunmenon.hashnode.dev/day-66-terraform-hands-on-project#heading-step-7-launch-ec2-instance-in-public-subnet "Permalink")
-------------------------------------------------------------------------------------------------------------------------------------

Step 7: Launch EC2 Instance in Public Subnet
--------------------------------------------

Extend the [`main.tf`](http://main.tf) file to create an EC2 instance in the public subnet:

```
resource "aws_instance" "web_instance" {
  ami                    = "ami-xxxxxxxxxxxxxxxxx" # Replace with Ubuntu AMI
  instance_type          = "t2.micro"
  subnet_id              = aws_subnet.public_subnet.id
  vpc_security_group_ids = [aws_security_group.web_sg.id]

  user_data = <<-EOF
    #!/bin/bash
    apt-get update
    apt-get install -y apache2
    systemctl start apache2
    systemctl enable apache2
    echo "<h1>Hello from Terraform</h1>" > /var/www/html/index.html
  EOF
}

```

Replace `"ami-xxxxxxxxxxxxxxxxx"` with the correct Ubuntu AMI for your region.

[](https://arjunmenon.hashnode.dev/day-66-terraform-hands-on-project#heading-step-8-apply-terraform-configuration "Permalink")
------------------------------------------------------------------------------------------------------------------------------

Step 8: Apply Terraform Configuration
-------------------------------------

Run the following command to apply the Terraform configuration:

```
terraform apply

```

Terraform will prompt you to confirm the changes. Type "yes" to proceed.

[](https://arjunmenon.hashnode.dev/day-66-terraform-hands-on-project#heading-step-9-verify-website-hosting "Permalink")
-----------------------------------------------------------------------------------------------------------------------

Step 9: Verify Website Hosting
------------------------------

After Terraform applies the configuration, open the EC2 instance's public IP or DNS in a web browser to verify that the website is hosted successfully.
