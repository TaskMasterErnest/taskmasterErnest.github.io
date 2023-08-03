# A Primer on Getting (UN)Comfortable with Terraform

*You know what they say about Terraform right? You can change the world with it. Lol*

**No?**

![jake-cricket-meme](/assets/jake-tapper-jake-tapper-crickets.gif)

Well let's get on with it then shall we?

## Overview
Terraform is an infrastructure provisioning tool. 
- It sets up all the infrastructure needed to be used in order to complete an application setup.
- It is quite different from Ansible, such that Terraform is mostly concerned about setting up the infrastructure needed to be configured for the application's environment. Ansible is more concerned about configuring the environment for the application to run on. ie what kind of software or tools need to be present in order to successfully start the application. 
	- kind of like a builder (Terraform) builds a house and an Interior designer (Ansible) decorates the interior to be hospitable.
- Terraform works with declarative language, meaning that a user tells it what to do and Terrraform figures out how to make it happen for the user.
	- It uses declarative syntax also, meaning the end result of the user's desires is specified and Terraform works to generate that result desired by the user.
- Terraform is used in these three ways:
	- create infrastructure
	- make changes to the created infrastructure
	- replicate that infrastructure
- Terraform has a central processing unit called the CORE. It is where the the configuration files that are to be processed are parsed. It is the engine that communicates with providers to get the work done. It is the engine used to process the commands passed to Terraform to execute.
	- the CORE works with two things; the `terraform config` and  `providers`. These tow things to Terraform are the input sources for the CORE.
	- the terraform config is what the user's desired result parsed into Terraform
	- the providers are platforms that Terraform connects to in order to generate the desired result for the user.
	- the CORE reconciles the terraform config and the providers; where the user's desired result matches the state in the provider.
- Terraform works with platforms, infrastructure providers and services in order to achieve that desired state. These are called 'Providers'.
	- the provider is essentially a plugin that allows terraform to connect to a specific platform and provision resources on it to match the desired state. 
	- examples are Kubernetes, AWS, Azure, GCP etc.
- In terms of working with infrastructure, Terraform shines when used to update existing infrastructure that when used to initially provision it.
	- it keeps track of the state, ie a file that stores information about the resources that have been provisioned by Terraform. (plays into the concept of GitOps and Infrastructure as Code.)
- Terraform has these basic commands which essentially work together to generate the user's desired result.
	- `refresh` - this is used to query a provider to get the current state of all resources managed by Terraform and update the Terraform state.
	- `plan` - this is used to create an execution plan ie. the actions Terraform must take in order to get the desired state.
	- `apply` - this is to execute the stages/plans that are guaranteed to get to the desired state.
	- `destroy` - this is to remove all resources provisioned by Terraform. aka remove all components of the desired state.

---

## Installing Terraform
Terraform can be installed locally or remotely depending on the environment to work in.
It is available for Linux, and MacOS.

---

### Learn by Example
Learn how to connect to AWS and provisioning some services/resources on the AWS account using Terraform.
In here, we are going to:
1. connect to an AWS Account
2. create a VPC.
3. create a subnet.

Terraform has a Registry that has all the providers that are used to connect to these resources.
It is well-documented and has better documentation with all the code examples needed to get started with these technologies.
The provider can be likened to dependencies in code, they are not directly installed in the code but in the environment in which the code is ran, they are specified and the code goes to download them to make then available for use.
When the provider is initialized, it gives the user full access to the API of that technology (everything that the provider is willing to let communicate with Terraform).

- Create a `main.tf` file, this will house all the Terraform code we will write.

```hcl
provider "aws" {
	region = "eu-west-2"
	# access_key = "access key"
	# secret_key = "secret key"
}

variable "dev_cidr_block" {
	description = "development cidr block"
	default = "10.0.0.0/16
}

variable "environment" {
	description = "environment to deploy in"
}

resource "aws_vpc" "development-vpc" {
	cidr_block = var.dev_cidr_block
	tags = {
		Name: var.environment
		vpc_env: "dev"
	}
}

variable "subnet_cidr_blocks" {
	description = "a collection of CIDR blocks for the subnets"
	type = list(string)
}

# referencing a custom env-var
variable "availability_zone" {}

resource "aws_subnet" "dev-subnet-1" {
	vpc_id = aws_vpc.development-vpc.id # to get the vpc ID
	cidr_block = var.subnet_cidr_blocks[0]
	availability_zone = var.availability_zone
	tags = {
		Name: "subnet-1-dev" # the default tag for name is Name
	}
}

data "aws_vpc" "existing_vpc" {
	default = true
}

resource "aws_subnet" "dev-subnet-2" {
	# getting the ID of the existing VPC
	vpc_id = data.aws_vpc.existing_vpc.id
	# in this case, the default VPC has its own IP address range it uses
	# the default VPC also has subnets, so the best option is to choose an IP range that is not already existing.
	cidr_block = var.subnet_cidr_block[1]
	availability_zone = "eu-west-2a"
	tags = {
		Name: "subnet-2-default"
	}
}

variable "staging_values" {
	description = "contains variables for the name and cidr_blocks for staging subnet"
}

resource "aws_subnet" "staging-subnet-1" {
	cidr_block = var.staging_values[0].cidr_block
	tags = {
		Name:	var.staging_values[0].name
	}
}

output "dev-subnet-id" {
	value = aws_vpc.development-vpc.id
}

output "aws-subnet-id" {
	value = aws_subnet.dev-subnet-2.id
}

```
-  the `terraform-dev.tfvars` file / the `terraform.tfvars` file.
```JSON
dev_cidr_block = "10.0.10.0/16"
environment = "development"
subnet_cidr_blocks = ["10.0.20.0/24", "172.31.48.0/20"]
staging_values = [
	{cidr_block = "10.0.56.0/20", name = "staging"}
]
```

