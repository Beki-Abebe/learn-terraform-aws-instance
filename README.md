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
operating system I'll show you for Linux OS. 

Print a colon-separated list of locations in your PATH.

`$ echo $PATH`

Move the Terraform binary to one of the listed locations. This command assumes that the binary is currently in your      
downloads folder and that your `PATH` includes `/usr/local/bin`, but you can customize it if your locations are   
different.

`$ mv ~/Downloads/terraform /usr/local/bin/`

For more detail about adding binaries to your `path`, see this Stack Overflow article.
### Verify the installation
Verify that the installation worked by opening a new terminal session and listing Terraform's available subcommands.

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
The AMI ID used in this configuration is specific to the us-west-2 region. If you would like to use a different region,    
see the Troubleshooting section for guidance.

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
  region  = "us-west-2"
}

resource "aws_instance" "app_server" {
  ami           = "ami-830c94e3"
  instance_type = "t2.micro"

  tags = {
    Name = "ExampleAppServerInstance"
  }
}
```
This is a complete configuration that you can deploy with Terraform. The following sections review each block of this     
configuration in more detail.

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
Use `resource` blocks to define components of your infrastructure. A resource might be a physical or virtual component  
such as an EC2 instance, or it can be a logical resource such as a Heroku application.  

Resource blocks have two strings before the block: the resource type and the resource name. In this example, the resource     
type is `aws_instance` and the name is `app_server`. The prefix of the type maps to the name of the provider. In the example    
configuration, Terraform manages the `aws_instance` resource with the `aws` provider. Together, the resource type and resource    
name form a unique ID for the resource. For example, the ID for your EC2 instance is `aws_instance.app_server`.

Resource blocks contain arguments which you use to configure the resource. Arguments can include things like machine sizes,   
disk image names, or VPC IDs. For your EC2 instance, the example configuration sets the AMI ID to an Ubuntu image, and the     
instance type to `t2.micro`, which qualifies for AWS free tier. It also sets a tag to give the instance a name.    

## Initialize the directory
When you create a new configuration — or check out an existing configuration from version control — you need to initialize    
the directory with `terraform init`.

Initializing a configuration directory downloads and installs the providers defined in the configuration, which in this case  
is the aws provider.

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

Terraform downloads the `aws` provider and installs it in a hidden subdirectory of your current working directory, named    
`.terraform`. The `terraform init` command prints out which version of the provider was installed. Terraform also creates  
a lock file named `.terraform.lock.hcl` which specifies the exact provider versions used, so that you can control when you  
want to update the providers used for your project.

## Format and validate the configuration

We recommend using consistent formatting in all of your configuration files. The `terraform fmt` command automatically  
updates configurations in the current directory for readability and consistency.  

Format your configuration. Terraform will print out the names of the files it modified, if any. In this case, your  
configuration file was already formatted correctly, so Terraform won't return any file names.  

`$ terraform fmt`

You can also make sure your configuration is syntactically valid and internally consistent by using the  
`terraform validate` command. Validate your configuration. The example configuration provided above is valid,  
so Terraform will return a success message.   

`$ terraform validate  
Success! The configuration is valid.`

## Create infrastructure
Apply the configuration now with the `terraform apply` command. Terraform will print output similar to what is shown below.  
We have truncated some of the output to save space.  
```
$ terraform apply

Terraform used the selected providers to generate the following execution plan.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_instance.app_server will be created
  + resource "aws_instance" "app_server" {
      + ami                          = "ami-830c94e3"
      + arn                          = (known after apply)
##...

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value:
```
- Tip If your configuration fails to apply, you may have customized your region or removed your default VPC.
Before it applies any changes, Terraform prints out the execution plan which describes the actions Terraform will take in
order to change your infrastructure to match the configuration.  

The output format is similar to the diff format generated by tools such as Git. The output has a `+` next to   
`aws_instance.app_server`, meaning that Terraform will create this resource. Beneath that, it shows the attributes that  
will be set. When the value displayed is (known after apply), it means that the value will not be known until the resource  
is created. For example, AWS assigns Amazon Resource Names (ARNs) to instances upon creation, so Terraform cannot know the  
value of the `arn` attribute until you apply the change and the AWS provider returns that value from the AWS API.  

Terraform will now pause and wait for your approval before proceeding. If anything in the plan seems incorrect or dangerous,  
it is safe to abort here before Terraform modifies your infrastructure.  

In this case the plan is acceptable, so type `yes` at the confirmation prompt to proceed. Executing the plan will take a few   
minutes since Terraform waits for the EC2 instance to become available.  
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
