# Orchestration with Terraform

Not using AWS cloud formation as this could then only be used on AWS. It is cloud dependent. Terraform is cloud independant and so it can be used on any cloud, local or hybrid. 

## What is it?

Terraform is an open-source Infrastructure as Code (IaC) tool developed by HashiCorp. It allows users to define and manage infrastructure resources, such as virtual machines, networks, storage, and more, using a declarative configuration language. With Terraform, you can describe your desired infrastructure state in code, and Terraform takes care of provisioning and managing the resources to match that state.

## Why use it and the benefits

1. **Infrastructure as Code (IaC):**
   Terraform's core benefit lies in its ability to treat infrastructure as code. Infrastructure is defined using human-readable configuration files, which makes it versionable, maintainable, and allows for collaboration across teams. This approach brings consistency and repeatability to infrastructure provisioning and management.

2. **Multi-Cloud and Multi-Provider Support:**
   Terraform supports various cloud providers (e.g., AWS, Azure, Google Cloud Platform) and on-premises infrastructure. This flexibility allows users to manage resources across multiple cloud providers and maintain a consistent deployment workflow, regardless of the underlying infrastructure.

3. **Declarative Configuration:**
   Terraform uses a declarative approach, meaning you define the desired state of your infrastructure rather than writing procedural scripts. This makes it easier to understand, manage, and modify infrastructure configurations.

4. **Dependency Management and Resource Graph:**
   Terraform automatically handles resource dependencies and creates a dependency graph. It ensures resources are provisioned in the correct order, reducing the risk of errors and ensuring the infrastructure is created efficiently.

5. **Change Management and Planning:**
   Terraform provides a preview of the changes it will apply before executing them. This allows users to review and validate proposed changes, reducing the likelihood of unexpected disruptions in the infrastructure.

6. **State Management:**
   Terraform keeps track of the state of the managed infrastructure in a state file. This state file is essential for tracking changes and performing updates or modifications to the infrastructure.


#### How to download terraform on Windows:

1. **Download Terraform**:
   - Go to the official Terraform website: [Terraform Downloads](https://www.terraform.io/downloads.html)
   - Download the Windows 64-bit version of Terraform (usually a zip archive).

2. **Extract the Archive**:
   - Once the download is complete, locate the downloaded zip archive.
   - Right-click on the zip file and select "Extract All..." to extract its contents to a folder of your choice. You can use the built-in Windows extraction tool or a third-party file archiver like 7-Zip.

3. **Add Terraform to System Path**:
   - To use Terraform from any directory on the command prompt, you need to add the folder containing the Terraform executable to your system's PATH environment variable.
   - Here's how you can do it:
     - Open the Start menu and search for "Environment Variables."
     - Select "Edit the system environment variables."
     - Click the "Environment Variables..." button.
     - In the System Variables section, scroll down and find the "Path" variable. Click "Edit."
     - Click "New," and then add the path to the folder where you extracted the Terraform executable. For example: `C:\Users\elena\terraform_1.5.3_windows_amd64`.

4. **Verify Installation**:
   - Open a new Command Prompt (or PowerShell) window.
   - Type `terraform --version` and press Enter.
   - If Terraform is correctly installed and added to the system PATH, it will display the version information.

![Alt text](imgs/terraform-downloaded.png)

## Infrastructure as Code Overview

![IaC](imgs/terraform/IaC-overview.png)

1. **Terraform Workflow**:
   - Use Terraform to define and provision the required infrastructure resources in your cloud provider (e.g., AWS, Azure, GCP) using Terraform configuration files.
   - Terraform ensures that the infrastructure is created and configured according to the defined specifications.

2. **Ansible Workflow**:
   - Once the infrastructure is provisioned by Terraform, you can use Ansible playbooks to configure and manage the software and settings on the provisioned resources.
   - Ansible connects to the servers and executes the tasks defined in the playbooks to ensure the desired configurations are applied.

Terraform needs to use the secret keys to be able to access AWS. These have been added as enviroment variables on windows. aws_secret_access_key

## Launch ec2 instance using terraform

I created a folder called terraform and a file called main.tf. In this file is the script which will create the ec2 instance when you use the command `terraform init`.

```
# Provider name
provider "aws"{
        # where in aws
        region = "eu-west-1"
}

# Launch an ec2 in Ireland
resource "aws_instance" "app_instance"{

# which machine/OS version etc AMI-id
  ami = "ami-0943382e114f188e8"

# What type of instance t2-micro
  instance_type = "t2.micro"

# Is the public ip required?
  associate_public_ip_address = true

# what would you line to name it tech241-elena-terraform-app
  tags = {
       Name = "tech241-elena-terraform-app"

  }
}
```

`terraform plan` - checks if there are any errors in the script: 

![Alt text](imgs/terraform/terraform-plan.png)

`terraform apply` - will create the instance

![Alt text](imgs/terraform/terraform-apply.png)

## VPC

![Alt text](imgs/terraform/isolated-vpc.png)

**App instance:**

Ingres:
- HTTP - TCP - 80 - 0.0.0.0/0
- SSH - TCP - 22 - 0.0.0.0/0
- Custom -TCP - 3000 - 0.0.0.0/0

Egres:
- All traffic 0.0.0.0/0

**DB instance:**

Ingres:
- SSH - TCP - 22 - 0.0.0.0/0
- Custom - TCP - 27017 - 0.0.0.0/0

Egres:
- All traffic 0.0.0.0/0