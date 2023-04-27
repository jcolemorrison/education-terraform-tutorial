# Getting Started with Terraform

Terraform enables you to define, provision, and manage your infrastructure as code (IaC).  This lets you manage resources in a codified approach that allows for human readability, auditable workflows, and integration with code tooling such as version control systems.  You can read more about the concepts of IaC in the tutorial [What is Infrastructure as Code with Terraform?](https://developer.hashicorp.com/terraform/tutorials/gcp-get-started/infrastructure-as-code).

## Prerequisites

This tutorial assumes that you have the following: 

- [Terraform CLI](https://developer.hashicorp.com/terraform/tutorials/docker-get-started/install-cli) version 0.14+ installed locally.
- [Docker](https://www.docker.com/products/docker-desktop/) installed locally and running.

You can find complete instructions on installing both of the above in the [Install Terraform](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli) tutorial.

## Create the Project Directory

Create a new directory on your local machine for Terraform configuration:  

```shell
$ mkdir terraform-demo && cd terraform-demo
```

You will only affect configuration within the directory where you run Terraform commands.  This is the main working directory of your Terraform project is the [root module](https://developer.hashicorp.com/terraform/language/modules#the-root-module).

## Create the Configuration File

Next, create a file for your Terraform configuration:

```shell
$ touch main.tf
```

You can structure Terraform modules and configuration files in a variety of ways.  For more information on best practices, read our documentation on [Standard Module Structure](https://developer.hashicorp.com/terraform/language/modules/develop/structure).

## Write the Configuration

Paste the following lines into your `main.tf` file:

```hcl
terraform {
  required_providers {
    docker = {
      source = "kreuzwerker/docker"
    }
  }
}

provider "docker" {
    host = "unix:///var/run/docker.sock"
}

resource "docker_image" "nginx" {
  name = "nginx:latest"
}

resource "docker_container" "nginx" {
  image = docker_image.nginx.image_id
  name  = "training"
  ports {
    internal = 80
    external = 80
  }
}
```

You define Terraform configuration using the [HashiCorp Configuration Language (HCL) syntax](https://developer.hashicorp.com/terraform/language/syntax/configuration). Each stanza in the example configuration (`terraform`, `resource` and `provider`) represent [blocks](https://developer.hashicorp.com/terraform/language/syntax/configuration#blocks).  Each block contain [arguments](https://developer.hashicorp.com/terraform/language/syntax/configuration#arguments) that configure the respective block.

### Terraform Block

The `terraform` block configures Terraform's settings, which lets you specify the Terraform version, required providers, backends, and more. The following configuration tells Terraform to use the `docker` provider.

```hcl
terraform {
  required_providers {
    docker = {
      source = "kreuzwerker/docker"
    }
  }
}
```

For more information, read the [Terraform Settings](https://developer.hashicorp.com/terraform/language/settings) documentation.


### Provider Block

The `provider` block configures the provider. This often includes authentication information that lets the provider interact with the underlying cloud and SaaS provider APIs. The following configuration configures the `docker` provider so Terraform can interact with your local Docker installation.

```hcl
provider "docker" {
    host = "unix:///var/run/docker.sock"
}
```

For more information, read the [Providers](https://developer.hashicorp.com/terraform/language/providers) documentation. Visit our [Terraform Registry](https://registry.terraform.io/browse/providers) for a full list of Terraform providers.

### Resource Block

Use `resource` blocks to describe infrastructure components made available for management through providers specified in your `terraform` block.  The resources you create are dependent upon which providers you have installed.

```hcl
resource "docker_image" "nginx" {
  name = "nginx:latest"
}

resource "docker_container" "nginx" {
  image = docker_image.nginx.image_id
  name  = "training"
  ports {
    internal = 80
    external = 80
  }
}
```

In this configuration we specify two resources:

1. `docker_image` specifies an Nginx docker image with the "latest" tag.  Because no additional settings were passed to the docker `provider`, docker will pull the image from Docker Hub.
2. `docker_container` defines the arguments that the container uses including image created from the `docker_image` resource.

For more information, read the [Resources](https://developer.hashicorp.com/terraform/language/resources) and [Docker provider](https://registry.terraform.io/providers/kreuzwerker/docker/latest/docs) documentation.

## Initialize the Working Directory

Ensure you are in the working directory `terraform-demo` and run the `init` command:  

```shell
$ terraform init

Initializing the backend...

Initializing provider plugins...
- Finding latest version of kreuzwerker/docker...
- Installing kreuzwerker/docker v3.0.2...
- Installed kreuzwerker/docker v3.0.2 (self-signed, key ID BD080C4571C6104C)

Partner and community providers are signed by their developers.
If you'd like to know more about provider signing, you can read about it here:
https://www.terraform.io/docs/cli/plugins/signing.html

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

The `init` command downloads and installs any specified providers and stores them in a hidden `.terraform` directory.  Terraform also creates a dependency lock file called `.terraform.lock.hcl` which stores versioning information about provider dependencies.  

For more information, read the [Dependency Lock Files](https://developer.hashicorp.com/terraform/language/files/dependency-lock) documentation.

You shoud check for any errors. If it ran successfully, provision the resource with the `apply` command.

```shell
$ terraform apply
```

The command will take up to a few minutes to run and will display a message indicating that the resource was created.

Finally, destroy the infrastructure.

```shell
$ terraform destroy
```

Look for a message are the bottom of the output asking for confirmation. Type `yes` and hit ENTER. Terraform will destroy the resources it had created earlier.