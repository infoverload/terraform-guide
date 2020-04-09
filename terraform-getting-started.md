# Getting Started with Terraform

Terraform is a popular tool for defining and provisioning infrastructure across multiple providers. The goal is to achieve a consistent and predictable environment by declaring what you want in code. Your entire infrastructure can be defined in a few configuration files using an intuitive configuration language. This can then be stored in source control. There are numerous benefits to the Infrastructure as Code (IaC) model including automated deployments, repeatable processes with reusable components, and documented architecture.  

To install Terraform, visit [Terraform.io](https://www.terraform.io/downloads.html), find the appropriate package for your system and download the binary. After downloading Terraform, unzip the package. Terraform runs as a single binary named "terraform".  Make sure that the binary is available on your system's [PATH](https://superuser.com/questions/284342/what-are-path-and-other-environment-variables-and-how-can-i-set-or-use-them) and verify the installation by typing `terraform` in your terminal prompt. You should see a list of Terraform commands and their description. 

Now you can start provisioning infrastructure. In this guide, you will get hands-on experience with Terraform by provisioning an nginx Docker container. You will create a basic configuration file, initialize the configuration, create the desired resources, and then destroy them. 

## Prerequisites

In order to successfully complete this guide, you need to install:

- Terraform v0.12.24 or later 
- [Docker](https://docs.docker.com/get-docker/)

## Create Terraform configuration files

The set of files used to describe infrastructure in Terraform's declarative model is known as a Terraform configuration. You describe the desired state of your infrastructure in these files. Then Terraform and the provider will plan how to configure the provider to match the desired state. 

You will now write a configuration to launch a Docker container with [nginx](https://www.nginx.com) running in it. 

Terraform configuration files should be in its own directory. Create a directory for the new configuration and change into the directory.

```shell
$ mkdir terraform-demo
$ cd terraform-demo
```

The deployed configuration will be contained in one or more Terraform files which typically have the file extension `.tf`. Terraform will detect all files in the working directory that end with `.tf` once you initialize and apply these states. Create a file for your Terraform configuration code in the `terraform-demo` directory.

```shell
$ touch main.tf
```

The format of the configuration files is [documented here](https://www.terraform.io/docs/configuration/index.html). Paste the configuration below into `main.tf` and save it. 

```hcl
# Configure the Docker provider
provider "docker" {
    host = "unix:///var/run/docker.sock"
}

# Create a container
resource "docker_container" "nginx" {
  image = docker_image.nginx.latest
  name  = "training"
  ports {
    internal = 80
    external = 80
  }
}

# Create an image
resource "docker_image" "nginx" {
  name = "nginx:latest"
}
```

The configuration file is using the HashiCorp Configuration Language ([HCL](https://github.com/hashicorp/hcl)), which allows you to define variables to store information. The file is made up of resources with settings and values representing the desired state of your infrastructure.

Here, you are defining the Docker provider with host properties, a Docker container resource, and a Docker image resource. 

A provider is a plugin that Terraform uses to translate API interactions with the service. It is responsible for creating and managing resources. In this case, the Docker provider interacts with Docker containers and images. It uses the Docker API to manage the lifecycle of Docker containers. You can even use resources from multiple providers together via multiple provider blocks. Check out the list of available providers [here](https://www.terraform.io/docs/providers/index.html).

The resource block has two strings before the block which designates the resource type and the resource name, respectively. The prefix of the resource type maps to the provider name. The properties for the resource are within the resource block. You can learn more about resources [here](https://www.terraform.io/docs/configuration/resources.html).


## Initialize your Terraform configuration

Terraform supports numerous infrastructure and service providers through its plugin-based model. If Terraform wants to talk to a provider and provision it's resources, it uses it's plugin.  When you have a new configuration or have checked out an existing configuration from version control, you need to initialize the configuration and download and install the necessary plugins. This is done by executing the `init` [command](https://www.terraform.io/docs/commands/init.html) in the project directory.

```shell
$ terraform init
```

You should see output similar to this:

```output
Initializing the backend...

Initializing provider plugins...
- Checking for available provider plugins...
- Downloading plugin for provider "docker" (terraform-providers/docker) 2.7.0...

The following providers do not have any version constraints in configuration,
so the latest version was installed.

To prevent automatic upgrades to new major versions that may contain breaking
changes, it is recommended to add version = "..." constraints to the
corresponding provider blocks in configuration, with the constraint strings
suggested below.

* provider.docker: version = "~> 2.7"

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.

```
The output shows which version of the plugin was installed. You should check for any errors. 

Terraform downloads the Docker provider and installs it in a hidden subdirectory of the current working directory called `.terraform` . Subsequent commands will use local settings and data that are initialized by `terraform init`.

## Apply your Terraform configuration

> Note: The commands shown in this guide apply to Terraform v0.12 and above. Earlier versions require using the `terraform plan` command to see the execution plan before applying it. Use `terraform version` to confirm your running version.

Now you can provision the resources specified in your configuration by creating and executing a plan. Terraform looks at the existing environment, looks at what you want to do, and figures out how to make reality match with your configuration. In the same directory as the `main.tf` file you created, run the following command to [apply](https://www.terraform.io/docs/commands/apply.html) your configuration:

```shell
$ terraform apply
```

This will generate and show you a plan, which tells you exactly what it will do before it does it. You should see output similar to below.

```output
An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # docker_container.nginx will be created
  + resource "docker_container" "nginx" {
      + attach           = false
      + bridge           = (known after apply)
      + command          = (known after apply)
      + container_logs   = (known after apply)
      + entrypoint       = (known after apply)
      + env              = (known after apply)
      + exit_code        = (known after apply)
      + gateway          = (known after apply)
      + hostname         = (known after apply)
      + id               = (known after apply)
      + image            = (known after apply)
      + ip_address       = (known after apply)
      + ip_prefix_length = (known after apply)
      + ipc_mode         = (known after apply)
      + log_driver       = "json-file"
      + logs             = false
      + must_run         = true
      + name             = "training"
      + network_data     = (known after apply)
      + read_only        = false
      + restart          = "no"
      + rm               = false
      + shm_size         = (known after apply)
      + start            = true

      + labels {
          + label = (known after apply)
          + value = (known after apply)
        }

      + ports {
          + external = 80
          + internal = 80
          + ip       = "0.0.0.0"
          + protocol = "tcp"
        }
    }

  # docker_image.nginx will be created
  + resource "docker_image" "nginx" {
      + id     = (known after apply)
      + latest = (known after apply)
      + name   = "nginx:latest"
    }

Plan: 2 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: 
```

The `+` sign indicates what will be created. When the value displayed is `(known after apply)`, it means that the value won't be known until the resource is created.

If the plan seems incorrect, it is safe to abort here with no changes made to your infrastructure. In this case, you can type "yes" at the confirmation prompt to proceed. The command may take up to a few minutes to run.

You should see output similar to below.

```output
docker_image.nginx: Creating...
docker_image.nginx: Still creating... [10s elapsed]
docker_image.nginx: Creation complete after 15s [id=sha256:ed21b7a8aee9cc677df6d7f38a641fa0e3c05f65592c592c9f28c42b3dd89291nginx:latest]
docker_container.nginx: Creating...
docker_container.nginx: Creation complete after 1s [id=c218448af8ae779372bd3c02d87d711965188c14054a87a15af0a8b28e419abc]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```

If `terraform apply` failed with an error, read the error message and try to fix the error that occurred. 

Terraform stores the current [state of the configuration](https://www.terraform.io/docs/state/index.html) in a Terraform state file (`.tfstate`), which is stored locally by default. This file keeps track of all the resource IDs and is very important for Terraform to manage and update your infrastructure. 


## Verify your provisioned resources

Now that Terraform has provisioned your nginx container, you can verify that it is running by inspecting the Terraform state.  In the project directory, run the following command:

```shell
$ terraform show
```

You should see output similar to below.

```output
# docker_container.nginx:
resource "docker_container" "nginx" {
    attach            = false
    command           = [
        "nginx",
        "-g",
        "daemon off;",
    ]
    cpu_shares        = 0
    entrypoint        = []
    env               = [
        "NGINX_VERSION=1.17.9",
        "NJS_VERSION=0.3.9",
        "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
        "PKG_RELEASE=1~buster",
    ]
    gateway           = "172.17.0.1"
    hostname          = "c218448af8ae"
    id                = "c218448af8ae779372bd3c02d87d711965188c14054a87a15af0a8b28e419abc"
    image             = "sha256:ed21b7a8aee9cc677df6d7f38a641fa0e3c05f65592c592c9f28c42b3dd89291"
    ip_address        = "172.17.0.2"
    ip_prefix_length  = 16
    ipc_mode          = "private"
    log_driver        = "json-file"
    logs              = false
    max_retry_count   = 0
    memory            = 0
    memory_swap       = 0
    must_run          = true
    name              = "training"
    network_data      = [
        {
            gateway          = "172.17.0.1"
            ip_address       = "172.17.0.2"
            ip_prefix_length = 16
            network_name     = "bridge"
        },
    ]
    network_mode      = "default"
    privileged        = false
    publish_all_ports = false
    read_only         = false
    restart           = "no"
    rm                = false
    shm_size          = 64
    start             = true

    labels {
        label = "maintainer"
        value = "NGINX Docker Maintainers <docker-maint@nginx.com>"
    }

    ports {
        external = 80
        internal = 80
        ip       = "0.0.0.0"
        protocol = "tcp"
    }
}

# docker_image.nginx:
resource "docker_image" "nginx" {
    id     = "sha256:ed21b7a8aee9cc677df6d7f38a641fa0e3c05f65592c592c9f28c42b3dd89291nginx:latest"
    latest = "sha256:ed21b7a8aee9cc677df6d7f38a641fa0e3c05f65592c592c9f28c42b3dd89291"
    name   = "nginx:latest"
}
```

You can also check the running Docker container through the Docker API. Running `docker ps` should give you an output similar to the following.

```
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
6a1093b14179        ed21b7a8aee9        "nginx -g 'daemon of…"   15 seconds ago      Up 13 seconds       0.0.0.0:80->80/tcp   training
```

You can also visit `localhost` in your web browser or use the `cURL` utility. Running `curl localhost` should give you the starter page for nginx like below. 

```output
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

## Destroy your Terraform configuration

If you are finished testing or don't want to keep your configuration, you can destroy it with the following command.

```shell
$ terraform destroy
```

Terraform looks at the state file and what resources were created.  It will list everything that will be destroyed and you should see an output similar to below. 

```
docker_image.nginx: Refreshing state... [id=sha256:ed21b7a8aee9cc677df6d7f38a641fa0e3c05f65592c592c9f28c42b3dd89291nginx:latest]
docker_container.nginx: Refreshing state... [id=c218448af8ae779372bd3c02d87d711965188c14054a87a15af0a8b28e419abc]

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # docker_container.nginx will be destroyed
  - resource "docker_container" "nginx" {
      - attach            = false -> null
      - command           = [
          - "nginx",
          - "-g",
          - "daemon off;",
        ] -> null
      - cpu_shares        = 0 -> null
      - dns               = [] -> null
      - dns_opts          = [] -> null
      - dns_search        = [] -> null
      - entrypoint        = [] -> null
      - env               = [
          - "NGINX_VERSION=1.17.9",
          - "NJS_VERSION=0.3.9",
          - "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
          - "PKG_RELEASE=1~buster",
        ] -> null
      - gateway           = "172.17.0.1" -> null
      - group_add         = [] -> null
      - hostname          = "c218448af8ae" -> null
      - id                = "c218448af8ae779372bd3c02d87d711965188c14054a87a15af0a8b28e419abc" -> null
      - image             = "sha256:ed21b7a8aee9cc677df6d7f38a641fa0e3c05f65592c592c9f28c42b3dd89291" -> null
      - ip_address        = "172.17.0.2" -> null
      - ip_prefix_length  = 16 -> null
      - ipc_mode          = "private" -> null
      - links             = [] -> null
      - log_driver        = "json-file" -> null
      - log_opts          = {} -> null
      - logs              = false -> null
      - max_retry_count   = 0 -> null
      - memory            = 0 -> null
      - memory_swap       = 0 -> null
      - must_run          = true -> null
      - name              = "training" -> null
      - network_data      = [
          - {
              - gateway          = "172.17.0.1"
              - ip_address       = "172.17.0.2"
              - ip_prefix_length = 16
              - network_name     = "bridge"
            },
        ] -> null
      - network_mode      = "default" -> null
      - privileged        = false -> null
      - publish_all_ports = false -> null
      - read_only         = false -> null
      - restart           = "no" -> null
      - rm                = false -> null
      - shm_size          = 64 -> null
      - start             = true -> null
      - sysctls           = {} -> null
      - tmpfs             = {} -> null

      - labels {
          - label = "maintainer" -> null
          - value = "NGINX Docker Maintainers <docker-maint@nginx.com>" -> null
        }

      - ports {
          - external = 80 -> null
          - internal = 80 -> null
          - ip       = "0.0.0.0" -> null
          - protocol = "tcp" -> null
        }
    }

  # docker_image.nginx will be destroyed
  - resource "docker_image" "nginx" {
      - id     = "sha256:ed21b7a8aee9cc677df6d7f38a641fa0e3c05f65592c592c9f28c42b3dd89291nginx:latest" -> null
      - latest = "sha256:ed21b7a8aee9cc677df6d7f38a641fa0e3c05f65592c592c9f28c42b3dd89291" -> null
      - name   = "nginx:latest" -> null
    }

Plan: 0 to add, 0 to change, 2 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: 
```

If all the resources that will be destroyed (as indicated with the `-` sign) look correct, type "yes" to confirm. You should see something like:

```
docker_container.nginx: Destroying... [id=c218448af8ae779372bd3c02d87d711965188c14054a87a15af0a8b28e419abc]
docker_container.nginx: Destruction complete after 1s
docker_image.nginx: Destroying... [id=sha256:ed21b7a8aee9cc677df6d7f38a641fa0e3c05f65592c592c9f28c42b3dd89291nginx:latest]
docker_image.nginx: Destruction complete after 0s

Destroy complete! Resources: 2 destroyed.
```

Terraform will destroy the resources it had created earlier. Typing `terraform show` should now show no more information. 

## Next steps

In this guide, you have learned about the benefits of Infrastructure as Code and its importance in achieving consistent environments and productive workflows. You created a Terraform configuration file with the HashiCorp Configuration Language and learned about the key components. These files serve as sources of truth when it comes to replicating, automating, and scaling your infrastructure and can act as architecture documentation. You then successfully leveraged Terraform to initialize the Docker provider and provision Docker resources with just a few commands.  

Docker is a popular container tool and was used in this exercise since it can be run locally, quickly, and without any costs. As a next step, try to add different [providers](https://www.terraform.io/docs/providers/index.html) or try to provision some live cloud resources (but be mindful of costs).

Terraform abstracts away the differences between different providers and allows you to go to different environments and recreate everything declaratively. Unlike a deployment script, you don't need to know how to do each step. You can make changes to your configuration files to reflect what you want, apply it as many times as you want, and see the changes to your infrastructure. 

There is a lot more that the Terraform toolset can do when it comes to automating your infrastructure. You can learn more by reading through the [documentation](https://www.terraform.io/docs/index.html) or engaging with the [community](https://www.terraform.io/community.html).  
