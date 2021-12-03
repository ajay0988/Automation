## this is provider####
provider "aws" {
  region                  = "ap-south-1"
  profile                 = "default"
}
#### create the vpc#####

resource "aws_vpc" "my-vpc" {
  cidr_block = "172.16.0.0/16"

  tags = {
    Name = "custom_vpc"
  }
}

#### create VPC peering Connection#########
resource "aws_vpc_peering_connection" "foo" {
  peer_vpc_id   = aws_vpc.my-vpc.id
  vpc_id="vpc-d9bb7bb2"
  auto_accept   = true
  tags = {
    Name = "VPC Peering between Custom and default"
  }
  depends_on = [
    aws_vpc.my-vpc
  ]

}

#### create the internet gate way inside above vpc#########

resource "aws_internet_gateway" "IGW" {
  vpc_id = aws_vpc.my-vpc.id

  tags = {
    Name = "my-igw"
  }
  depends_on = [
    aws_vpc.my-vpc
  ]
}
### create route table inside above vpc###
resource "aws_route_table" "route-tb" {
  vpc_id = aws_vpc.my-vpc.id

  route = [
   {
    cidr_block = "0.0.0.0/0"
    gateway_id  = aws_internet_gateway.IGW.id
    carrier_gateway_id         = ""
    destination_prefix_list_id = ""
    egress_only_gateway_id     = ""
    instance_id                = ""
    ipv6_cidr_block            = ""
    local_gateway_id           = ""
    nat_gateway_id             = ""
    network_interface_id       = ""
    transit_gateway_id         = ""
    vpc_endpoint_id            = ""
    vpc_peering_connection_id  = ""

  }
  ]
  tags = {
    Name = "myroutetable"
  }

  depends_on = [
    aws_vpc.my-vpc,
    aws_internet_gateway.IGW
  ]
}

### create route table for deafult vpc##





### create subnet####

resource "aws_subnet" "public-subnet" {
  vpc_id     = aws_vpc.my-vpc.id
  cidr_block = "172.16.1.0/24"
  availability_zone= "ap-south-1a"

  tags = {
    Name = "Public subnet"
  }

  depends_on = [
    aws_vpc.my-vpc
  ]
}

## associate subnet with route table###
resource "aws_route_table_association" "public-rt-asso" {
  subnet_id      = aws_subnet.public-subnet.id
  route_table_id = aws_route_table.route-tb.id
depends_on = [
    aws_subnet.public-subnet,
    aws_route_table.route-tb
  ]

}

### associate default route table with default vpc###




####create security group##

resource "aws_security_group" "my-sg" {
  name        = "allow_tls"
  description = "Allow all inbound traffic"
  vpc_id      = aws_vpc.my-vpc.id

  ingress {
    description      = "TLS from VPC"
    from_port        = 0
    to_port          = 0
    protocol         = "-1"
    cidr_blocks      = ["0.0.0.0/0"]

  }

  egress {
    from_port        = 0
    to_port          = 0
    protocol         = "-1"
    cidr_blocks      = ["0.0.0.0/0"]

  }

  tags = {
    Name = "custom-sg"
  }
}

### launch ec2 instance###
resource "aws_instance" "master_os" {
  ami           = "ami-0a23ccb2cdd9286bb"
  instance_type = "t2.micro"
  subnet_id= aws_subnet.public-subnet.id
  key_name= "devops_project_key"

  associate_public_ip_address = "true"
  vpc_security_group_ids= [aws_security_group.my-sg.id]
  #security_groups=aws_security_group.my-sg.name
  connection {
    type     = "ssh"
    user     = "ec2-user"
    private_key = file("devops_project_key.pem")
    host     = aws_instance.master_os.public_ip
  }
   provisioner "remote-exec" {
    inline = [
      "sudo amazon-linux-extras install epel -y",
      "sudo yum-config-manager --enable epel",
      "sudo yum install sshpass -y",
      "sudo sshpass -p ajay scp -o StrictHostKeyChecking=no root@13.127.98.130:/terraform-ws/script .",
      "sudo chmod +x script",
      "sudo ./script"
    ]
  }

  tags = {
    Name = "nginx-docker-host"
  }
  depends_on = [

    aws_subnet.public-subnet,



  ]
}

### update ansible inventory###

resource "null_resource" "update_the_ansible_inventory" {
  connection {
    type     = "ssh"
    user     = "root"
    password = "ajay"
    host     = "172.31.37.133"
  }

  provisioner "remote-exec" {
    inline = [
      "cd /ansible-ws",
      "echo ${aws_instance.master_os.public_ip}  ansible_ssh_pass=ajay ansible_user=root > hosts",
    ]
  }
  depends_on = [
    aws_instance.master_os
  ]
}
