# What is Infrastructure as Code with Terraform?
**Infrastructure as Code (IaC)** tools allow you to manage infrastructure with configuration files rather than through  
a graphical user interface. IaC allows you to build, change, and manage your infrastructure in a safe, consistent,   
and repeatable way by defining resource configurations that you can version, reuse, and share.

**Terraform** is HashiCorp's infrastructure as code tool. It lets you define resources and infrastructure in   
human-readable, declarative configuration files, and manages your infrastructure's lifecycle.

# Install Terraform
Retrieve the terraform binary by downloading a pre-compiled binary or compiling it from source.  
here is the link [Terraform](https://developer.hashicorp.com/terraform/install?product_intent=terraform)

To install `Terraform`, find the appropriate package for your system and download it as a zip archive. After  
downloading `Terraform`, unzip the package. `Terraform` runs as a single binary named terraform. Any other  
files in the package can be safely removed and `Terraform` will still function.  

Finally, make sure that the terraform binary is available on your `PATH`. This process will differ depending on your  
operating system I'll show you for Linux OS and Windows OS. 

## For Linux OS

Print a colon-separated list of locations in your PATH.

`$ echo $PATH`

Move the Terraform binary to one of the listed locations. This command assumes that the binary is currently in your      
downloads folder and that your `PATH` includes `/usr/local/bin`, but you can customize it if your locations are   
different.

`$ mv ~/Downloads/terraform /usr/local/bin/`

## For Windows
Extract it to a location like `C:\Terraform`. 

Then Add Terraform to System PATH

  - Open `Start Menu` and search for `Environment Variables`.
  - Click `Edit` the system environment variables.
  - In the System Properties window, click `Environment Variables`.
  - Under System variables, find and select `Path`, then click `Edit`.
  - Click `New`, then enter `C:\Terraform` (or the folder where you extracted Terraform).
  - Click `OK` to `save` and close the windows.  
    
### Verify the installation
Verify that the installation worked by opening a new terminal session and listing Terraform's available subcommands For Linux OS.

`$ terraform -help` 

you should be seeing  
```
Usage: terraform [-version] [-help] <command> [args]  

The available commands for execution are listed below.  
The most common, useful commands are shown first, followed by  
less common or more advanced commands. If you're just getting  
started with Terraform, stick with the common commands. For the  
other commands, please read the help and docs before usage.  
##...
```
Verify Installation worked For Windows OS Open Command Prompt (cmd) or PowerShell and run:  

`terraform -version`

If Terraform is installed correctly, it will display the version number.
# Build infrastructure

With Terraform installed, we are ready to create your first infrastructure. we will provision an EC2 instance on   
Amazon Web Services. EC2 instances are virtual machines running on AWS,and a common component of many   
infrastructure projects.  

**Prerequisites**  
*To follow this steps you will need:*
- The Terraform CLI (1.2.0+) installed.
- The AWS CLI installed.
- AWS account and associated credentials that allow you to create resources.
  
# Write configuration
The set of files used to describe infrastructure in Terraform is known as a Terraform configuration. we will write  
our first configuration to define a single AWS EC2 instance.

Each Terraform configuration must be in its own working directory. Let's Create a directory for our configuration.

`$ mkdir learn-terraform-aws-instance`

Change into the directory.

`$ cd learn-terraform-aws-instance`

Create a file to define your infrastructure.

`$ touch main.tf`

Open `main.tf` in your code editor, paste in the configuration below, and save the file.

- Tip
The AMI ID used in this configuration is specific to the us-west-2 region. If you would like to use a different
region, see the Troubleshooting section for guidance.  

```
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.16"
    }
  }

  required_version = ">= 1.2.0"
}

provider "aws" {
  region  = "us-east-1"
}

resource "aws_instance" "app_server" {
  ami           = "ami-0c02fb55956c7d316"
  instance_type = "t2.micro"

  tags = {
    Name = "ExampleAppServerInstance"
  }
}
```
This is a complete configuration that you can deploy with Terraform. The following sections review each block of    
this configuration in more detail.  

## Terraform Block
The `terraform {}` block contains Terraform settings, including the required providers Terraform will use to   
provision your infrastructure. For each provider, the `source` attribute defines an optional hostname,  
a namespace, and the provider type. Terraform installs providers from the Terraform Registry by default.  
In this example configuration, the `aws` provider's source is defined as `hashicorp/aws`, which is shorthand  
for `registry.terraform.io/hashicorp/aws`.      

You can also set a version constraint for each provider defined in the `required_providers` block. The   
`version` attribute is optional, but we recommend using it to constrain the provider version so that    
Terraform does not install a version of the provider that does not work with your configuration. If  
you do not specify a provider version, Terraform will automatically download the most recent version
during initialization.    

## Providers
The `provider` block configures the specified provider, in this case `aws`. A provider is a plugin that  
Terraform uses to create and manage your resources.   

You can use multiple provider blocks in your Terraform configuration to manage resources from different  
providers. You can even use different providers together. For example, you could pass the IP address of   
your AWS EC2 instance to a monitoring resource from DataDog.    

## Resources
Use `resource` blocks to define components of your infrastructure. A resource might be a physical or  
virtual component such as an EC2 instance, or it can be a logical resource such as a Heroku application.    

Resource blocks have two strings before the block: the resource type and the resource name. In this  
example, the resource type is `aws_instance` and the name is `app_server`. The prefix of the type maps  
to the name of the provider. In the example configuration, Terraform manages the `aws_instance` resource
with the `aws` provider. Together, the resource type and resource name form a unique ID for the resource.
For example, the ID for your EC2 instance is `aws_instance.app_server`.  

Resource blocks contain arguments which you use to configure the resource. Arguments can include things    
like machine sizes, disk image names, or VPC IDs. For your EC2 instance, the example configuration sets   
the AMI ID to an Ubuntu image, and the instance type to `t2.micro`, which qualifies for AWS free tier.   
It also sets a tag to give the instance a name.     

## Initialize the directory
When you create a new configuration — or check out an existing configuration from version control — you  
need to initialize the directory with `terraform init`.

Initializing a configuration directory downloads and installs the providers defined in the configuration,  
which in this case is the aws provider.  

Initialize the directory.
```
$ terraform init

Initializing the backend...

Initializing provider plugins...
- Finding hashicorp/aws versions matching "~> 4.16"...
- Installing hashicorp/aws v4.17.0...
- Installed hashicorp/aws v4.17.0 (signed by HashiCorp)

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

Terraform downloads the `aws` provider and installs it in a hidden subdirectory of your current working  
directory, named `.terraform`. The `terraform init` command prints out which version of the provider was  
installed. Terraform also creates a lock file named `.terraform.lock.hcl` which specifies the exact provider
versions used, so that you can control when you want to update the providers used for your project.  

## Format and validate the configuration

We recommend using consistent formatting in all of your configuration files. The `terraform fmt` command  
automatically updates configurations in the current directory for readability and consistency.    

Format your configuration. Terraform will print out the names of the files it modified, if any. In this   
case, your configuration file was already formatted correctly, so Terraform won't return any file names.    

`$ terraform fmt`

You can also make sure your configuration is syntactically valid and internally consistent by using the    
`terraform validate` command. Validate your configuration. The example configuration provided above is   
valid, so Terraform will return a success message.     

`$ terraform validate  
Success! The configuration is valid.`

## Create infrastructure
Apply the configuration now with the `terraform apply` command. Terraform will print output similar to  
what is shown below. We have truncated some of the output to save space.   
```
$ terraform apply

Terraform used the selected providers to generate the following execution plan.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_instance.app_server will be created
  + resource "aws_instance" "app_server" {
      + ami                          = "ami-0c02fb55956c7d316"
      + arn                          = (known after apply)
##...

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value:
```
- Tip If your configuration fails to apply, you may have customized your region or removed your
  default VPC. Before it applies any changes, Terraform prints out the execution plan which describes  
   the actions Terraform will take in order to change your infrastructure to match the configuration.    

The output format is similar to the diff format generated by tools such as Git. The output has a `+`   
next to `aws_instance.app_server`, meaning that Terraform will create this resource. Beneath that, it 
shows the attributes that will be set. When the value displayed is (known after apply), it means that the   
value will not be known until the resource is created. For example, AWS assigns Amazon Resource Names (ARNs) 
to instances upon creation, so Terraform cannot know the value of the `arn` attribute until you apply the 
change and the AWS provider returns that value from the AWS API.   

Terraform will now pause and wait for your approval before proceeding. If anything in the plan seems incorrect  
or dangerous, it is safe to abort here before Terraform modifies your infrastructure.  

In this case the plan is acceptable, so type `yes` at the confirmation prompt to proceed. Executing the plan will  
take a few minutes since Terraform waits for the EC2 instance to become available.  
```
  Enter a value: yes

aws_instance.app_server: Creating...
aws_instance.app_server: Still creating... [10s elapsed]
aws_instance.app_server: Still creating... [20s elapsed]
aws_instance.app_server: Still creating... [30s elapsed]
aws_instance.app_server: Creation complete after 36s [id=i-01e03375ba238b384]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```
You have now created infrastructure using Terraform! Visit the EC2 console and find your new EC2 instance.
Once you no longer need infrastructure, you may want to destroy it to reduce your security exposure and costs.

`terraform destroy`

# Set the instance name with a variable

The current configuration includes a number of hard-coded values. Terraform variables allow you to write  
configuration that is flexible and easier to re-use.  

Add a variable to define the instance name.  

Create a new file called `variables.tf` with a block defining a new `instance_name` variable.  

```
variable "instance_name" {
  description = "Value of the Name tag for the EC2 instance"
  type        = string
  default     = "ExampleAppServerInstance"
}
```
In `main.tf`, update the `aws_instance` resource block to use the new variable. The `instance_name`  
variable block will default to its default value ("ExampleAppServerInstance") unless you    
declare a different value.  

```
 resource "aws_instance" "app_server" {
   ami           = "ami-0c02fb55956c7d316"
   instance_type = "t2.micro"

   tags = {
-    Name = "ExampleAppServerInstance"
+    Name = var.instance_name
   }
 }
```
## Apply your configuration

Apply the configuration. Respond to the confirmation prompt with a `yes`.  
```
$ terraform apply

Terraform used the selected providers to generate the following execution plan.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_instance.app_server will be created
  + resource "aws_instance" "app_server" {
      + ami                          = "ami-0c02fb55956c7d316"
      + arn                          = (known after apply)
##...

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_instance.app_server: Creating...
aws_instance.app_server: Still creating... [10s elapsed]
aws_instance.app_server: Still creating... [20s elapsed]
aws_instance.app_server: Still creating... [30s elapsed]
aws_instance.app_server: Still creating... [40s elapsed]
aws_instance.app_server: Creation complete after 50s [id=i-0bf954919ed765de1]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```
Now apply the configuration again, this time overriding the default instance name by passing in    
a variable using the `-var` flag. Terraform will update the instance's Name tag with the new    
`name`. Respond to the confirmation prompt with `yes`. 

```
$ terraform apply -var "instance_name=YetAnotherName"
aws_instance.app_server: Refreshing state... [id=i-0bf954919ed765de1]

Terraform used the selected providers to generate the following execution plan.
Resource actions are indicated with the following symbols:
  ~ update in-place

Terraform will perform the following actions:

  # aws_instance.app_server will be updated in-place
  ~ resource "aws_instance" "app_server" {
        id                           = "i-0bf954919ed765de1"
      ~ tags                         = {
          ~ "Name" = "ExampleAppServerInstance" -> "YetAnotherName"
        }
        # (26 unchanged attributes hidden)




        # (4 unchanged blocks hidden)
    }

Plan: 0 to add, 1 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_instance.app_server: Modifying... [id=i-0bf954919ed765de1]
aws_instance.app_server: Modifications complete after 7s [id=i-0bf954919ed765de1]

Apply complete! Resources: 0 added, 1 changed, 0 destroyed.
```
Setting variables via the command-line will not save their values. Terraform supports many ways  
to use and set variables so you can avoid having to enter them repeatedly as you execute commands.

# Query data with outputs

Create a file called `outputs.tf` in your `learn-terraform-aws-instance` directory.

Add the configuration below to `outputs.tf` to define outputs for your EC2 instance's ID and  
IP address.  

```
output "instance_id" {
  description = "ID of the EC2 instance"
  value       = aws_instance.app_server.id
}

output "instance_public_ip" {
  description = "Public IP address of the EC2 instance"
  value       = aws_instance.app_server.public_ip
}
```
## Inspect output values

You must apply this configuration before you can use these output values. Apply your configuration   
now. Respond to the confirmation prompt with `yes`

```
$ terraform apply
aws_instance.app_server: Refreshing state... [id=i-0bf954919ed765de1]

Changes to Outputs:
  + instance_id        = "i-0bf954919ed765de1"
  + instance_public_ip = "54.186.202.254"

You can apply this plan to save these new output values to the Terraform state,
without changing any real infrastructure.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes


Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

Outputs:

instance_id = "i-0bf954919ed765de1"
instance_public_ip = "54.186.202.254"
```
Terraform prints output values to the screen when you apply your configuration.  
Query the outputs with the `terraform output` command.  

```
$ terraform output
instance_id = "i-0bf954919ed765de1"
instance_public_ip = "54.186.202.254"

```
Destroy your infrastructure. Respond to the confirmation prompt with yes.

`
$ terraform destroy
`

