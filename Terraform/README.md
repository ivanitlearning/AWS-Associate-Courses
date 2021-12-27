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