- Initialize the Terraform file with `terraform init`, ie. download the provider that is going to be used in the Terraform code. 
	- two new hidden file(s) and directories are provided: `.terraform/providers/` and a `.terraform.lock.hcl`. These contain all information regarding the provider as it is being used. 
	- the `.terraform/providers/` stores the provider plugins.
	- the `.terraform.lock.hcl` is a lock file that records the specific versions of the provider's dependencies and modules.
- In order to use a resource by a provider, Terraform has a way to let the user write that definition; `resource "<provider-name>_<provider-resource>" "user-defined-name" {attributes}`.
	- this same state, to obtain an attribute from another resource that is yet to be created, Terraform uses this syntax, `<provider>.<attribute-name>.<resource to reference from attribute>`.
	- the attributes are documented in the Terraform docs for the provider and show exactly what to write to use an attribute.
	- tags can also be applied to resources in order to give them an identity, in the above definition, the user-defined-name can be applied to a tag and that will persist the name and the resource created will have that specific tag as its actual name when activated.
- To provision the resources, run the `terraform apply` command. 
	- it gives a breakdown of the attributes to be provisioned by the resource and asks yo to confirm in order to apply them.
- Terraform provides `data sources`. This `data source` is a way to get information from external systems or services and make them available within a Terraform configuration. 
	- this allows data to be fetched for use in the Terraform configuration.
	- it differs from `resource` as resource is used to create a resource and `data` is used to query an import and already existing resource and use that data within the configuration.
	- it uses this syntax `data "<provider-name>_<resource-to-query>" "name-of-existing-resource" { attributes }`.
	- in order to reference a data source and use it, we put `data.` in front of the resource attributes we want.

## More Stuff to Know
### Changing/Destroying a Resource
- Before getting to know which resources to use, we have to get the resources to have a name / tags that can make them easily identifiable when they are provisioned.
	- To do this, we add a parameter called tags. These tags are in a `key:value` pair. Tags are provisioned in this way `tags = { key:value }` in the resource definition that we want.
- To destroy Terraform configurations, there are two ways to do this
	1. deleting / removing the record of the resource from the configuration file; and then applying the new configuration using `terraform apply`.
	2. targeting the resource to remove by taking the resource-type and the user-defined name and using the `terraform destroy -target resource-type.user-defined-name` command to delete/destroy it. eg `terraform destroy -target aws_subnet.dev-subnet-2`.
	- The best way is to use the first method, because this makes the configuration file match with the state file that is stored.

### Terraform Commands
- `terraform plan` : this command is to check the difference between the current state and the desired state. It show you what will be configured newly as against the old configuration. No need to use the `apply` command to check the new configuration.
- `--auto-approve`: this is a flag added to the `terraform apply` command to skip the whole process of manually showing changes and approving those changes in configuration. It goes straight to provision your resources.
- `terraform destroy`: deletes / removes ALL the resources that are in the configuration file. 

### Terraform State
- The Terraform state is a collection of information about the resources provisioned, their configurations and the relationship between these resources as specified in the infrastructure-as-code file.
- The state file is JSON file that contains the information about the resources and their inter-relationship as specified in the main file.
	- It is considered a private API for use internally by Terraform to compare the current and desired states.
	- It should not be tampered with in any way by the user, it is for Terraform to manipulate.
- There is also a backup state file; it stores the previous state after changes have been made and applied to main Terraform file.
- The user has some commands available to communicate with the state. This command has the preamble `terraform state`. 
	- It has sub-commands that can be used to interact with the state file and/or the state on the provider itself.

