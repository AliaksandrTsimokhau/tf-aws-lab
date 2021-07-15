### Problem to Be Solved 
 This lab shows you how to use Terraform to create infrastructure in AWS including auto-scaling group, VPC, subnets, security groups and IAM role. Each instance will report its data to a specified S3 bucket on startup. This task is binding to real production needs – for instance developers could request instances with ability to writing debug information to S3 bucket.

 
### Explanation of the Solution 
You will use Terraform with AWS provider to create 2 separate Terraform configurations:
 1) Base configuration
 2) Compute configuration
After you’ve created configuration, we will work on its optimization like using data driven approach and creating modules.


## PRE-REQUISITES
1. Learn about [terraform state](https://www.terraform.io/docs/language/state/index.html).

2. Create a folder with 2 subfolders: ‘base’ and ‘compute’. E.g. mkdir -p `~/tf_aws_lab/{base,compute}`

3. All actions should be done under those subfolder as Terraform gets it context from current working directory. Change current directory  to `~/tf_aws_lab/base` folder and create root.tf file. 
Add `terraform {}`empty block to this file. Create AWS provider block inside `root.tf` file with the following attributes: region= "us-east-1", alias = "use1", shared_credentials_file = "~/.aws/credentials".

Hint: Add your AWS credentials to   "~/.aws/credentials" folder. See [this](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html) document for details

Run `terraform init` to initialize your configuration. 
Run `terraform validate`  and `terraform fmt` to check if your configuration valid and fits to a canonical format and style. Do this each time before apply your changes.
Run `terraform plan` to ensure that there are no changes
Please use underscore Terraform resources naming, e.g. "my_resource" instead of "my-resource".

4. Change current directory  to `~/tf_aws_lab/compute` and repeat the steps in [3].

You are ready for lab!

# Creating Infrastructure

## TASK 1
Change current directory  to `~/tf_aws_lab/base`

Create network stack for your infrastructure:

-	VPC (name=vpc-01, cidr=10.10.0.0/16)
-	Private subnet(name=private-01, cidr=10.10.2.0/24)
-	Public subnet (name=public-01, cidr=10.10.1.0/24)
-	Internet gateway 
-	Routing table to bind IGW with Public subnet

Store all resources from this task in `vpc.tf` file.
If applicable all resources should be tagged with following tags {Terraform=true, Project=epam-tf-aws-lab}.
If applicable all resources should be defined with the provider alias.
Run `terraform validate`  and `terraform fmt` to check if your configuration valid and fits to a canonical format and style. Do this each time before apply your changes.
Run `terraform plan` to see your changes.

Apply your changes when you're ready.

## TASK 2
Ensure that current directory is `~/tf_aws_lab/base`

Create custom ssh key-pair to access your ec2 instances:

-	Create your ssh key pair [refer to this document](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html#how-to-generate-your-own-key-and-import-it-to-aws)
-	Create `variables.tf` file with empty variable "ssh_key" but with the following description "Provides custom public ssh key". Never store you secrets inside the code!
-	Create `key_pai.tf` file with `aws_key_pair` resource. Use ssh_key variable as a public key source.
- Run `terraform plan` and provide required public key. Observe the output and run `terraform plan` again.
- To prevent providing ssh key on each configuration run but stay secure - set binding environment variable-`export TF_VAR_ssh_key="YOUR_PUBLIC_SSH_KEY_STRING"`
- Run `terraform plan` and observe the output.

If applicable all resources should be tagged with following tags {Terraform=true, Project=epam-tf-aws-lab}.
If applicable all resources should be defined with the provider alias.
Run `terraform validate`  and `terraform fmt` to check if your configuration valid and fits to a canonical format and style. Do this each time before apply your changes.

Apply your changes when you're ready.

## TASK 3

Ensure that current directory is  `~/tf_aws_lab/base`

Create S3 bucket as the storage for your infrastructure:

-	Create `s3.tf`. Name your bucket "epam-aws-tf-lab-${random_string.my_numbers.result}" to provide it with partition unique name. See [random_string](https://registry.terraform.io/providers/hashicorp/random/latest/docs/resources/string) documentation for details
-	Set bucket acl as private. Never share your bucket to a world!


If applicable all resources should be tagged with following tags {Terraform=true, Project=epam-tf-aws-lab}.
If applicable all resources should be defined with the provider alias.
Run `terraform validate`  and `terraform fmt` to check if your configuration valid and fits to a canonical format and style. Do this each time before apply your changes.
Run `terraform plan` to see your changes.

Apply your changes when you're ready.

### TASK 4
Ensure that current directory is  `~/tf_aws_lab/base`

Create IAM resources:

-	IAM group (name=test-move).
-	IAM policy with write permission for "epam-aws-tf-lab" bucket only (name=s3-write-epam-aws-tf-lab-${random_string.my_numbers.result}). Hint: store your policy as yaml document side by side with configurations(or create 'files' subfolder for storing policy) and use templatefile() function to transfer IAM policy with imported S3 bucket name to a resource.
-	Create IAM role, attach the policy to it and create IAM instance profile for this IAM role.


Store all resources from this task in `iam.tf` file.
If applicable all resources should be tagged with following tags {Terraform=true, Project=epam-tf-aws-lab}.
If applicable all resources should be defined with the provider alias.
Run `terraform validate`  and `terraform fmt` to check if your configuration valid and fits to a canonical format and style. Do this each time before apply your changes.
Run `terraform plan` to see your changes.

Apply your changes when you're ready.

### TASK 5
Ensure that current directory is  `~/tf_aws_lab/base`

Create security group resource to access instances via ssh:

-	Security group group (name=ssh-inbound, port=22, allowed_ip_range="your_IP or EPAM_office-IP_range").


Store all resources from this task in `sg.tf` file.
If applicable all resources should be tagged with following tags {Terraform=true, Project=epam-tf-aws-lab}.
If applicable all resources should be defined with the provider alias.
Run `terraform validate`  and `terraform fmt` to check if your configuration valid and fits to a canonical format and style. Do this each time before apply your changes.
Run `terraform plan` to see your changes.

Apply your changes when you're ready.

### TASK 6
Ensure that current directory is  `~/tf_aws_lab/base`

Create outputs for your configuration:

-	Create `outputs.tf` file.
- Following outputs required: vpc_id, private subnet id, public subnet id, security group id, ssh key name, iam instance profile name, s3 bucket name, 


Store all resources from this task in `output.tf` file.

Run `terraform validate`  and `terraform fmt` to check if your configuration valid and fits to a canonical format and style. Do this each time before apply your changes.
Run `terraform plan` to see your changes.

Apply your changes when you're ready. You can update outputs without apply in a fact with `terraform refresh` command.

### TASK 7

Learn about [terraform remote state data source](https://www.terraform.io/docs/language/state/remote-state-data.html).

! Change current directory to  `~/tf_aws_lab/compute`

Add remote state resources to your configuration to be able to import output resources:

-	Create data resource for base remote state. (backend="local")

Store all resources from this task in `data.tf` file.

Run `terraform validate`  and `terraform fmt` to check if your configuration valid and fits to a canonical format and style. Do this each time before apply your changes.
Run `terraform plan` to see your changes.

Apply your changes when you're ready.

### TASK 8
Ensure that current directory is  `~/tf_aws_lab/compute`

Create auto-scaling group resources:

-	Create Launch Template resource. (name=epam-aws-tf-lab,image_id="actual Amazon Linux AMI2 image id", instance_type=t3.micro,security_group_id,key_name,iam_instance_profile, user_data script)
- Provide template with `delete_on_termination = true` network interface parameter - to automate clean-up of the resources
- Author User Data bash script which should get 2 parameters on instance start-up and send it to a S3 Bucket as a text file with instance_id as its name:

Parameters:
```
EC2_MACHINE_UUID=$(cat /sys/devices/virtual/dmi/id/product_uuid |tr '[:upper:]' '[:lower:]')
INSTANCE_ID=$(curl --connect-timeout 5 -s http://169.254.169.254/latest/meta-data/instance-id 2>/dev/null)
```

command to send text to S3 bucket:
```
This message was generated on instance {INSTANCE_ID} with the following UUID {EC2_MACHINE_UUID}
echo "This message was generated on instance ${INSTANCE_ID} with the following UUID ${EC2_MACHINE_UUID}" | aws s3 cp - s3://rtcp-perftest/${INSTANCE_ID}.txt
```

- Create aws_autoscaling_group resource. (name=epam-aws-tf-lab,max_size=min_size=1,launch_template=epam-aws-tf-lab)


Store all resources from this task in `asg.tf` file.
If applicable all resources should be tagged with following tags {Terraform=true, Project=epam-tf-aws-lab}. Please keep in mind that autoscaling group requires special format for `Tags` section!
If applicable all resources should be defined with the provider alias.

Run `terraform validate`  and `terraform fmt` to check if your configuration valid and fits to a canonical format and style. Do this each time before apply your changes.
Run `terraform plan` to see your changes.

Apply your changes when you're ready.

As the result ec2 instance should be launched by autoscaling-group and new file should be created on S3 bucket. 
