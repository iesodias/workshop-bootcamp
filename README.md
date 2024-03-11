# Workshop Bootcamp - DevopsFest 2024

## Lab-00: Aws Configure

1. Access the AWS Console:
2. Go to the AWS Console and log in to your account.
3. Navigate to the Security Credentials Page:
- In the upper-right corner, click on your account name or the "Services" dropdown and select "My Security Credentials."
- Access User Security Credentials:
- On the left navigation bar, click on "Users."
- Select the user for which you want to create credentials.
- Go to the "Security credentials" tab.
4. Create or View Access Credentials:
5. Under "Access keys," you can create a new key or view an existing one.
6. If creating a new one, click on "Create access key."
7. Take Note of the Access Credentials:
- After creation or viewing, take note of the "Access key ID" and "Secret access key." These details are needed to configure the AWS CLI.
8. Open Terminal or Command Prompt:
9. Open the terminal on your operating system.
10. Run the aws configure Command:
- Execute the following command and input the information when prompted:
```bash
aws configure
```
You will be prompted to enter the access key, secret access key, default region, and output format.

## Lab-01: Setup Environment

**Estimated time to complete:** 20 minutes

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
9. The above commands will install the required packages and then install Ansible using pip.
10. To install Ansible using pip, run the following commands:
```bash
sudo apt-get install -y python3-pip
sudo pip3 install ansible
```
10. Once installed, you can verify the Ansible installation by running:
```bash
ansible --version
```
11. Run the following AWS CLI command to create a new key pair and save the private key to a local file named kp_devops_fest.pem:
```bash
aws ec2 create-key-pair --key-name kp_devops_fest --query 'KeyMaterial' --output text > kp_devops_fest.pem

chmod 400 kp_devops_fest.pem
```
12. Create the folder structure for the project

```bash
mkdir mdc-terraform
```

## Lab-02: Storage Account Basic

**Estimated time to complete:** 20 minutes

**Work directory:** 01-mdc-terraform

1. Create a folder named "01-mdc-terraform" and navigate into it:
```bash
mkdir 01-mdc-terraform
cd 01-mdc-terraform
vi main.tf
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

## Lab-03: EC2 Aws Instance

**Estimated time to complete:** 40 minutes

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
  key_name      = "kp_devops_fest"

  tags = {
    Name = "control-ec2-mdc"
  }

  vpc_security_group_ids = [
    aws_security_group.allow_ssh_control.id,
    aws_security_group.allow_http_control.id
  ]
}

resource "aws_instance" "worker_instance" {
  ami           = var.ami_id
  instance_type = "t2.micro"
  key_name      = "kp_devops_fest"

  tags = {
    Name = "worker-ec2-mdc"
  }

  vpc_security_group_ids = [
    aws_security_group.allow_ssh_worker.id,
    aws_security_group.allow_http_worker.id
  ]
}

resource "aws_security_group" "allow_ssh_control" {
  name        = "allow_ssh_control"
  description = "Allow inbound SSH traffic for control instance"
  vpc_id      = var.vpc_id

  ingress {
    from_port   = 22
    to_port     = 22
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

resource "aws_security_group" "allow_http_control" {
  name        = "allow_http_control"
  description = "Allow inbound HTTP traffic for control instance"
  vpc_id      = var.vpc_id

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

resource "aws_security_group" "allow_ssh_worker" {
  name        = "allow_ssh_worker"
  description = "Allow inbound SSH traffic for worker instance"
  vpc_id      = var.vpc_id

  ingress {
    from_port   = 22
    to_port     = 22
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

resource "aws_security_group" "allow_http_worker" {
  name        = "allow_http_worker"
  description = "Allow inbound HTTP traffic for worker instance"
  vpc_id      = var.vpc_id

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
7. Crete the var.tf file:
```pl
variable "region" {
  default = "us-east-1"
}

variable "ami_id" {
  default = "ami-0e731c8a588258d0d"
}

