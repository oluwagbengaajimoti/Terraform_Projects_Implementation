## AWS Red Hat EC2 Deployment Script with EBS Volume Configuration Overview
This script deploys AWS instances powered by Red Hat Enterprise Linux 9, along with a security group that grants access to ports 22 (SSH), 2049 (TCP), and 2049 (UDP). It also attaches three Elastic Block Storage Volumes, each with a size of 50GiB, to the EC2 instance. Every resource is named logically, and it offers the public and private IP addresses of the created instances for future reference. Prior to running this script, please make sure that you have set up the AWS CLI with the necessary credentials.
```
# Define the AWS Provider Configuration
provider aws {
  region = "us-east-1" # Change to your desire AWS region
}

variable "ami" {
  default = "ami-026ebd4cfe2c043b2"
}
variable "subnet_id" {
  default = "subnet-0a6cf669303b14138"
}
variable "instance_type" {
  default = "t2.micro"
}
variable "keypair_name" {
  default = "MyNewKey"
}
variable "volume_names" {
  default = ["nfs_volume1", "nfs_volume2", "nfs_volume3"]
}

resource "aws_security_group" "nsf_server_sg" {
  name = "nsf_server_sg"
  description = "Allow traffic from port 22, and 4049"

  ingress {
    from_port = "22"
    to_port = "22"
    protocol = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port = "2049"
    to_port = "2049"
    protocol = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port = "2049"
    to_port = "2049"
    protocol = "udp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_instance" "nsf_server" {
  tags = {
    Name = "nsf_server"
  }
  ami = var.ami
  instance_type = var.instance_type
  associate_public_ip_address = true
  subnet_id = var.subnet_id
  vpc_security_group_ids = [aws_security_group.nsf_server_sg.id]
  key_name = var.keypair_name
}

resource "aws_ebs_volume" "ebs_volumes" {
  count = 3
  size = 50
  availability_zone = "us-east-1a"
  tags = {
    Name = "${element(var.volume_names, count.index)}"
  }
}

resource "aws_volume_attachment" "ebs_volume_attachments" {
  count      = 3
  device_name = "/dev/sdh"  # Adjust device names as needed
  volume_id   = element(aws_ebs_volume.ebs_volumes[*].id, count.index)
  instance_id = aws_instance.nsf_server.id
}

output "nsf_server_private_ip" {
  value  = [aws_instance.nsf_server.private.ip]
}

output "nsf_server_public_ip" {
  value  = [aws_instance.nsf_server.public.ip]
}
```
### Terraform Plan and Apply
Use the Terraform extension in VSCode or the terminal to run `terraform plan` and `terraform apply` to create and manage your AWS resources.
