# Automated deployment of virtual machines and AppDynamics Machine Agent
 This repo contains and example of deploying a virtual machine in vSphere using Terraform. After Terraform provisions the virtual machines, a provisioner within Terraform calls the Ansible playbook which installs Apache, Docker, and the AppDynamics Machine Agent.

## Requirements

Here is a list of dependencies to make this work in your environment:

- vSphere 6.7 or higher
- Terraform v0.15.2 or higher
- Ansible 2.10.2
- AppDynamics controller instance along with its credentials
- A virtual machine template with CentOS 8.2 installed as the guest OS

## Credentials

We purposely did not add credentials and other sensitive information to the repo by including them in the `.gitignore` file. As such, if you clone this repo, you will need to create two files. The first file named `secret.tfvars` contains sensitive Terraform variables. The second file named `variables.yml` is used by Ansible. In this scenario, we encrypted `variables.yml` using the command `ansible-vault` command and decrypt it as needed locally. You could take the same approach or leave the file unencrypted if you are confident it won't be shared or inadvertently uploaded to a repo.

Here is a list of variables you'll need to include and define for each file

- `secret.tfvars` in HCL format (file is in the same directory as the `terraform.tfvars` file):
  - vsphere_user
  - vsphere_password
  - vsphere_server (the IP address or FQDN)
  - vsphere_vm_firmware (default is `vsphere_vm_firmware = bios`)
  - ssh-pub-key (an SSH key used with a service account that allows Ansible to connect over SSH)
  - service_account_username
  - service_account_password
- `variables.yml` (written in YAML) file:
  - CONTROLLER_HOST (the URI of the AppDynamics Controller)
  - CONTROLLER_PORT (typically 443)
  - ACCOUNT_NAME (AppDynamics Account Name)
  - MACHINE_PATH (This is a heirarchy separated with a |. For example: San Jose|Rack1|)
  - ACCOUNT_ACCESS_KEY (this value is available in the AppDynamics Controller)
  - APPD_BEARER_TOKEN (this is the token derived from the available image download via cURL)


## What Terraform Provisions

In this example, Terraform uses the `vsphere` provider and a `vsphere_virtual_machine` resource to:

- Create 5 virtual machines from virtual machine template
- Add the SSH key of a service account to each host
- Run an Ansible playbook that performs the steps in the next section

## What Ansible Installs and Configures

After Terraform creates 5 virtual machines, the Ansible playbook installs and configures:

- Apache Web Server
- Firewall with port 80 opened
- Docker
- DNS (resolv.conf is configured)
- AppDynamics Machine Agent

Each Apache Web Server is configured with a custom (using a Jinja template) `index.html` page that displays the hostname.

The same approach is taken with the AppDynamics Machine Agent. In other words, a Jinja template creates a custom file for each containing the hostname.

## Creating and Applying the Terraform Plan

Here are the steps needed to run Terraform along with examples of each:

1. Initialize Terraform:
    
`terraform init  -var-file="secret.tfvars"`

2. Create a Terraform Plan:

`terraform plan -out appd-machine-agent-installation.tfplan -var-file="secret.tfvars"`

3. Apply the Terraform Plan:

`terraform apply -var-file="secret.tfvars"`

Each of these commands includes the `secret.tfvars` containing the sensitive variables needed to connect to the different resources as described in the previous section.

## Results

You will see 5 virtual machines created with static IP addresses in vSphere, each running an Apache Web Server, and all 5 appearing in the AppDynamics controller. 