variable "vpc_id" {
  default = "vpc-f1639e8c"
}
```
8. Open/Create the "outputs.tf"
```bash
vi output.tf
```
9. Add the desired outputs, such as the instance's public IP.
```pl
output "control_instance_public_ip" {
  value = aws_instance.control_instance.public_ip
}

output "worker_instance_public_ip" {
  value = aws_instance.worker_instance.public_ip
}
```
10. Initialize the Terraform project directory to download the required plugins:
```bash
terraform init
```
11. Preview the execution plan to see the changes that will be applied to the infrastructure:
```bash
terraform plan
```
12. Apply the changes and create the resource group:
```bash
terraform apply
```
12. AWS Console:
- Go to the AWS Management Console.
- Navigate to the "EC2" service.
13. EC2 Dashboard:
- In the EC2 Dashboard, click on "Instances" in the left navigation pane.
14. Locate Worker Instance:
- Find your worker EC2 instance in the list of instances.
15. Retrieve Public DNS:
- Click on the row corresponding to your worker instance to select it.
- At the bottom of the page, you should see details about your instance. Look for the "Public DNS (IPv4)" field. Copy this value.
```bash
ec2-3-91-150-100.compute-1.amazonaws.com # Exemple WOKER an
```
16. SSH Connection:
- Open a terminal on your local machine.
- Use the following SSH copy control EC2 instance. 
```bash
# Copy the key pair to the worker EC2 instance
scp -i "/Users/iesodias/Documents/Projetos/workshop-bootcamp/mdc-terraform/kp_devops_fest.pem" "/Users/iesodias/Documents/Projetos/workshop-bootcamp/mdc-terraform/kp_devops_fest.pem" ec2-user@ec2-54-211-206-27.compute-1.amazonaws.com:~
```

## Lab-04: Configuring and Running Ansible on the Control Instance (CONTROL EC2):

**Estimated time to complete:** 30 minutes

**Work directory:** 02-mdc-terraform

1. Connect to the Control Instance:
```bash
# SSH into the worker EC2 instance
ssh -i "/Users/iesodias/Documents/Projetos/workshop-bootcamp/mdc-terraform/kp_devops_fest.pem" ec2-user@ec2-54-211-206-27.compute-1.amazonaws.com
```
2. Install Ansible:
- Install Ansible on your CONTROL EC2 to enable remote management of other instances. Run the following commands:
```bash
sudo yum install -y python3-pip
sudo pip3 install ansible
```
3. Copy Worker Machine's IP:
- Save the Worker EC2 instance's IP address into an Ansible inventory file. This file is crucial for Ansible to know which machines to manage. Run this command:
```bash
echo -e "mdc-target1 ansible_host=34.227.16.33 ansible_user=ec2-user ansible_ssh_private_key_file=/home/ec2-user/kp_devops_fest.pem" > inventory.txt
```
4. Test Connection to Worker Machine:
- Ensure Ansible can communicate with the Worker EC2 instance:
```bash
ansible mdc-target1 -i inventory.txt -m ping
```
5. Create the YAML Playbook (playbook.yaml):
- Craft an Ansible playbook to install Nginx, start its service, and create a simple webpage:
```yaml
- hosts: mdc-target1
  become: yes
  tasks:
    - name: Instalar o Nginx
      yum:
        name: nginx
        state: present

    - name: Iniciar o serviço Nginx
      service:
        name: nginx
        state: started

    - name: Habilitar o serviço Nginx para iniciar na inicialização
      service:
        name: nginx
        enabled: yes

    - name: Criar arquivo index.html
      copy:
        content: "Bem Vindos a mentoria devops cloud"
        dest: /usr/share/nginx/html/index.html
```
6. Run the Playbook:
- Execute the playbook to apply the configurations on the Worker EC2:
```bash
ansible-playbook -i inventory.txt playbook.yaml
```
7. Verify the Application on Port 80:
- Open your web browser and enter your Worker EC2 instance's IP address followed by ":80" (e.g., http://YOUR_IP:80). This should display the welcome message.
- Make sure your Worker EC2 instance is reachable on port 80 and adjust the playbook as needed for your specific application requirements.