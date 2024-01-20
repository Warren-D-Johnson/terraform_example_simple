# terraform_example_simple

### Simple Terraform Example

Updated 10/23/2023

This simple Terraform configuration example demonstrates how to automate some basic setup steps in a new AWS Account.  We demonstrate the following:

* Creating billing alerts in Cloudwatch.
* Creating SNS topics triggered by the billing alerts.
* Creating an admininistrative user with password and MFA.
* Creating a policy to purge any Cloudfront distribution in the system.

*For a more complex example of a full Terraform configuration, including AWS and Cloudflare services, please reference the terraform_example_advanced.*


Dependencies:

* terraform installed
* "pwgen" package installed
* AWS CLI V2 installed

Security Notes:

* There are a few folders I purposely didn't exclude from this repository (such as mfacodes) for teaching purposes.  You should ignore that folder as well as any tfstate files that may include sensitive information.
* Some like to gitignore *.tfvars as well but if you don't put anything sensitive in there, its nice to have them for reference.

For a more complex example of a full Terraform configuration, including AWS and Cloudflare services, please reference the terraform_example_advanced.

**Pre-Amble**

When I create a standlone AWS account, I like to launch an EC2 instance,  create a user "terraform", lock it down with Security Group rules and then use it for my Terraform and Puppet management.  I create an IAM role with full adminstrative privileges and assign it to the EC2.  When the EC2 is not in-use, it is shut down.

Terraform will install provider dependencies all over your code increasing its size and causing some issues with git and versioning.  To handle that, I use the Terraform filesystem mirror plugin.  This lets you centralize your provider dependencies in your ~/.terraform.d.

*The versions below were current as of this writing, you may have to visit the Hashicorp website and find new ones.*

**As terraform user:**

cd /home/terraform

Download and install a centralized AWS Terraform Provider:
mkdir -p /home/terraform/.terraform.d/plugins/registry.terraform.io/hashicorp/aws/5.22.0/linux_amd64
wget -O /home/terraform/terraform-provider-aws_5.22.0_linux_amd64.zip https://releases.hashicorp.com/terraform-provider-aws/5.22.0/terraform-provider-aws_5.22.0_linux_amd64.zip
unzip /home/terraform/terraform-provider-aws_5.22.0_linux_amd64.zip -d/home/terraform/.terraform.d/plugins/registry.terraform.io/hashicorp/aws/5.22.0/linux_amd64

Download and install a centralized "null" Terraform Provider (used for some configuration convenience later on):

mkdir -p /home/terraform/.terraform.d/plugins/registry.terraform.io/hashicorp/null/3.2.1/linux_amd64
wget -O /home/terraform/terraform-provider-null_3.2.1_linux_amd64.zip https://releases.hashicorp.com/terraform-provider-null/3.2.1/terraform-provider-null_3.2.1_linux_amd64.zip
unzip /home/terraform/terraform-provider-null_3.2.1_linux_amd64.zip -d/home/terraform/.terraform.d/plugins/registry.terraform.io/hashicorp/null/3.2.1/linux_amd64

Download and install a centralized external Terraform Provider (used for some configuration convenience later on):
mkdir -p /home/terraform/.terraform.d/plugins/registry.terraform.io/hashicorp/external/2.3.1/linux_amd64
wget -O /home/terraform/terraform-provider-external_2.3.1_linux_amd64.zip https://releases.hashicorp.com/terraform-provider-external/2.3.1/terraform-provider-external_2.3.1_linux_amd64.zip
unzip terraform-provider-external_2.3.1_linux_amd64.zip -d /home/terraform/.terraform.d/plugins/registry.terraform.io/hashicorp/external/2.3.1/linux_amd64

rm /home/terraform/terraform-provider-aws_5.22.0_linux_amd64.zip /home/terraform/terraform-provider-cloudflare_3.32.0_linux_amd64.zip /home/terraform/terraform-provider-null_3.2.1_linux_amd64.zip terraform-provider-external_2.3.1_linux_amd64.zip

Let Terraform know we already have a centralized copy of all of the providers
echo -e "provider_installation {\n  filesystem_mirror {\n	path	= \"/home/terraform/.terraform.d/plugins\"\n	include = [\"registry.terraform.io/hashicorp/*\"]\n  }\n  direct {\n	exclude = [\"hashicorp/*\"]\n  }\n}" > /home/terraform/.terraformrc

*Remember that passwords and other sensitive information may be in your state file.*

Thanks!