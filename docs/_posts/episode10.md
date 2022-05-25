---
layout: page
name: Episode 10 - Demonstration of Terraform Modules; Deploy a VM in AWS, Azure and GCP at once

---

# Episode 10 - Demonstration of Terraform Modules; Deploy a VM in AWS, Azure and GCP at once

<div class="video-container"><iframe width="560" height="315" src="https://www.youtube.com/embed/v4VeMa-OgL8" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></div>

Welcome to Episode 10 of the “Head in the Clouds” Video Series. I am Ken Hartman, a SANS Certified Instructor and content creator for the SANS Cloud Security Curriculum.

Today’s episode is titled: “Demonstration of Terraform Modules; Deploy a VM in AWS, Azure and GCP at once”

The purpose of HITC is to teach foundational cloud skills and security knowledge that will help others thrive in the cloud. The content ideas come from my personal observation of skills that I see some students lacking when they show up to a SANS Cloud Security course. Other ideas are passed on from fellow SANS instructors.

The idea for today's content came from teaching [SANS SEC510: Public Cloud Security: AWS, Azure, and GCP](https://www.sans.org/cyber-security-courses/public-cloud-security-aws-azure-gcp/) where we use various Terraform scripts to deploy assets to AWS, Azure, and GCP throughout the week. While not strictly necessary, an understanding of Terraform modules will help students taking that course. Furthermore, since Terraform is a leading Infrastructure-as-Code (IoC) tool, understanding how modules can be used to abstract infrastructure details is important knowledge for a cloud security professional to possess.

## Overview

