# Optimizing Infrastructure
On the following task we will start refinement of our script to use data driven approach and to re-use code with Terraform modules.

### TASK 9
Learn about [terraform data sources](https://www.terraform.io/docs/language/data-sources/index.html) and [querying terraform data sources](https://learn.hashicorp.com/tutorials/terraform/data-sources?in=terraform/configuration-language&utm_source=WEBSITE&utm_medium=WEB_BLOG&utm_offer=ARTICLE_PAGE).

Change current directory to `~/tf_aws_lab/base`
Refine your configuration :

- Use data source to request Avalability zones for us-east-1 region and assign your vpc with appropriate AZs. Hint: select such AZ numbers which will not initiate resource recreation and has already assigned to your VPC.
Store all resources from this task in `data.tf` file.

Run `terraform validate`  and `terraform fmt` to check if your configuration valid and fits to a canonical format and style. Do this each time before apply your changes.
Run `terraform plan` to see your changes.

Apply your changes when you're ready.


Change current directory to   `~/tf_aws_lab/compute`

Refine your configuration :

- Use data source to request the following resources: vpc_id, private subnet id, public subnet id, security group id, ssh key name, iam instance profile name, s3 bucket name.
These data sources should replace remote state outputs therefore you can delete `data "terraform_remote_state" "base"` resource from current state and the `outputs.tf` file from the `base` configuration.
Hint: run `terraform refresh` command under `base` configuration to reflect changes.

Store all resources from this task in `data.tf` file.

Run `terraform validate`  and `terraform fmt` to check if your configuration valid and fits to a canonical format and style. Do this each time before apply your changes.
Run `terraform plan` to see your changes.

Apply your changes when you're ready.


### TASK 10

Learn about [terraform modules](https://www.terraform.io/docs/language/modules/develop/index.html)

Refine your configurations:

- Refine `base` configuration by a creating module for VPC related resources: vpc, subnets, routes, internet gateways.
- Refine `base` configuration by a creating module for security group related resources.
- Refine `base` configuration by a creating module for IAM role related resources.
- [Optional] Refine `compute` configuration by a creating Autoscaling group module.


Store modules in `~/tf_aws_lab/modules/` subfolders.

Run `terraform validate`  and `terraform fmt` to check if your modules valid and fits to a canonical format and style.
Run `terraform plan` to see your changes and re-apply your changes if it needed.
