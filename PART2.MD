# Optimizing Infrastructure
On the following task we will start refinement of our script to use data driven approach and to re-use code with Terraform modules.

### TASK 9

Learn about [terraform backend in AWS S3](https://www.terraform.io/docs/language/settings/backends/s3.html)

Refine your configurations:

- Refine `base` configuration by a moving local state to a s3.
- Refine `base` configuration by adding locking with DynamoDB.
- Refine `compute` configuration by a moving local state to a s3.
- Refine `compute` configuration by adding locking with DynamoDB.

Do not forget to change path to a remote state for `compute` configuration.
Store modules in `~/tf_aws_lab/modules/` subfolders.

Run `terraform validate`  and `terraform fmt` to check if your modules valid and fits to a canonical format and style.
Run `terraform plan` to see your changes and re-apply your changes if it needed.

### TASK 10

Learn about [terraform state mv](https://www.terraform.io/docs/cli/commands/state/mv.html) command

You are going to move previously created resource(IAM group) between states.
Hint: Keep in mind that there are 3 instances: AWS resource, Terraform state file which store some state of that resource, and Terraform configuration which describe resource. This meaning "move" resource is move it between states. Moreover to make it works you should delete resource from source configuration and add it to destination configuration(this action is not automated).

- Move `test-move` IAM group from the `base` state to the `compute` using `terraform state mv` command.
- Update both configuration according to this move.
- Run `terraform plan` on both configurations and observe the changes. Hint: there are should not be no changes detected (No resource creation or deletion in case of the correct resource move)

Run `terraform validate`  and `terraform fmt` to check if your modules valid and fits to a canonical format and style.

### TASK 11

Learn about [terraform import](https://www.terraform.io/docs/cli/import/index.html) command

You are going to import new resource(IAM group) to your state.
Hint: Keep in mind that there are 3 instances: AWS resource, Terraform state file which store some state of that resource, and Terraform configuration which describe resource. This meaning to "import" resource you should import resource attributes to Terraform state. Then you have to add resource to destination configuration(this action is not automated).

- Create IAM group(name="test-import").
- Add new resource aws_iam_group "test-import" to the `compute` configuration.
- Run `terraform plan` to see your changes but do not apply changes.
- Import `test-import` IAM group to the `compute` state.
- Run `terraform plan` again to ensure that import was successful.

Run `terraform validate`  and `terraform fmt` to check if your modules valid and fits to a canonical format and style.
If applicable all resources should be tagged with following tags {Terraform=true, Project=epam-tf-aws-lab}.
If applicable all resources should be defined with the provider alias.

### TASK 12
Learn about [terraform data sources](https://www.terraform.io/docs/language/data-sources/index.html) and [querying terraform data sources](https://learn.hashicorp.com/tutorials/terraform/data-sources?in=terraform/configuration-language&utm_source=WEBSITE&utm_medium=WEB_BLOG&utm_offer=ARTICLE_PAGE).

#### base configuration 
Change current directory to `~/tf_aws_lab/base`
Refine your configuration :

- Use data source to request Availability zones for us-east-1 region and assign your vpc with appropriate AZs. Hint: select such AZ numbers which will not initiate resource recreation and has already assigned to your VPC.

Store all resources from this task in `data.tf` file.

Run `terraform validate`  and `terraform fmt` to check if your configuration valid and fits to a canonical format and style. Do this each time before apply your changes.
Run `terraform plan` to see your changes. Also you can use `terraform refresh`.
If applicable all resources should be defined with the provider alias.

Apply your changes when you're ready.

#### compute configuration 
Change current directory to   `~/tf_aws_lab/compute`

Refine your configuration :

- Use data source to request the following resources: vpc_id, private subnet id, public subnet id, security group id, ssh key name, iam instance profile name, s3 bucket name.

These data sources should replace remote state outputs therefore you can delete `data "terraform_remote_state" "base"` resource from current state and the `outputs.tf` file from the `base` configuration.
Hint: run `terraform refresh` command under `base` configuration to reflect changes.

Store all resources from this task in `data.tf` file.

Run `terraform validate`  and `terraform fmt` to check if your configuration valid and fits to a canonical format and style. Do this each time before apply your changes.
Run `terraform plan` to see your changes. Also you can use `terraform refresh`.
If applicable all resources should be defined with the provider alias.

Apply your changes when you're ready.

### TASK 13

Learn about [terraform modules](https://www.terraform.io/docs/language/modules/develop/index.html)

Refine your configurations:

- Refine `base` configuration by a creating module for VPC related resources: vpc, subnets, routes, internet gateways.
- Refine `base` configuration by a creating module for security group related resources.
- Refine `base` configuration by a creating module for IAM role related resources.
- [Optional] Refine `compute` configuration by a creating Autoscaling group module.


Store modules in `~/tf_aws_lab/modules/` subfolders.

Run `terraform validate`  and `terraform fmt` to check if your modules valid and fits to a canonical format and style.
Run `terraform plan` to see your changes and re-apply your changes if it needed.
