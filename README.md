  * [Problem to Be Solved](#problem-to-be-solved)
  * [Explanation of the Solution](#explanation-of-the-solution)
  * [PRE-REQUISITES](#pre-requisites)
- [Creating Infrastructure](#creating-infrastructure)
  * [TASK 1 - Creating VPC](#task-1---creating-vpc)
  * [TASK 2 - Import Your SSH Key into AWS](#task-2---import-your-ssh-key-into-aws)
  * [TASK 3 - Create S3 Bucket](#task-3---create-s3-bucket)
  * [TASK 4 - Create IAM Resources](#task-4---create-iam-resources)
  * [TASK 5 - Create Security Group](#task-5---create-security-group)
  * [TASK 6 - Form TF Output](#task-6---form-tf-output)
  * [TASK 7 - Configure remote data source](#task-7---configure-remote-data-source)
  * [TASK 8 - Create EC2/ASG/ELB](#task-8---create-ec2-asg-elb)
- [Working with state](#working-with-state)
  * [TASK 9 - Move state to S3/Locking](#task-9---move-state-to-s3-locking)
  * [TASK 10 - Move resources](#task-10---move-resources)
  * [TASK 11 - Import resources](#task-11---import-resources)
  * [TASK 12 - Use data discovery](#task-12---use-data-discovery)
- [Advanced tasks](#advanced-tasks)
  * [TASK 13 - Expose node output with nginx](#task-13---expose-node-output-with-nginx)
  * [TASK 14 - Modules](#task-14---modules)

### Problem to Be Solved 
 This lab shows you how to use Terraform to create infrastructure in AWS including auto-scaling group, VPC, subnets, security groups and IAM role. Each instance will report its data to a specified S3 bucket on startup. This task is binding to real production needs – for instance developers could request instances with ability to writing debug information to S3 bucket.

 
### Explanation of the Solution 
You will use Terraform with AWS provider to create 2 separate Terraform configurations:
 1) Base configuration
 2) Compute configuration
After you’ve created configuration, we will work on its optimization like using data driven approach and creating modules.


## PRE-REQUISITES
1. Learn about [terraform state](https://www.terraform.io/docs/language/state/index.html).

2. Create a folder with 2 subfolders: `base` and `compute`. E.g. `mkdir -p ~/tf_aws_lab/{base,compute}`

3. All actions should be done under those subfolder as Terraform gets it context from current working directory: 
    - Change current directory  to `~/tf_aws_lab/base` folder and create `root.tf` file. 
    - Add `terraform {}`empty block to this file. Create AWS provider block inside `root.tf` file with the following attributes: 
        - `region = "us-east-1"`
        - `alias = "use1"`
        - `shared_credentials_file = "~/.aws/credentials"`.

**Hint**: Add your AWS credentials to   `~/.aws/credentials` file. See [this](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html) document for details

Run `terraform init` to initialize your configuration. 
Run `terraform validate`  and `terraform fmt` to check if your configuration valid and fits to a canonical format and style. Do this each time before apply your changes.
Run `terraform plan` to ensure that there are no changes.

Please use **underscore** Terraform resources naming, e.g. `my_resource` instead of `my-resource`.

4. Change current directory  to `~/tf_aws_lab/compute` and repeat the steps in [3].
5. Consider use Terraform [best practises](https://www.terraform-best-practices.com/naming)

You are ready for lab!

# Creating Infrastructure

## TASK 1 - Creating VPC
Change current directory  to `~/tf_aws_lab/base`

Create network stack for your infrastructure:

-	**VPC**: `name={StudentName}-{StudentSurname}-01-vpc`, `cidr=10.10.0.0/16`
-	**Public subnets**:
    - `name={StudentName}-{StudentSurname}-01-subnet-public-a`, `cidr=10.10.1.0/24`, `az=a`)
    - `name={StudentName}-{StudentSurname}-01-subnet-public-b`, `cidr=10.10.3.0/24`, `az=b`)
    - `name={StudentName}-{StudentSurname}-01-subnet-public-c`, `cidr=10.10.5.0/24`, `az=c`)
-	**Internet gateway**: `{StudentName}-{StudentSurname}-01-igw`
-	**Routing table to bind IGW with Public subnets**: `name={StudentName}-{StudentSurname}-01-rt`

Store all resources from this task in `vpc.tf` file.

Equip all resources with following tags:
    - `Terraform=true`, 
    - `Project=epam-tf-aws-lab`
    - `Owner={StudentName}_{StudentSurname}`

Run `terraform validate`  and `terraform fmt` to check if your configuration valid and fits to a canonical format and style. Do this each time before applying your changes.
Run `terraform plan` to see your changes.

Apply your changes when you're ready.

### Definition of DONE:

- Terraform created infrastructure with no errors
- AWS Resources Created as expected (check AWS Console)
- Save following artifacts under `/reports/task1/` folder:
    - `terraform.tfstate` file
    - `terraform apply` log (`tf_apply.log`)
    - `terraform plan` (after changes) log (`tf_plan_after.log`)

## TASK 2 - Import Your SSH Key into AWS

Ensure that current directory is `~/tf_aws_lab/base`

Create custom ssh key-pair to access your ec2 instances:

-	Create your ssh key pair [refer to this document](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html#how-to-generate-your-own-key-and-import-it-to-aws)
-	Create `variables.tf` file with empty variable "ssh_key" but with the following description "Provides custom public ssh key". Never store you secrets inside the code!
-	Create `key_pair.tf` file with `aws_key_pair` resource. Use ssh_key variable as a public key source.
- Run `terraform plan` and provide required public key. Observe the output and run `terraform plan` again.
- To prevent providing ssh key on each configuration run but stay secure - set binding environment variable - `export TF_VAR_ssh_key="YOUR_PUBLIC_SSH_KEY_STRING"`
- Run `terraform plan` and observe the output.


Equip all resources with following tags:
    - `Terraform=true`, 
    - `Project=epam-tf-aws-lab`
    - `Owner={StudentName}_{StudentSurname}`


Run `terraform validate` and `terraform fmt` to check if your configuration valid and fits to a canonical format and style. Do this each time before apply your changes.

Apply your changes when you're ready.

### Definition of DONE:

- Terraform created infrastructure with no errors
- AWS Resources Created as expected (check AWS Console)
- Save following artifacts under `/reports/task2/` folder:
    - `terraform.tfstate` file
    - `terraform apply` log (`tf_apply.log`)
    - `terraform plan` (after changes) log (`tf_plan_after.log`)

## TASK 3 - Create S3 Bucket

Ensure that current directory is  `~/tf_aws_lab/base`

Create S3 bucket as the storage for your infrastructure:

-	Create `s3.tf`. Name your bucket "epam-aws-tf-lab-${random_string.my_numbers.result}" to provide it with partition unique name. See [random_string](https://registry.terraform.io/providers/hashicorp/random/latest/docs/resources/string) documentation for details
-	Set bucket acl as private. Never share your bucket to a world!

Equip all resources with following tags:
    - `Terraform=true`, 
    - `Project=epam-tf-aws-lab`
    - `Owner={StudentName}_{StudentSurname}`

Run `terraform validate`  and `terraform fmt` to check if your configuration valid and fits to a canonical format and style. Do this each time before apply your changes.
Run `terraform plan` to see your changes.

Apply your changes when you're ready.

### Definition of DONE:

- Terraform created infrastructure with no errors
- AWS Resources Created as expected (check AWS Console)
- Save following artifacts under `/reports/task3/` folder:
    - `terraform.tfstate` file
    - `terraform apply` log (`tf_apply.log`)
    - `terraform plan` (after changes) log (`tf_plan_after.log`)


## TASK 4 - Create IAM Resources
Ensure that current directory is  `~/tf_aws_lab/base`

Create IAM resources:

-	**IAM group** (`name=test-move`).
-	**IAM policy** with write permission for "epam-aws-tf-lab" bucket only (`name=s3-write-epam-aws-tf-lab-${random_string.my_numbers.result}`). Hint: store your policy as yaml document side by side with configurations(or create 'files' subfolder for storing policy) and use templatefile() function to transfer IAM policy with imported S3 bucket name to a resource.
-	Create **IAM role**, attach the policy to it and create **IAM instance profile** for this IAM role. Allow to assume this role for ec2 service

Store all resources from this task in `iam.tf` file.

Equip all resources with following tags:
    - `Terraform=true`, 
    - `Project=epam-tf-aws-lab`
    - `Owner={StudentName}_{StudentSurname}`

Run `terraform validate`  and `terraform fmt` to check if your configuration valid and fits to a canonical format and style. Do this each time before apply your changes.
Run `terraform plan` to see your changes.

Apply your changes when you're ready.

### Definition of DONE:

- Terraform created infrastructure with no errors
- AWS Resources Created as expected (check AWS Console)
- Save following artifacts under `/reports/task4/` folder:
    - `terraform.tfstate` file
    - `terraform apply` log (`tf_apply.log`)
    - `terraform plan` (after changes) log (`tf_plan_after.log`)

## TASK 5 - Create Security Group
Ensure that current directory is  `~/tf_aws_lab/base`

Create following resources:

-	Security group (`name=ssh-inbound`, `port=22`, `allowed_ip_range="your_IP or EPAM_office-IP_range"`, `description="allows ssh access from safe IP-range"`).
-	Security group (`name=lb-http-inbound`, `port=80`, `allowed_ip_range="your_IP or EPAM_office-IP_range"`, `description="allows http access from safe IP-range to a LoadBalancer"`).
-	Security group (`name=http-inbound`, `port=80`, `source_security_group_id=id_of_lb-http-inbound_sg`, `description="allows http access from LoadBalancer"`). Hint: source_security_group_id is an attribute of[aws_security_group_rule resource](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group_rule). For details about how to configure securitygroups for loadbalancer see [documentation] (https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/elb-security-groups.html)


Store all resources from this task in `sg.tf` file.

Equip all resources with following tags:
    - `Terraform=true`, 
    - `Project=epam-tf-aws-lab`
    - `Owner={StudentName}_{StudentSurname}`

Run `terraform validate`  and `terraform fmt` to check if your configuration valid and fits to a canonical format and style. Do this each time before apply your changes.
Run `terraform plan` to see your changes.

Apply your changes when you're ready.

### Definition of DONE:

- Terraform created infrastructure with no errors
- AWS Resources Created as expected (check AWS Console)
- Save following artifacts under `/reports/task5/` folder:
    - `terraform.tfstate` file
    - `terraform apply` log (`tf_apply.log`)
    - `terraform plan` (after changes) log (`tf_plan_after.log`)

## TASK 6 - Form TF Output 
Ensure that current directory is  `~/tf_aws_lab/base`

Create outputs for your configuration:

- Create `outputs.tf` file.
- Following outputs required: `vpc_id`, `public subnet id`, `security group id`, `iam instance profile name`.

Store all resources from this task in `outputs.tf` file.

Run `terraform validate`  and `terraform fmt` to check if your configuration valid and fits to a canonical format and style. Do this each time before apply your changes.
Run `terraform plan` to see your changes.

Apply your changes when you're ready. You can update outputs without apply in a fact with `terraform refresh` command.

### Definition of DONE:

- Save following artifacts under `/reports/task6/` folder:
    - `terraform apply` log (`tf_apply.log`)
    - `terraform output ...` log (`tf_output.log`)

## TASK 7 - Configure remote data source

Learn about [terraform remote state data source](https://www.terraform.io/docs/language/state/remote-state-data.html).

! Change current directory to  `~/tf_aws_lab/compute`

Add remote state resources to your configuration to be able to import output resources:

-	Create data resource for base remote state. (backend="local")

Store all resources from this task in `data.tf` file.

Run `terraform validate`  and `terraform fmt` to check if your configuration valid and fits to a canonical format and style. Do this each time before apply your changes.
Run `terraform plan` to see your changes.

Apply your changes when you're ready.

### Definition of DONE:

- Save following artifacts under `/reports/task7/` folder:
    - `terraform.tfstate` file
    - `terraform init` log (`tf_init.log`)
    - `terraform apply` log (`tf_apply.log`)

## TASK 8 - Create EC2/ASG/ELB

Ensure that current directory is  `~/tf_aws_lab/compute`

Create auto-scaling group resources:

- Create Launch Template resource. (`name=epam-aws-tf-lab`,`image_id="actual Amazon Linux AMI2 image id"`, `instance_type=t2.micro`,`security_group_id={ssh-inbound-id,http-inbound-id}`,`key_name`,`iam_instance_profile`, `user_data script`)
- Provide template with `delete_on_termination = true` network interface parameter - to automate clean-up of the resources
- Author User Data bash script which should get 2 parameters on instance start-up and send it to a S3 Bucket as a text file with instance_id as its name:

User Data Details:

Getting EC2 Metadata
```
EC2_MACHINE_UUID=$(cat /sys/devices/virtual/dmi/id/product_uuid |tr '[:upper:]' '[:lower:]')
INSTANCE_ID=$(replace this text with request instance id from metadata e.g. using curl)
```

command to send text to S3 bucket (**use data rendenring to pass Bucket Name to this script**):
```
This message was generated on instance {INSTANCE_ID} with the following UUID {EC2_MACHINE_UUID}
echo "This message was generated on instance ${INSTANCE_ID} with the following UUID ${EC2_MACHINE_UUID}" | aws s3 cp - s3://{Backet name from task 3}/${INSTANCE_ID}.txt
```

- Create `aws_autoscaling_group` resource. (`name=epam-aws-tf-lab`,`max_size=min_size=1`,`launch_template=epam-aws-tf-lab`)
- Create Classic Loadbalancer and attach it to an auto-scaling group with `aws_autoscaling_attachment`. Configure `aws_autoscaling_group`  to ignore changes to the `load_balancers` and `target_group_arns` arguments within a lifecycle configuration block. (lb_port=80, instance_port=80, protocol=http, `security_group_id={lb-http-inbound-id}`).

Store all resources from this task in `asg.tf` file.

Equip all resources with following tags:
    - `Terraform=true`, 
    - `Project=epam-tf-aws-lab`
    - `Owner={StudentName}_{StudentSurname}`

Please keep in mind that autoscaling group requires special format for `Tags` section!


Run `terraform validate`  and `terraform fmt` to check if your configuration valid and fits to a canonical format and style. Do this each time before apply your changes.
Run `terraform plan` to see your changes.

Apply your changes when you're ready.

As the result ec2 instance should be launched by autoscaling-group and new file should be created on S3 bucket. 

### Definition of DONE:

- Terraform created infrastructure with no errors
- AWS Resources Created as expected (check AWS Console)
- Save following artifacts under `/reports/task8/` folder:
    - `terraform.tfstate` file
    - `terraform apply` log (`tf_apply.log`)
    - `terraform plan` (after changes) log (`tf_plan_after.log`)

    
# Working with state

## TASK 9 - Move state to S3/Locking

Hint: Create an S3 Bucket(`name=epam-aws-tf-state`) and DynamoDB table as a pre-requirement for this task. There are multiple ways to do this: including Terraform and CloudFormation. But
please just create both resources by a hands in AWS console. Those resources will be out of our IaC approach as they will never be recreated.

Learn about [terraform backend in AWS S3](https://www.terraform.io/docs/language/settings/backends/s3.html)

Refine your configurations:

- Refine `base` configuration by moving local state to a s3.
- Refine `base` configuration by adding locking with DynamoDB.
- Refine `compute` configuration by moving local state to a s3.
- Refine `compute` configuration by adding locking with DynamoDB.

Do not forget to change path to a remote state for `compute` configuration.

Run `terraform validate`  and `terraform fmt` to check if your modules valid and fits to a canonical format and style.
Run `terraform plan` to see your changes and re-apply your changes if it needed.

## TASK 10 - Move resources

Learn about [terraform state mv](https://www.terraform.io/docs/cli/commands/state/mv.html) command

You are going to move previously created resource(IAM group) from `base` to ` compute` state.
Hint: Keep in mind that there are 3 instances: AWS resource, Terraform state file which store some state of that resource, and Terraform configuration which describe resource. This meaning "move" resource is move it between states. Moreover to make it works you should delete resource from source configuration and add it to destination configuration(this action is not automated).

- Move `test-move` IAM group from the `base` state to the `compute` using `terraform state mv` command.
- Update both configuration according to this move.
- Run `terraform plan` on both configurations and observe the changes. Hint: there are should not be no changes detected (No resource creation or deletion in case of the correct resource move)

Run `terraform validate`  and `terraform fmt` to check if your configuration valid and fits to a canonical format and style.

### Definition of DONE:

- Terraform moved resources with no errors
- AWS Resources NOT changed (check AWS Console)
- Save following artifacts under `/reports/task10/` folder:
    - `terraform.tfstate` file for both configurations

## TASK 11 - Import resources

Learn about [terraform import](https://www.terraform.io/docs/cli/import/index.html) command

You are going to import new resource(IAM group) to your state.
Hint: Keep in mind that there are 3 instances: AWS resource, Terraform state file which store some state of that resource, and Terraform configuration which describe resource. This meaning to "import" resource you should import resource attributes to Terraform state. Then you have to add resource to destination configuration(this action is not automated).

- Create IAM group in AWS Console(name="test-import").
- Add new resource aws_iam_group "test-import" to the `compute` configuration.
- Run `terraform plan` to see your changes but do not apply changes.
- Import `test-import` IAM group to the `compute` state.
- Run `terraform plan` again to ensure that import was successful.

Run `terraform validate`  and `terraform fmt` to check if your configuration valid and fits to a canonical format and style.
If applicable all resources should be tagged with following tags {Terraform=true, Project=epam-tf-aws-lab}.
If applicable all resources should be defined with the provider alias.


- Terraform imported resources with no errors
- AWS Resources NOT changed (check AWS Console)
- Save following artifacts under `/reports/task11/` folder:
    - `terraform.tfstate` file for `compute` configuration

## TASK 12 - Use data discovery
Learn about [terraform data sources](https://www.terraform.io/docs/language/data-sources/index.html) and [querying terraform data sources](https://learn.hashicorp.com/tutorials/terraform/data-sources?in=terraform/configuration-language&utm_source=WEBSITE&utm_medium=WEB_BLOG&utm_offer=ARTICLE_PAGE).

In this task we are going to use data driven approach instead to use remote state data source.

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

- Use data source to request the following resources: `vpc_id`, `public subnet id`, `security group id`, `iam instance profile name`.

Hint: These data sources should replace remote state outputs therefore you can delete `data "terraform_remote_state" "base"` resource from current state and the `outputs.tf` file from the `base` configuration. **Don't forget to replace references with a new data sources.**
Hint: run `terraform refresh` command under `base` configuration to reflect changes.

Store all resources from this task in `data.tf` file.

Run `terraform validate`  and `terraform fmt` to check if your configuration valid and fits to a canonical format and style. Do this each time before apply your changes.
Run `terraform plan` to see your changes. Also you can use `terraform refresh`.
If applicable all resources should be defined with the provider alias.

Apply your changes when you're ready.


# Advanced tasks

## TASK 13 - Expose node output with nginx

Ensure that current directory is  `~/tf_aws_lab/compute`

Change User Data script at Launch Template as follows:

-   Nginx binary should be installed on instance (`port=80`).
-   Variables INSTANCE_ID and EC2_MACHINE_UUID should be defined( see Task 8).
-   Nginx default page should be configured to return the same text as we put previously to  S3 bucket in Task 8: "This message was generated on instance ${INSTANCE_ID} with the following UUID ${EC2_MACHINE_UUID}".
-   Nginx server should be started

Run `terraform validate`  and `terraform fmt` to check if your configuration valid and fits to a canonical format and style. Do this each time before apply your changes.
Run `terraform plan` to see your changes.

Apply your changes when you're ready.

### Definition of DONE:

- Terraform created infrastructure with no errors
- AWS Resources Created as expected (check AWS Console)
- Nginx server respond on Loadbalancer's IP Address with expected response 
- Save following artifacts under `/reports/task13/` folder:
    - `terraform.tfstate` file
    - `terraform apply` log (`tf_apply.log`)
    - `terraform plan` (after changes) log (`tf_plan_after.log`)

## TASK 14 - Modules

Learn about [terraform modules](https://www.terraform.io/docs/language/modules/develop/index.html)

Refine your configurations:

- Refine `base` configuration by creating module for VPC related resources: vpc, subnets, routes, internet gateways.
- Refine `base` configuration by creating module for security group related resources.
- Refine `base` configuration by creating module for IAM role related resources.
- [Optional] Refine `compute` configuration by creating Autoscaling group module.


Store modules in `~/tf_aws_lab/modules/` subfolders.

Run `terraform validate`  and `terraform fmt` to check if your modules valid and fits to a canonical format and style.
Run `terraform plan` to see your changes and re-apply your changes if it needed.
