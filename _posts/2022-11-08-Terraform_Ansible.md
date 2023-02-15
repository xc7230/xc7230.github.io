---
title: '[AWS] Terraform+Ansible'
categories:
    - IaC
    - Terraform
    - Ansible
tag:
    - IaC
    - Terraform
    - Ansible


last_modified_at: 2022-11-08T09:00:00+09:00
toc: true
---




# Terraform+Ansible
## 설치
1. Centos 8에 테라폼 설치<br/>
```shell
yum install -y yum-utils
yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
yum -y install terraform
```
2. aws cli설치<br/>
```shell
yum install -y zip
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

3. aws credentials 설정<br/>
```shell
aws configure
# Access Key 내용 입력
# Region: ap-northeast-2
# 마지막 그냥 엔터
```

4. 가상환경에 key설정하기
```shell
cd ~
vi key  # .pem형식으로 변한 키내용 입력.
chmod 600 key 
```

## Terrafrom Ansible연동 실습
- ansible playbook생성<br/>
```shell
vi apache2-install.yaml
```
`apache2-install.yaml`
```yaml
- hosts: all
  tasks:
  - name: "Insatll httpd server"
    shell: |
      apt update -y
      apt install -y apache2
    become: yes
```

- terraform 생성<br/>
```shell
mkdir terraform
cd terraform
```
`variables.tf`
```tf
variable "app_server_ami" {
  type = string
  default = "ami-068a0feb96796b48d"
}

variable "app_server_type" {
  type = string
  default = "t2.micro"
}

variable "my_ec2_keyname" {
  type = string
  default = "cloudkey"
}

```

`main.tf`
```tf
provider "aws" {
  region  = "ap-northeast-2"
}

## vpc
resource "aws_vpc" "my-vpc2" {
  cidr_block       = "200.200.0.0/16"
  instance_tenancy = "default"
  enable_dns_hostnames = true
  tags = {
    Name = "my-vpc2"
  }
}

## subnet
resource "aws_subnet" "my2-subnet-1" {
  vpc_id     = aws_vpc.my-vpc2.id
  cidr_block = "200.200.10.0/24"
  availability_zone = "ap-northeast-2a"
  map_public_ip_on_launch = true
  tags = {
    Name = "my2-subnet-1"
  }
}
resource "aws_subnet" "my2-subnet-2" {
  vpc_id     = aws_vpc.my-vpc2.id
  cidr_block = "200.200.20.0/24"
  availability_zone = "ap-northeast-2b"

  tags = {
    Name = "my2-subnet-2"
  }
}

## gateway
resource "aws_internet_gateway" "my-gw2" {
  vpc_id = aws_vpc.my-vpc2.id

  tags = {
    Name = "my-gw2"
  }
}
## route
resource "aws_default_route_table" "my2-route-table" {
  default_route_table_id = aws_vpc.my-vpc2.default_route_table_id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.my-gw2.id
  }

  tags = {
    Name = "my2-route-table"
  }
}


## security group
resource "aws_security_group" "ec2_allow_rule2" {
    vpc_id      = aws_vpc.my-vpc2.id
  ingress {
    description = "SSH"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]

  }

  ingress {
    description = "HTTP"
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

## ec2
resource "aws_instance" "app_server2" {
  associate_public_ip_address = true
  ami           = var.app_server_ami
  instance_type = var.app_server_type
  vpc_security_group_ids = [aws_security_group.ec2_allow_rule2.id]
  subnet_id = aws_subnet.my2-subnet-1.id
  key_name = "cloudcamp"
  tags = {
    Name = "ExampleAppServerInstance"
  }
  provisioner "local-exec" {
    command = <<EOF
      sleep 60
      EOF
  }
  provisioner "local-exec" {
    command = <<EOF
      echo "[all]" > inventory
      echo "${aws_instance.app_server2.public_ip} ansible_ssh_user=ubuntu ansible_ssh_private_key_file=~/key" >> inventory
      EOF
  }

  provisioner "local-exec" {
    command = <<EOF
      ANSIBLE_HOST_KEY_CHECKING=False \
      ansible-playbook -i inventory ~/apache2-install.yaml
      EOF
  }

}

output "app_server_public_ip" {
  description = "aws instance public_ip"
  value = aws_instance.app_server2.public_ip
}

```
```shell
terraform init
terraform apply
```

- 확인<br/>
생성된 EC2인스턴스 아이피를 입력했을때, 아파치 웹서버가 실행되면 성공<br/>
![image](/assets/img/image/tera_ansi/1.png)<br/>