### Terraform Outputs
- Terraform outputs are used to show specific outputs that the user desires to see after the configuration has been applied.
	- it is written as `value = <resource-type>.<user-defined-name>.attribute`.
	- the `attribute` can either be a name or id of a resource as it has been given by the provider.
	- the attributes can be gotten for each resource by running the `terraform plan` command and looking through the list of attributes that the provider has for that resource.
- The outputs can only be a single one per definition in the configuration file.

### Terraform Variables
- Terraform variables are just like normal variables that are used to denote dynamic elements in code.
- In essence, they are placeholder elements to be used in referencing dynamic values to be passed to resources in the configuration file.
- They have their own block with the format `variable "user-defined-name-of-variable" { description }`
- Variables by convention have to be defined in a separate file that can be referenced by the main file when applying the configuration.
	- the default name for the variables file is `terraform.tfvars`. All variables have to be specified here; the variables block has to be in the main file. The `tfvars` extension tells Terraform that this is a variable file it can use.
	- if the name of the file is changed from the default name, `terraform-dev.tfvars`, then the variables file has to be passed as a parameter when applying the configuration. `terraform apply -var-file terraform-dev.tfvars`.
		- a usefulness of having variables is that the variables can be changed accordingly when the configuration is to be applied for different environments but using the same configuration.
- There can be `default` values for variables. These values are what are used in case no other values are defined for the variable. In the variable block, the syntax for them is `default = <variable-placement>`.
- The variables can also be set to be strictly of a `type`. The type locks down the way the variable is defined as per that the user prefers and what the resource takes. The syntax in the variable block is `type = <kind-of-type>`.
	- There are many of `type`; it could be a string, bool or number.
- There can also be multiple values defined to be used by the variable. These are stated in the type by data types (lol), eg,string, array etc.
	- they are stated with the type given, syntax is `type = <data-type>(<type>)`.
	- these can be referenced in the main configuration file with the position of the values defined in the variables file.
	- An example will be using objects, a list of them to specify variables to be used in a configuration file. Look at `staging_values` and the `staging-subnet-1` resource to see how this is done.
	- the usage of a type is that, it can be used to validate the object type that should be passed through it. To strictly guide a third-person on what values to use when inputting into the variable.
```JSON
  variable "subnet_cidr_block" {
	description: "CIDR blocks for VPC and subnets"
	# here we validate the data that should be passed in.
	# here, we specify that the data type of the values should be a string.
	type = list(object({
		cidr_block = string
		name = string
	}))
  }
```

### Terraform Environment Variables
- Terraform environment variables are a way to define variables that should be picked up or passed into Terraform.
- Let's say we want to deploy infrastructure with AWS, instead of hardcoding the ACCESS_KEY_ID and the SECRET_ACCESS_KEY_ID values into the template, we can make use of two options: using local env-vars or using the authentication method provided by the provider.
	- for AWS, you can set the secret keys as env-vars by exporting them, eg `export AWS_SECRET_ACCESS_KEY_ID=secret-access-key` and `export AWS_ACCESS_KEY_ID=access-key-id`. When set in this manner, Terraform will automatically pick it up.
	- the other way is to set the variables in the AWS CLI for Terraform to use. Here, you run the `aws configure` command, set the access-key and secret-access-key and default region. It is populated in the `~/.aws/credentials` file and Terraform can automatically pick it from there.
- The user can set custom variables to be used in Terraform. They are set as local env-vars and referenced in Terraform. The way to set them is this, `export TF_VAR_name-of-env-var="env-var-name"`. Let's reference the availability zone for dev-subnet-1 through a custom variable. Let's set is as `export TF_VAR_availability_zone="eu-west-2b"`.

## Terraform in Git
- After writing the Terraform file, you may want to check it into a Git repository for the following benefits:
	1. safekeeping.
	2. having a history of changes.
	3. sharing with others to collaborate on.
	4. for reviewing infrastructure changes using merge requests.
- To get to store this in a Git repo like GitHub, you have to initialize Git in the directory the Terraform file resides.
- After initialization, you have to ignore the pushing of some files to the public repo. These files either contain sensitive data or are large files to send.
- A typical `.gitignore` file for Terraform will be:

```text
# everything in the .terraform directory
.terraform/*

# the state files, main and backup
*.tfstate
*.tfstate.*

# the terraform variables file
*.tfvars
```

Now you are ready to go change the world with Terraform...

![lets-go-fishing](https://th.bing.com/th/id/R.a801c5e00d5fb915e939a5c30385b941?rik=7ygoHJl4WuKeJQ&riu=http%3a%2f%2fwww.animatedimages.org%2fdata%2fmedia%2f157%2fanimated-fishing-image-0131.gif&ehk=rQq%2bd2QDkwjFPyX1hPoyHfP%2bFC7iis9SSANWH6%2bO9dk%3d&risl=&pid=ImgRaw&r=0)
