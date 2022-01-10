# Notes on Terraform

Based on Kodekloud's Terraform course

# 1. HCL Basics

Steps once you have .tf file. Run in directory.

1. `terraform init` - This can be run as many times without changing infrastructure.
2. `terraform plan`
3. `terraform apply`

Run `terraform show` to display file

HCL format

```terraform
resource /* Block name */ "aws_s3_bucket" /* Provider_Resource */ "data" /* Resource name */ {
    bucket = "webserver-bucket-200" # Arguments
    acl = "private"
}
```

Note you can [enable terraform bash autocomplete](https://www.terraform.io/cli/commands) with `terraform -install-autocomplete` which adds `complete -C /usr/local/bin/terraform terraform` to ~/.bashrc

## 1.1 Update infrastructure

```terraform
# local.tf
resource "local_file" "pet" {
    filename = "/root/pets.txt"
    content = "We love pets!"
    file_permission = "0700"
}
```

On running `terraform plan` it says "local file.pet must be replaced", run `terraform apply` to replace and finally `terraform destroy` to delete the infrastructure.

# 2. Terraform Basics

## 2.1 Terraform Providers

Official ones:

1. AWS / GCP / GCP or local

Verified (by Hashicorp)

1. f5
2. Heroku
3. Digital Ocean

Community (not verified but by community)

1. activedirectory
2. ucloud

**Format**: \<hostname>/\<organizational-namespace>/\<provider type> eg. registry.terraform.io/hashicorp/local

When you run `terraform init` plugins for resources are downloaded into the current directory's **.terraform/plugins**.

To create a single terraform resource, given

```terraform
resource "local_file" "cyberpunk" {
  filename     = "/root/cyberpunk2077.txt"
  content  = "All I need for Christmas is Cyberpunk 2077!"
}
resource "local_file" "xbox" {
  filename     = "/root/xbox.txt"
  content  = "Wouldn't mind an XBox either!"
}
```

Do `terraform apply -target=local_file.xbox` to create just the xbox resource. Note there is [no way](https://devops.stackexchange.com/questions/4292/terraform-apply-only-one-tf-file) to terraform apply on just one .tf file.

## 2.2 Terraform variables

Listed in variables.tf

```ruby
# variables.tf
variable "jedi" {
     type = map
     default = {
     filename = "/root/first-jedi"
     content = "phanius"
     }
  
}
```

Arguments [defined here](https://www.terraform.io/language/values/variables)

`type` determines the data type, `description` is optional etc.

In main.tf or other TF files they can be referenced with `var.jedi` or with index to specify element

```terraform
resource "local_file" "jedi" {
     filename = var.jedi["filename"]
     content = var.jedi["content"]
}
```

Can also omit the values in variables.tf but this will make terraform prompt for values for all variables.

Alternatively specify them as arguments in command with `terraform apply -var "filename=/root/pets.txt"` or export as env variables `export TF_VAR_filename="/root/pets.txt"`

Lastly one can put them in *variable definition files* 

* terraform.tfvars
* terraform.tfvars.json
* *.auto.tfvars
* *.auto.tfvars.json

then do `terraform apply`

```text
# terraform.tfvars
filename = "/root/pets.txt"
content = "We love pets!"
prefix = "Mrs"
separator = "."
length = "2"
```

Or can pass the var file in argument `terraform apply -var-file variables.tfvars` or *.tfvars.json

### 2.2.1 Variable definition precedence

From [docs](https://www.terraform.io/language/values/variables), the following in reverse order if all are present ie. 5 take precedence over 4 and 1.

1. Environment variables
2. The `terraform.tfvars` file, if present.
3. The `terraform.tfvars.json` file, if present.
4. Any `*.auto.tfvars` or `*.auto.tfvars.json` files, processed in lexical order of their filenames.
5. Any `-var` and `-var-file` options on the command line, in the order they are provided. (This includes variables set by a Terraform Cloud workspace.)

## 2.3 Resource Attributes

Similar to cross-stack references in CFN, references output of another resource in one resource definition.

For example for time_static

```ruby
resource "time_static" "example" {}
```

The exported resource has the [following attributes](https://registry.terraform.io/providers/hashicorp/time/latest/docs/resources/static), of which one is **id**.

It can be referenced with **time_static.example.id** (referenced expression)

```terraform
resource "local_file" "time" {
    content     = "Time stamp of this file is ${time_static.time_update.id}"
    filename = "/root/time.txt"
}
```

## 2.4 Resource Dependencies

If resource attribute is referenced in a resource definition, dependency is implicit and TF creates the dependency resource first.

### 2.4.1 Implicit dependency

Implicit dependency example for [tls_private_key](https://registry.terraform.io/providers/hashicorp/tls/latest/docs/resources/private_key#private_key_pem), note the referenced expression for attribute `private_key_pem` is **tls_private_key.pvtkey.private_key_pem**.

```terraform
resource "tls_private_key" "pvtkey" {
  algorithm = "RSA"
  rsa_bits = 4096
}

resource "local_file" "key_details" {
    content     = tls_private_key.pvtkey.private_key_pem
    filename = "/root/key.txt"
}
```

### 2.4.2 Explicit dependency

[Explicit](https://learn.hashicorp.com/tutorials/terraform/dependencies?in=terraform/0-13) resource dependency is used when there is no resource attribute referenced but you still want to create one first then another.

In this example, local_file whale explicitly depends on local_file krill, and there's no referenced expression.

```terraform
resource "local_file" "whale" {
    filename = "/root/whale"
    depends_on = [local_file.krill]
}

resource "local_file" "krill" {
    filename = "/root/krill"
}
```

## 2.5 Output Variables

We can [export outputs](https://www.terraform.io/language/values/outputs) from resources like this

```terraform
resource "random_uuid" "id7" {
   
}
resource "random_integer" "order1" {
  min     = 1
  max     = 99999
 
}
output "id7" {
    value = random_uuid.id7.result
   
}
output "order1" {
 value = random_integer.order1.result
 
}
```

After applying we can view the output like this

```text
root@iac-server:~/terraform-projects/data# terraform output order1
30513
```

# 3. Terraform State

TF creates a **terraform.tfstate** file when `terraform apply` is run. The is in JSON format, which records infrastructure created by TF (similar to etcd store in k8s).

terraform.tfstate also stores dependencies so that if main.tf or other TF files have deleted resources it knows which should be removed first, in order. 

File can be stored in remote state stores so that each user will update the same key-value store when they run TF instead of creating their own terraform.tfstate

`terraform init` doesn't affect state file.

# 4. Working with TF

## 4.1 TF commands

`terraform validate` used to validate config

`terraform fmt` formats the TF files to be more readable

`terraform providers` displays all the providers used

`terraform providers mirror /target/path` Copies TF plugins to specified dir

`terraform output` prints all outputs.

`terraform refresh` updates state file with detected drift with actual infrastructure. Runs automatically with `plan` and `apply`

`terraform state show resource.resource-name` to display attributes of a resource captured in TF state.

## 4.2 Mutable vs Immutable Infrastructure

Immutable infrastructure means config management destroys and recreates infrastructure rather than update existing ones. For example, local_file updates mean old files are deleted and new ones with the desired spec are created.

### 4.2.1 Lifecycle Rules

If you want to create new infrastructure first before destroying old ones, use lifecycle rules. Also can be used to prevent destruction of resources. [Ref](https://www.terraform.io/language/meta-arguments/lifecycle)

Options

1. create_before_destroy
2. prevent_destroy
3. ignore_changes - Tells TF to ignore changes to specified arguments eg. EC2 instance name tags.

For example

```terraform
resource "random_string" "string" {
  length = var.length
  keepers = {
    length = var.length
  }

  lifecycle {
    create_before_destroy = true
  }
}
```

Note that create_before_destroy may end up destroying the resource if only one resource can exist at any one time eg. files. New file gets created -> then gets destroyed.

## 4.3 Datasources

* Allow TF to read data from sources created outside its control, instead of managed resources which are provisioned by TF.
* Block name **data** instead of **resource**
* [Ref](https://www.terraform.io/language/data-sources)

Example

```terraform
output "os-version" {
  value = data.local_file.os.content # Format for referencing data
}
data "local_file" "os" {
  filename = "/etc/os-release"
}
```

## 4.4 Meta-Arguments

* Used to contain code variables like loop arguments (eg. i in for loops)

### 4.4.1 Count

Simple usage: This creates the file 3 times, all overwriting.

```terraform
resource "local_file" "name" {
    filename = "/root/user-data"
    sensitive_content = "password: S3cr3tP@ssw0rd"
  
    count = 3
}
```

Or you can have it like this

```terraform
# main.tf
resource "local_file" "pet" {
    filename = var.filename[count.index]
    count = length(var.filename)
}
# variables.tf
variable "filename" {
    default = [
        "/root/dogs.txt",
        "/root/cats.txt"
    ]
}
```

This creates /root/dogs.txt and /root/cats.txt

### 4.4.2 for_each

This example creates files /root/user10, /root/user11, /root/user12 with the content "password: S3cr3tP@ssw0rd".

```terraform
# main.tf 
resource "local_file" "name" {
    filename = each.value
    sensitive_content = var.content
  
    for_each = toset(var.users)
}
# variables.tf
variable "users" {
    type = list(string)
    default = [ "/root/user10", "/root/user11", "/root/user12", "/root/user10"]
}
variable "content" {
    default = "password: S3cr3tP@ssw0rd"
  
}
```

This is what is displayed in `terraform show`

```text
root@iac-server:~/terraform-projects/project-shade# terraform show
# local_file.name["/root/user11"]:
resource "local_file" "name" {
    directory_permission = "0777"
    file_permission      = "0777"
    filename             = "/root/user11"
    id                   = "6b32344cb73c40d126d99ce62309878befca64ce"
    sensitive_content    = (sensitive value)
}

# local_file.name["/root/user12"]:
resource "local_file" "name" {
    directory_permission = "0777"
    file_permission      = "0777"
    filename             = "/root/user12"
    id                   = "6b32344cb73c40d126d99ce62309878befca64ce"
    sensitive_content    = (sensitive value)
}

# local_file.name["/root/user10"]:
resource "local_file" "name" {
    directory_permission = "0777"
    file_permission      = "0777"
    filename             = "/root/user10"
    id                   = "6b32344cb73c40d126d99ce62309878befca64ce"
    sensitive_content    = (sensitive value)
}
```

## 4.5 Version Constraints [[Ref](https://www.terraform.io/language/expressions/version-constraints)]

`terraform init` always downloads the latest plugin versions, but we may not want that.

To use a specific version of provider, go to Use Provider in the TF registry ([eg](https://registry.terraform.io/providers/hashicorp/local/1.4.0/docs/resources/file)) and add a block that resembles this to use 1.4.0

```terraform
terraform {
  required_providers {
    local = {
      source = "hashicorp/local"
      version = "1.4.0"
    }
  }
}
```

* Exclude latest version with ` version = "!= 1.4.0"`

* Use lower version `version = "< 1.4.0"`

* Use higher version `version = "> 1.4.0"` 
* More complex constraints `version = "> 1.2.0, < 2.0.0 != 1.4.0"`

Pessimistic constraints allow only the rightmost component to increment eg. `~> 1.0.4` allows from 1.0.4 up till 1.0.9 but not 1.1.0

* In general, TF downloads the highest version that meets constraints.

# 5. Terraform with AWS

## 5.1 IAM with TF

To create a user

```terraform
resource "aws_iam_user" "users" {
  name = "mary"
}
provider "aws" {
  region     = "us-east-1"
  access_key = "my-access-key"
  secret_key = "my-secret-key"
}
```

Here we create a list of users using count. Note that the provider *aws* must be included or it'll throw an error.

```terraform
# iam-user.tf
resource "aws_iam_user" "users" {
  name = var.project-sapphire-users[count.index]
  count = length(var.project-sapphire-users)
}
# provider.tf
provider "aws" {
  region     = "us-east-1"
  access_key = "my-access-key"
  secret_key = "my-secret-key"
}
# variables.tf
variable "project-sapphire-users" {
     type = list(string)
     default = [ "mary", "jack", "jill", "mack", "buzz", "mater"]
}
```

After creating it shows

```text
root@iac-server:~/terraform-projects/IAM# terraform show
# aws_iam_user.users[2]:
resource "aws_iam_user" "users" {
    arn           = "arn:aws:iam::000000000000:user/jill"
    force_destroy = false
    id            = "jill"
    name          = "jill"
    path          = "/"
    tags_all      = {}
    unique_id     = "d8180z4gpje966rjzkzu"
}

# aws_iam_user.users[3]:
resource "aws_iam_user" "users" {
    arn           = "arn:aws:iam::000000000000:user/mack"
    force_destroy = false
    id            = "mack"
    name          = "mack"
    path          = "/"
    tags_all      = {}
    unique_id     = "fegoztb3cggb1imvjn5v"
}

# aws_iam_user.users[4]:
resource "aws_iam_user" "users" {
    arn           = "arn:aws:iam::000000000000:user/buzz"
    force_destroy = false
    id            = "buzz"
    name          = "buzz"
    path          = "/"
    tags_all      = {}
    unique_id     = "uzwknw1793wxaaimk313"
}

# aws_iam_user.users[5]:
resource "aws_iam_user" "users" {
    arn           = "arn:aws:iam::000000000000:user/mater"
    force_destroy = false
    id            = "mater"
    name          = "mater"
    path          = "/"
    tags_all      = {}
    unique_id     = "8tyg774w4paat87tzbkc"
}

# aws_iam_user.users[0]:
resource "aws_iam_user" "users" {
    arn           = "arn:aws:iam::000000000000:user/mary"
    force_destroy = false
    id            = "mary"
    name          = "mary"
    path          = "/"
    tags          = {}
    tags_all      = {}
    unique_id     = "dpquntk6bxndc6qqjaaq"
}

# aws_iam_user.users[1]:
resource "aws_iam_user" "users" {
    arn           = "arn:aws:iam::000000000000:user/jack"
    force_destroy = false
    id            = "jack"
    name          = "jack"
    path          = "/"
    tags_all      = {}
    unique_id     = "utcsesuqhq8ncczf8ix1"
}
```

## 5.2 S3 with TF

To upload a file **/root/woody.jpg** to an existing S3 bucket named **pixar-studios-2020**. Docs [here](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/s3_bucket_object)

```terraform
# main.tf
data "aws_s3_bucket" "pixar-studios-2020" {
  bucket = "pixar-studios-2020"
}

resource "aws_s3_bucket_object" "object" {
  bucket = data.aws_s3_bucket.pixar-studios-2020.id
  key    = "woody.jpg"
  source = "/root/woody.jpg"

}
# provider.tf
provider "aws" {
  region                      = var.region
  s3_force_path_style         = true
  access_key = "my-access-key"
  secret_key = "my-secret-key"
    
}
# terraform.tfvars
region = "us-east-1"
# variables.tf
variable "region" {
}
```

After applying

```text
root@iac-server:~/terraform-projects/S3-Buckets/Pixar# terraform show
# aws_s3_bucket_object.object:
resource "aws_s3_bucket_object" "object" {
    acl                = "private"
    bucket             = "pixar-studios-2020"
    bucket_key_enabled = false
    content_type       = "binary/octet-stream"
    etag               = "d134df2a93e3249d0a944bed29c4a0e4"
    force_destroy      = false
    id                 = "woody.jpg"
    key                = "woody.jpg"
    source             = "/root/woody.jpg"
    storage_class      = "STANDARD"
    tags_all           = {}
}

# data.aws_s3_bucket.pixar-studios-2020:
data "aws_s3_bucket" "pixar-studios-2020" {
    arn                         = "arn:aws:s3:::pixar-studios-2020"
    bucket                      = "pixar-studios-2020"
    bucket_domain_name          = "pixar-studios-2020.s3.amazonaws.com"
    bucket_regional_domain_name = "pixar-studios-2020.s3.amazonaws.com"
    hosted_zone_id              = "Z3AQBSTGFYJSTF"
    id                          = "pixar-studios-2020"
    region                      = "us-east-1"
}
```

To-do: Include bucket policy example here.

## 5.4 DynamoDB with TF

Every table needs a name, hash_key (partition key) and an [attribute object](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/dynamodb_table#attribute) for that primary key. The attribute object specifies the type of data the partition key is (String, Number or Binary data)

```terraform
# main.tf
resource "aws_dynamodb_table" "project_sapphire_user_data" {
  name           = "userdata"
  billing_mode   = "PAY_PER_REQUEST"
  hash_key       = "UserId"

  attribute {
    name = "UserId"
    type = "N" # Number
  }
}
# provider.tf
provider "aws" {
  region                      = var.region
  s3_force_path_style = true
  ...
}
# variables.tf
variable "region" {
    default = "us-west-2"
  
}
```

Another example, suppose we want to insert this data

```json
{
"AssetID": {"N": "1"},
"AssetName": {"S": "printer"},
"age": {"N": "5"},
"Hardware": {"B": "true" }
}
```

into DynamoDB table **inventory**

```terraform
resource "aws_dynamodb_table" "project_sapphire_inventory" {
  name           = "inventory"
  billing_mode   = "PAY_PER_REQUEST"
  hash_key       = "AssetID"

  attribute {
    name = "AssetID"
    type = "N"
  }
  attribute {
    name = "AssetName"
    type = "S"
  }
  attribute {
    name = "age"
    type = "N"
  }
  attribute {
    name = "Hardware"
    type = "B"
  }
  global_secondary_index {
    name             = "AssetName"
    hash_key         = "AssetName"
    projection_type    = "ALL"
    
  }
  global_secondary_index {
    name             = "age"
    hash_key         = "age"
    projection_type    = "ALL"
    
  }
  global_secondary_index {
    name             = "Hardware"
    hash_key         = "Hardware"
    projection_type    = "ALL"
    
  }
}
```

We can just add this resource

```terraform
resource "aws_dynamodb_table_item" "upload" {
  table_name = aws_dynamodb_table.project_sapphire_inventory.name
  hash_key   = aws_dynamodb_table.project_sapphire_inventory.hash_key

  item = <<ITEM
  {
  "AssetID": {"N": "1"},
  "AssetName": {"S": "printer"},
  "age": {"N": "5"},
  "Hardware": {"B": "true" }
  }
ITEM
}
```

After applying, TF shows

```text
# aws_dynamodb_table_item.upload:
resource "aws_dynamodb_table_item" "upload" {
    hash_key   = "AssetID"    
    id         = "inventory|AssetID|||1"
    item       = jsonencode(
        {                   
            AssetID   = {         
                N = "1"           
            }                  
            AssetName = {         
                S = "printer" 
            }                 
            Hardware  = {   
                B = "true"              
            }                           
            age       = {      
                N = "5"           
            }                 
        }                     
    )                  
    table_name = "inventory"
}
```

# 6. Remote State

* State files normally stored in the working directory where command is run.
* Not advisable to store state file in Git because it may contain sensitive info, since it stores all info about existing infra.
* Github doesn't support state locking, so issues arise when multiple users access it.
* When remote backend is configured, TF loads/unloads state file every time its required.
* Options are S3 for state file storage and DynamoDB for locking state.

## 6.1 S3 backend config

Configure this for terraform block

```terraform
# terraform.tf
terraform {
  backend "s3" {
    bucket = "remote-state" # bucket name
    key    = "terraform.tfstate" # name of TF state file
    region = "us-east-1"
  }
}
```

Run `terraform init` once done to store it in S3 bucket, then can delete file terraform.tfstate on local machine

## 6.2 TF state commands

List resources in TF state with `terraform state list` or append resource name at the end to select just one

Print all attributes of resource with `terraform state show resource`

Rename a TF resource with `terraform state mv aws_dynamodb_table.state-locking aws_dynamodb_table.state-locking-db`. If you also edited TF files to reflect this, `terraform plan` will not detect any difference.

Display remote TF state with `terraform state pull`

Remove resource from state file (resource not destroyed) `terraform state rm resource`. Used when you don't want TF to manage resource any more.

# 7. Terraform Provisioners

## 7.1 AWS EC2 with TF

This provisions instance in region with specified AMI and user-data

```terraform
# provider.tf
provider "aws" {
  region = var.region
}
# main.tf
variable "region" {
  type    = string
  default = "eu-west-2"
}

variable "ami" {
  type    = string
  default = "ami-06178cf087598769c"
}

variable "instance_type" {
  type    = string
  default = "m5.large"
}

resource "aws_instance" "cerberus" {
  ami           = var.ami
  instance_type = var.instance_type
  key_name      = "cerberus"
  user_data     = file("/root/terraform-projects/project-cerberus/install-nginx.sh")

}

resource "aws_key_pair" "cerberus-key" {
  key_name   = "cerberus"
  public_key = file("/root/terraform-projects/project-cerberus/.ssh/cerberus.pub")
}
```

After creating

```text
# aws_instance.cerberus:
resource "aws_instance" "cerberus" {
    ami                          = "ami-06178cf087598769c"
    arn                          = "arn:aws:ec2:eu-west-2::instance/i-161187f3d0df86050"
    associate_public_ip_address  = true
    availability_zone            = "eu-west-2a"
    disable_api_termination      = false
    ebs_optimized                = false
    get_password_data            = false
    id                           = "i-161187f3d0df86050"
    instance_state               = "running"
    instance_type                = "m5.large"
    ipv6_address_count           = 0
    ipv6_addresses               = []
    key_name                     = "cerberus"
    monitoring                   = false
    primary_network_interface_id = "eni-1b0de80a"
    private_dns                  = "ip-10-198-49-32.eu-west-2.compute.internal"
    private_ip                   = "10.198.49.32"
    public_dns                   = "ec2-54-214-63-252.eu-west-2.compute.amazonaws.com"
    public_ip                    = "54.214.63.252"
    secondary_private_ips        = []
    security_groups              = []
    source_dest_check            = true
    subnet_id                    = "subnet-73ae4073"
    tags_all                     = {}
    tenancy                      = "default"
    user_data                    = "ace853dfcd5deb36a3802184e0347bf471f627ed"
    vpc_security_group_ids       = []

    root_block_device {
        delete_on_termination = true
        device_name           = "/dev/sda1"
        encrypted             = false
        iops                  = 0
        tags                  = {}
        throughput            = 0
        volume_id             = "vol-e50ecace"
        volume_size           = 8
        volume_type           = "standard"
    }
}
# aws_key_pair.cerberus-key:
resource "aws_key_pair" "cerberus-key" {
    arn         = "arn:aws:ec2:eu-west-2::key-pair/cerberus"
    fingerprint = "45:1c:7b:3c:1f:cc:1a:4f:9f:e8:37:8b:76:f4:17:04"
    id          = "cerberus"
    key_name    = "cerberus"
    public_key  = "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDg0WB80QKTPvOKauzJdtJQKdaVs7+fdyuEUwaJlh8rWkRr5aAPDjJCrCKo8u+KlS8cE+rbGS2LRRNbmltmXgmgsjHjGCcvqZXCByNSVzVBXno0rx+qiSSXRrSIj3lZcVvXhQGWD/DKoG+2UwPxPIR2nzgDRqbz7bsoLULt2ZG8iGvEZ05sDLbfauZoxUJb/BgwpdkbfL+0R07JMuqkWTeU9ya9FgFslXLKF8SeYRYCUNx71CypB3laPjF6ySqBkKKfqnVo7AaLw+ajWcnqbWqlMvo+L0e9xav9gotoQyJt2dXKgbm5yp6O97OS+cwj9kmcTWhbaFbvRMIiIXp5wQQ9 root@iac-server"
    tags        = {}
    tags_all    = {}
}
```

To create a key pair

```terraform
resource "aws_key_pair" "cerberus-key" {
  key_name   = "cerberus"
  public_key = file("/root/terraform-projects/project-cerberus/.ssh/cerberus.pub")
}
```



## 7.2 TF provisioners

* [remote-exec](https://www.terraform.io/language/resources/provisioners/remote-exec) allows you to run commands on the remote AWS EC2 instance similar to user-data. Requires connectivity to EC2 instance
* [local-exec](https://www.terraform.io/language/resources/provisioners/local-exec) runs commands on the local machine with TF installed. Used for example to get public IP of EC2 instance.

This local-exec copies public_dns to a file on the local machine.

```terraform
resource "aws_eip" "eip" {
  instance = aws_instance.cerberus.id
  vpc      = true
  provisioner "local-exec" {
    command = "echo ${self.public_dns} >> /root/cerberus_public_dns.txt"
  }
}
```

# 8. Terraform Import, Tainting and Debugging

## 8.1 Tainting

When any part of resource creation fails, TF marks it as tainted, visible with `terraform plan`. Terraform will re-create the entire resource if `apply` is run again, even if most of it succeeded.

Terraform will try to update the resource in place if not tainted.

Can manually `terraform taint resource` to re-create. Untaint with `terraform untaint resource`

## 8.2 Debugging

Set `export TF_LOG=<log_lvl>` to set logging verbosity

Levels:

1. INFO
2. WARNING
3. ERROR
4. DEBUG
5. TRACE (most verbose)

Export to log file `export TF_LOG_PATH=/path/to/log.txt`. Unset env to disable logging.

## 8.3 Import

Datasources lets you read or reference resources TF doesn't have in its state file. But TF still doesn't manage it and can't update it. Must import resource for TF to manage it.

`terraform import resource_type.resource_name id`

eg. `terraform import aws_instance.webserver-2 i-062e3432sdfsdf`

Will see error when it is run because there is no config block for that resource. Create an empty resource block to hold the details so the error is not displayed

```terraform
resource "aws_instance" "webserver-2" {
    # resource arguments
}
```

Can get details to fill in resource with `terraform show resourcename` or by viewing the TF state file.

# 9. Terraform Modules

## 9.1 Modules
This makes TF looks for a dir in the parent dir containing the TF files like main.tf and variables.tf The source specifies the path to the child module.

```terraform
module "dev-webserver" {
    source = "../aws-instance"
}
```

If not specified, TF takes variable names and arguments from child module otherwise it takes from root module. This allows us to customise specific arguments for each module.

Can also use modules from registry

## 9.2 TF registry modules

This references the public TF registry [submodule here](https://registry.terraform.io/modules/terraform-aws-modules/iam/aws/latest/submodules/iam-user) and creates the IAM user max.

```terraform
module "iam_iam-user" {
  source  = "terraform-aws-modules/iam/aws//modules/iam-user"
  version = "3.4.0"
  # insert the 1 required variable here
  name = "max"
}
```

Check the [Inputs tab](https://registry.terraform.io/modules/terraform-aws-modules/iam/aws/latest/submodules/iam-user?tab=inputs) to see what resources will be created by default and disable if necessary.

# 10. Terraform functions

Use `terraform console` to evaluate [TF functions](https://www.terraform.io/language/functions) like

1. `file("/root/TF/main.tf")` 
2. `length(var.region)`
3. `toset(var.region)`
4. `max(var.num)`
5. `index(var.sf,"oni")` returns index of element "oni"

Use `terraform console` to check what functions evaluate to

```text
> var.cloud_users
andrew:ken:faraz:mutsumi:peter:steve:braja
> split(":", var.cloud_users)
[
  "andrew",
  "ken",
  "faraz",
  "mutsumi",
  "peter",
  "steve",
  "braja",
]
```

Console can also check resources

```text
> aws_iam_user.cloud[6]
{
  "arn" = "arn:aws:iam::000000000000:user/braja"
  "force_destroy" = false
  "id" = "braja"
  "name" = "braja"
  "path" = "/"
  "tags_all" = {}
  "unique_id" = "m6w6qzqytrvm1h6d6f53"
}
```

Can use conditionals to determine arguments

```terraform
resource "aws_instance" "mario_servers" {
  ami           = var.ami
  instance_type = var.name == "tiny" ? "t2.nano" : "t3.large"

  tags = {
    Name = var.name
  }
}
```

Then apply with `terraform apply -var name=tiny` to test

## 10.2 Terraform Workspaces

Workspaces segregate resources, similar to k8s namespaces. Default workspace is named **default**

Create workspace with `terraform workspace new ProjectA`

List workspaces with `terraform workspace list`

Switch workspace with `terraform workspace select ProjectA`

Reference `terraform.workspace` to select the current workspace in TF files.

State files stored in dir terraform.tfstate.d/ segregated by workspace names.

Example of invoking terraform.workspace in TF config files

```terraform
module "payroll_app" {
    source = "../modules/payroll-app"
    app_region = var.region[terraform.workspace]
    ami = var.ami[terraform.workspace]
}
```

