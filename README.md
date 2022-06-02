# Packer
All things Packer

Note: to follow along with this code, you must first configure your connection to AWS using `aws configure` providing your AWS CLI credentials, the default region, and a default output format (i.e us-east-1 and json).

Here is a link to the official Hashicorp Packer tutorials used in this repo: https://learn.hashicorp.com/collections/packer/aws-get-started

## Install Packer on Windows

Open PowerShell as an adminsitrator. Please make sure chocolatey is also installed - https://chocolatey.org/install.  

```
choco install packer
```

Mine is already installed and on the latest version.

![choco install packer](./images/choco-install-packer.png)

YOu can also check if your package needs to be updated using `choco upgrade packer`. I usually verify it as well because sometimes Chocolately doesn't have the most recent packages, but it usually does. Still, it doesn't hurt to check (https://packer.io/downloads).

Refresh your environment variables to include any installed by Packer that may not be loaded.

```
refreshenv
```

You should get the following output:

![refreshenv](./images/refreshenv.png)

Verify the installation was successful with the following command.

```
packer
```

You should get output like this (I switched from PowerShell to VSCode from this point on):

![packer](./images/packer.png)

You have successfully installed Packer on your PC.

## Build an Image

The first step is creating a directory that will contain your Packer template, a configuration file that defines the image.

```
mkdir packer_tutorial
```

Navigate to the directory.

```
cd packer_tutorial
```

Create a file `aws-ubuntu.pkr.hcl` and add the following HCL block to it, then save.

```
packer {
  required_plugins {
    amazon = {
      version = ">= 0.0.2"
      source  = "github.com/hashicorp/amazon"
    }
  }
}

source "amazon-ebs" "ubuntu" {
  ami_name      = "learn-packer-linux-aws"
  instance_type = "t2.micro"
  region        = "us-west-2"
  source_ami_filter {
    filters = {
      name                = "ubuntu/images/*ubuntu-xenial-16.04-amd64-server-*"
      root-device-type    = "ebs"
      virtualization-type = "hvm"
    }
    most_recent = true
    owners      = ["099720109477"]
  }
  ssh_username = "ubuntu"
}

build {
  name    = "learn-packer"
  sources = [
    "source.amazon-ebs.ubuntu"
  ]
}
```

This template will create an AWS AMI from an EC2 Instance with the properties shown in the template. It uses the amazon plug-in with an EC2 Instance as the source to build an AMI named "learn-packer."

Next, initialize Packer within the packer_tutorial directory.

```
packer init .
```

If you haven't used this plugin before, you should get output after the command stated that the plugin has been installed.

Next, format and validate your Packer template.

```
packer fmt .
packer validate .
```

![packer format and validate](./images/packer-format-validate.png)

This will fix any formatting inconsistencies and validate the configuration of your template.

Now you can build your Packer image!

```
packer build aws-ubuntu-pkr.hcl
```

You should see it start to build your AMI and get confirmation like this:

![build](./images/build.png)

You can now check your AMI using the AWS CLI or AWS Management Console. 

IMPORTANT! Packer does not manage your AMIs. SO if you need to delete them or any snapshots, make sure you do so on your own!!!