First, take a look at the repository in Github - [https://github.com/Resistor52/tf-vm_in3csps](https://github.com/Resistor52/tf-vm_in3csps). The main README describes how to deploy a VM in all three CSPs at once. We will come back to that in the second part after we deploy each module individually.

Next, let's take a look at the README for each module. Each module gives the specifics of how to set up the authentication for each of the three cloud service providers. In this episode, I will not be dwelling on that configuration because each module provides the relevant document reference in the README and I have covered this in other HitC episodes.

## Set Up

To follow along with this episode, start with a new Ubuntu 20.04 system. For example, you could use a virtual machine on AWS, GCP, or VMWare. It really doesn't matter, but I recommend that you start with a clean system to avoid issues.

### Set up GCP

We are going to start with Google Cloud Platform. We will be setting this up following the steps from [HitC Episode 4](https://headintheclouds.site/episodes/episode4). The steps are summarized here for convenience, but refer to that episode for details and a video demonstration.

1. Generate a new SSH Key Pair. Run `ssh-keygen` and accept the defaults. This key pair will be used by all three VMs.
2. Log into the GCP Cloud Console and create a new project. Use a name similar to "tf-gcp-" plus six random digits so that the project name is globally unique.
3. Create a service account named "terraform" and attach the "Compute Admin" role to the service account. Click "Done." Next, click "Manage Keys" under the Actions heading. Then click "ADD KEY" and select "Create new key." Select "JSON" as the format and click "CREATE." This will generate a JSON file that grants the holder access to the project, so it must be carefully protected.
4. Move a copy of this key onto the Ubuntu system, placing it in a directory called "accesskey" and renaming the file "service_account.json"
5. Since this is a new project, we will need to enable the GCE service via the web console. Do that now.

Refer to the module README for more information.

### Set up AWS

1. First let's install the AWS CLI. Always remember that the website [headintheclouds.site](https://headintheclouds.site) contains the show notes for every episode, including the commands so that you can cut and paste them.

  ```
  sudo curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "/tmp/awscliv2.zip"
  sudo unzip /tmp/awscliv2.zip -d /tmp
  sudo /tmp/aws/install
  ```

2. Next, in the AWS Account that will host your EC2 Instance, create an IAM User account with administrator access and generate an AWS Access Key.
3. Run `aws configure` to configure the CLI with the AWS Access Key that was created in the previous step.

Refer to the module README for more information.

### Set up Azure

1. Install the Azure CLI:

  ```
  curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
  ```

2. Lastly, authenticate with Microsoft Azure using `az login`

Refer to the module README for more information.

### Install Terraform

Run the following commands to download and install terraform. Remember to copy and paste
these from the show notes:

```
sudo apt install -y unzip
wget https://releases.hashicorp.com/terraform/1.0.4/terraform_1.0.4_linux_amd64.zip -O /tmp/terraform.zip
sudo unzip /tmp/terraform.zip -d /usr/local/bin/
rm /tmp/terraform.zip
```

confirm the version and execution:

```
terraform --version
```

You should see the following:

```
ubuntu@unbuntu:~$ terraform --version
Terraform v1.0.4
on linux_amd64
```

## Try Each Module as Stand-alone Code

Ok, now that we have the authentication configured for each CSP, we will git clone our
repository to our Ubuntu system and look at the root module:

```
cd ~
git clone https://github.com/Resistor52/tf-vm_in3csps.git
cd tf-vm_in3csps/
ls
```

You should see:

```
README.md  main.tf  modules  output.tf  terraform.tfvars.example  variables.tf
```

Note that `modules` directory contains our nested modules. Take a look:

```
cd modules
ls
```

Shows:

```
aws  azure  gcp
```

Great, now we will play with each module in turn.

### AWS Module

Change into the `aws` module:

```
cd ~/tf-vm_in3csps/modules/aws
ls
```

This shows:

```
README.md  keyPair.tf  main.tf  network.tf  output.tf  terraform.tfvars.example  variables.tf  vm.tf
```

Before we can deploy this VM, we need to set the variables. Copy `terraform.tfvars.example` using the following command:

```
cp terraform.tfvars.example terraform.tfvars
```

And then edit it as appropriate for your AWS configuration.

Because I accepted the defaults when I generated the SSH key pair and am deploying to us-east-1 with the
default profile, I did not need to make any modifications. Here is what the default `terraform.tfvars` file looks like:

```
aws_region = "us-east-1"
aws_creds_file = "~/.aws/credentials"
aws_profile = "default"
ssh_pub_key_file = "~/.ssh/id_rsa.pub"
ssh_priv_key_file = "~/.ssh/id_rsa"
````

**NOTE:** We should still be in the `tf-vm_in3csps/modules/aws` directory.

Time to deploy the AWS VM. Run the following commands:

```
terraform init
terraform apply
```

Type 'yes' when prompted. Eventually, we will see something similar to:

```
Apply complete! Resources: 10 added, 0 changed, 0 destroyed.

Outputs:

ip = "54.159.93.238"
ssh_connection_string = "ssh -i ~/.ssh/id_rsa ubuntu@54.159.93.238"
```

Feel free to SSH into the new instance from the Ubuntu system using the string inside the
quotation marks. Note that the terraform script restricted the ingress to just SSH from
just the public IP address of the system running the script.

Since that worked, lets tear it down:

```
terraform destroy
```

Type 'yes' when prompted. Eventually, we will see:

```
Destroy complete! Resources: 10 destroyed.
```

### Azure Module

Change into the `azure` module:

```
cd ~/tf-vm_in3csps/modules/azure
ls
```

This shows:

```
README.md  main.tf  network.tf  output.tf  resource_group.tf  terraform.tfvars.example  variables.tf  vm.tf
```

As with the AWS module, we need to set the variables before we can deploy the Azure VM.
Copy `terraform.tfvars.example` using the following command:

```
cp terraform.tfvars.example terraform.tfvars
```

And then edit it as appropriate for your Azure configuration.

Because I accepted the defaults when I generated the SSH key pair and am deploying to the "Central US"
region, I did not need to make any modifications. Here is what the default `terraform.tfvars` file
looks like for Azure:

```
ssh_user ="ubuntu"
azure_location = "Central US"
ssh_pub_key_file = "~/.ssh/id_rsa.pub"
ssh_priv_key_file = "~/.ssh/id_rsa"
````

**NOTE:** We should still be in the `tf-vm_in3csps/modules/azure` directory.

To deploy the Azure VM. Run the following commands:

```
terraform init
terraform apply
```

Type 'yes' when prompted. Eventually, we will see something like:

```
Apply complete! Resources: 8 added, 0 changed, 0 destroyed.

Outputs:

ip = "40.122.135.106"
ssh_connection_string = "ssh -i ~/.ssh/id_rsa ubuntu@40.122.135.106"
```

Cool, that worked, Now we can tear it down:

```
terraform destroy
```

Type 'yes' when prompted. Eventually, we will see:

```
Destroy complete! Resources: 8 destroyed.
```

### GCP Module

Lastly, let's test the GCP Module. Change into the `gcp` module:

```
cd ~/tf-vm_in3csps/modules/gcp
ls
```

This shows:

```
README.md  ip_address.tf  main.tf  network.tf  output.tf  terraform.tfvars.example  variables.tf  vm.tf
```

As before, we need to set the variables before we can deploy the GCP VM.
Copy `terraform.tfvars.example` file:

```
cp terraform.tfvars.example terraform.tfvars
```

And then edit it as appropriate for your GCP configuration.

Here is what the default `terraform.tfvars` file looks like for GCP:

```
ssh_user ="ubuntu"
gcp_project = "PROJECT_ID"
gcp_region  = "us-central1"
gcp_zone    = "us-central1-a"
gcp_key_file = "~/accesskey/service_account.json"
ssh_pub_key_file = "~/.ssh/id_rsa.pub"
ssh_priv_key_file = "~/.ssh/id_rsa"
````

Even if you are ok with the default GCP region and zone, you will need to replace the
_PROJECT_ID_ with the project id for the project you created earlier in the set up steps.

Like last time, run the following commands and type 'yes' when prompted:

```
terraform init
terraform apply
```

After it completes, we will see something like:

```
Apply complete! Resources: 4 added, 0 changed, 0 destroyed.

Outputs:

ip = "35.193.238.218"
ssh_connection_string = "ssh -i ~/.ssh/id_rsa ubuntu@35.193.238.218"
```

Looks good, go ahead and tear it down:

```
terraform destroy
```

## Deploy Virtual Machines to All Three Clouds  

Now that we see how each module can work as stand-alone Infrastructure-as-Code, lets
try deploying the entire project.

Change into the directory for the root module:

```
cd ~/tf-vm_in3csps
```

First, lets clean up the Terraform state files, the *.tfvars files for the modules, and the
hidden terraform directory in each module:

```
rm ~/tf-vm_in3csps/modules/*/terraform.tf*
rm -rf ~/tf-vm_in3csps/modules/*/.terraf*
```

We also need to set up the `terraform.tfvars` file for the root module.

```
cp terraform.tfvars.example terraform.tfvars
```

Here is the default configuration:

```
# AWS Variables
aws_region = "us-east-1"
aws_creds_file = "~/.aws/credentials"
aws_profile = "default"

# Azure Variables
azure_location = "Central US"

# GCP Variables
gcp_project = "PROJECT_ID"
gcp_region  = "us-central1"
gcp_zone    = "us-central1-a"
gcp_key_file = "~/accesskey/service_account.json"

# Common Variables
ssh_user ="ubuntu"
ssh_pub_key_file = "~/.ssh/id_rsa.pub"
ssh_priv_key_file = "~/.ssh/id_rsa"
```

Be sure to modify the _PROJECT_ID_ and any other parameters as appropriate for your cloud environments.

Having done that, we can deploy the complete infrastructure:

```
terraform init
terraform apply
```

Very cool! You should see:

```
Apply complete! Resources: 22 added, 0 changed, 0 destroyed.

Outputs:

aws_ip = "107.20.14.112"
aws_ssh_connection_string = "ssh -i ~/.ssh/id_rsa ubuntu@107.20.14.112"
az_ip = "52.165.170.251"
az_ssh_connection_string = "ssh -i ~/.ssh/id_rsa ubuntu@52.165.170.251"
gcp_ip = "35.193.238.218"
gcp_ssh_connection_string = "ssh -i ~/.ssh/id_rsa ubuntu@35.193.238.218"
```

We now have a virtual machine in each of the big three clouds!

Feel free to SSH into each, if you like. When you are done, destroy the deployment.

```
terraform destroy
```

## Examine the Code

The first thing that you may have noticed is that the outputs of the root module is different
than the outputs of each of the nested modules. The main reason is to differentiate the outputs from
each of the modules. Note that the root module simply references the output of the nested module. While
you are at it, compare the differences in each of the nested modules as to how each provider accesses
the public IP attribute.

### The AWS module output.tf

```
output "ip" {
  description = "The public IP address of the virtual machine"
  value = aws_instance.vm_instance.public_ip
}

output "ssh_connection_string" {
  description = "The SSH command to connect to the virtual machine"
  value = "ssh -i ${var.ssh_priv_key_file} ubuntu@${aws_instance.vm_instance.public_ip}"
}
```

### The Azure module output.tf

```
output "ip" {
  description = "The public IP address of the virtual machine"
  value = data.azurerm_public_ip.vm.ip_address
}

output "ssh_connection_string" {
  description = "The SSH command to connect to the virtual machine"
  value = "ssh -i ${var.ssh_priv_key_file} ${var.ssh_user}@${data.azurerm_public_ip.vm.ip_address}"
}
```

### The GCP module output.tf

```
output "ip" {
  description = "The public IP address of the virtual machine"
  value = google_compute_instance.vm_instance.network_interface.0.access_config.0.nat_ip
}

output "ssh_connection_string" {
  description = "The SSH command to connect to the virtual machine"
  value = "ssh -i ${var.ssh_priv_key_file} ${var.ssh_user}@${google_compute_instance.vm_instance.network_interface.0.access_config.0.nat_ip}"
}
```

### The root module output.tf

```
# AWS Outputs
output "aws_ip" {
  description = "The public IP address of the AWS virtual machine"
  value = module.aws.ip
}

output "aws_ssh_connection_string" {
  description = "The SSH command to connect to the AWS virtual machine"
  value = module.aws.ssh_connection_string
}

# Azure Outputs
output "az_ip" {
  description = "The public IP address of the Azure virtual machine"
  value = module.azure.ip
}

output "az_ssh_connection_string" {
  description = "The SSH command to connect to the Azure virtual machine"
  value = module.azure.ssh_connection_string
}

# GCP Outputs
output "gcp_ip" {
  description = "The public IP address of the GCP virtual machine"
  value = module.gcp.ip
}

output "gcp_ssh_connection_string" {
  description = "The SSH command to connect to the GCP virtual machine"
  value = module.gcp.ssh_connection_string
}
```

### The nested module *.tf files

Take a look at the output of the `tree` command from the home directory.

```
.
├── accesskey
│   └── service_account.json
└── tf-vm_in3csps
    ├── README.md
    ├── main.tf
    ├── modules
    │   ├── aws
    │   │   ├── README.md
    │   │   ├── keyPair.tf
    │   │   ├── main.tf
    │   │   ├── network.tf
    │   │   ├── output.tf
    │   │   ├── variables.tf
    │   │   └── vm.tf
    │   ├── azure
    │   │   ├── README.md
    │   │   ├── main.tf
    │   │   ├── network.tf
    │   │   ├── output.tf
    │   │   ├── resource_group.tf
    │   │   ├── variables.tf
    │   │   └── vm.tf
    │   └── gcp
    │       ├── README.md
    │       ├── ip_address.tf
    │       ├── main.tf
    │       ├── network.tf
    │       ├── output.tf
    │       ├── variables.tf
    │       └── vm.tf
    ├── output.tf
    ├── terraform.tfstate
    ├── terraform.tfstate.backup
    ├── terraform.tfvars
    ├── terraform.tfvars.example
    └── variables.tf

6 directories, 30 files
```

Each of the modules could be implemented in a single main.tf module, but Hashicorp
recommends that each module have an output.tf and a variables.tf file as well as other *.tf
files to compartmentalize the infrastructure and make it more readable. See
[Standard Module Structure](https://www.terraform.io/docs/language/modules/develop/structure.html).
The document also recommends that the modules be located in a directory called "modules" but in practice
it seems that many developers ignore that advice. Each module is supposed to have a README to improve
portability and code reuse. Similarly, each variable and output is to have a description so that the code
is self-documenting.

Take the time to explore each of these files in the [code repository](https://github.com/Resistor52/tf-vm_in3csps).

### The root module main.tf

Lastly, check out the [main.tf of the root module](https://github.com/Resistor52/tf-vm_in3csps/blob/main/main.tf):

```
module "aws" {
  aws_region = var.aws_region
  aws_creds_file = var.aws_creds_file
  aws_profile = var.aws_profile
  ssh_pub_key_file = var.ssh_pub_key_file
  ssh_priv_key_file = var.ssh_priv_key_file
  source = "./modules/aws"
}

module "azure" {
  azure_location = var.azure_location
  ssh_user = var.ssh_user
  ssh_pub_key_file = var.ssh_pub_key_file
  ssh_priv_key_file = var.ssh_priv_key_file
  source = "./modules/azure"
}

module "gcp" {
  gcp_project = var.gcp_project
  gcp_region = var.gcp_region
  gcp_zone = var.gcp_zone
  gcp_key_file = var.gcp_key_file
  ssh_user = var.ssh_user
  ssh_pub_key_file = var.ssh_pub_key_file
  ssh_priv_key_file = var.ssh_priv_key_file
  source = "./modules/gcp"
}
```

These module blocks are passing in the variables defined in the root module `variables.tf` file
and set in the root module `terraform.tfvars` file to the module variables that coincidently
(but not necessarily) have the same name.

Notice that the module blocks each contain a "source" statement that contains the relative path
to the code for that module.

## Wrap Up

Well, there you go. Hopefully this episode has demystified Terraform modules. Each module is a self-contained
deployable chunk of infrastructure code. We can nest these modules under a root module to deploy a larger infrastructure. The [Hashicorp Terraform documentation on Modules](https://www.terraform.io/docs/language/modules/index.html) provides two important recommendations:

* Abstract infrastructure into modules only when it makes sense and improves manageability and readability.
* Use a flat hierarchy; Avoid nesting modules under nested modules.

If you have thoughts or comments on today’s episode, feel free to chime in on the comments for this YouTube video.

If you appreciate this video and want to see more like it, be sure to give it a "thumbs up."

Stay tuned for another installment of “Head in the Clouds” as announcements of new episodes are made on the SANS Cloud Security Twitter feed.

Meanwhile, be sure to check out the other great videos on the [SANS Cloud Security YouTube Channel](https://www.youtube.com/c/SANSCloudSecurity).

Take care.
