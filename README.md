# workshop-bootcamp

## Lab-01: Setup Environment

**Estimated time to complete:** 15 minutes

- CloudShell
- Terraform
- Ansible

1. Download Terraform binary for Linux (64-bit):
```bash
curl -LO https://releases.hashicorp.com/terraform/1.0.8/terraform_1.0.8_linux_amd64.zip
```
2. Unzip the downloaded Terraform binary:
```bash
unzip terraform_1.0.8_linux_amd64.zip
```
3. Create the directory where you'll store the Terraform binary:
```bash
mkdir -p ~/.local/bin
```
4. Move the Terraform binary to the newly created directory:
```bash
mv terraform ~/.local/bin/
```
5. Add Terraform to your PATH by appending it to the .bashrc file:
```bash
echo 'export PATH=$HOME/.local/bin:$PATH' >> ~/.bashrc
```
6. Source the .bashrc file to apply the changes immediately:
```bash
source ~/.bashrc
```
7. Verify the Terraform installation by checking the version:
```bash
terraform --version
```
8. To install Ansible using pip, run the following commands:
```bash
sudo amazon-linux-extras install epel -y
sudo yum install -y python3-pip
sudo pip3 install ansible
```
9. The above commands will install the required packages and then install Ansible using pip.
10. Once installed, you can verify the Ansible installation by running:
```bash
ansible --version
```
11. Run the following AWS CLI command to create a new key pair and save the private key to a local file named mentoria_keypair.pem:
```bash
aws ec2 create-key-pair --key-name mentoria_keypair --query 'KeyMaterial' --output text > mentoria_keypair.pem
```


### Create the folder structure for the project

```bash
mkdir mdc-terraform
```

## Lab-02: Storage Account Basic

**Estimated time to complete:** 15 minutes

**Work directory:** 01-mdc-terraform

1. Create a folder named "01-mdc-terraform" and navigate into it:
```bash
mkdir 01-mdc-terraform
cd 01-mdc-terraform
touch main.tf
```
2. Create a file named "main.tf" inside the "01-mdc-terraform" folder.
3. In the "main.tf" file, add the following code to configure the Microsoft Azure provider and create a S3:

```pl
provider "aws" {
  region = "us-east-1"
}

provider "random" {}

resource "random_id" "bucket_id" {
  byte_length = 4
}

resource "aws_s3_bucket" "example_bucket" {
  bucket = "mentoriadevops-${random_id.bucket_id.hex}"

  acl           = "private"

  tags = {
    Name        = "mentoria-devops"
    Environment = "Production"
  }
}

```
4. Initialize the Terraform project directory to download the required plugins:
```bash
terraform init
```
5. Preview the execution plan to see the changes that will be applied to the infrastructure:
```bash
terraform plan
```
6. Apply the changes and create the resource group:
```bash
terraform apply
```
7. To clean up and destroy all resources created by Terraform, run:
```bash
terraform destroy
```

## Lab-02: EC2 Aws Instance

**Estimated time to complete:** 15 minutes

**Work directory:** 02-mdc-terraform

1. Create a folder named "02-mdc-terraform" and navigate into it:
```bash
mkdir 02-mdc-terraform
cd 02-mdc-terraform
touch main.tf
```
2. Create a file named "main.tf" inside the "02-mdc-terraform" folder.
3. In the "main.tf" file, add the following code to configure the AWS provider and create a EC2:

```pl
provider "aws" {
  region = var.region
}

resource "aws_instance" "control_instance" {
  ami           = var.ami_id
  instance_type = "t2.micro"
  key_name      = "mentoria_keypair"  # Replace with your key pair name

  tags = {
    Name = "control-ec2-mdc"
  }

  vpc_security_group_ids = [
    aws_security_group.allow_ssh.id,
    aws_security_group.allow_http.id
  ]
}

resource "aws_instance" "worker_instance" {
  ami           = var.ami_id
  instance_type = "t2.micro"
  key_name      = "mentoria_keypair"  # Replace with your key pair name

  tags = {
    Name = "worker-ec2-mdc"
  }

  vpc_security_group_ids = [
    aws_security_group.allow_ssh.id,
    aws_security_group.allow_http.id
  ]
}

resource "aws_security_group" "allow_ssh" {
  name        = "allow_ssh"
  description = "Allow inbound SSH traffic"
  vpc_id      = var.vpc_id

  ingress {
    from_port = 22
    to_port   = 22
    protocol  = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_security_group" "allow_http" {
  name        = "allow_http"
  description = "Allow inbound HTTP traffic"
  vpc_id      = var.vpc_id

  ingress {
    from_port = 80
    to_port   = 80
    protocol  = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```
7. Open/Create the "outputs.tf"

```bash
vi output.tf
```
8. Add the desired outputs, such as the instance's public IP.
```pl
output "control_instance_public_ip" {
  value = aws_instance.control_instance.public_ip
}

output "worker_instance_public_ip" {
  value = aws_instance.worker_instance.public_ip
}
```
9. Initialize the Terraform project directory to download the required plugins:
```bash
terraform init
```
10. Preview the execution plan to see the changes that will be applied to the infrastructure:
```bash
terraform plan
```
11. Apply the changes and create the resource group:
```bash
terraform apply
```
