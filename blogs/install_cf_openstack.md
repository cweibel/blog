# Deploying Cloud Foundry on Terraform using Terraform

# Training Manual

Welcome!

This manual will guide you through the steps necessary to deploy Cloud Foundry using Terraform on OpenStack. A tremendous amount of automation has been put in place to allow you to quickly deploy Cloud Foundry in an easy and repeatable way.

# Overview

There are four primary steps to deploying Cloud Foundry (and one to destroy), we will explore each step in the subsequent sections
 - Prerequisites
 - Configure Install
 - Connect to Cloud Foundry
 - Deploy Services
 - Destroy the Deployment

# Exercise 1 - Prerequisites

## Local Configuration

We’ll assume that you have an OSX or Linux computer you are working from locally.  There are a couple pieces of software which you will need installed:
 - git
 - Terraform 0.4.0+

### Install git
You likely already have git installed if you do any development locally.  Follow the instructions here to install git: http://git-scm.com/book/en/v2/Getting-Started-Installing-Git a short summary is:
 - OSX - “brew install git”
 - RHEL/CentOS/Fedora - “sudo yum install -y git”
 - Ubuntu/Debian = “sudo apt-get install -y git”

### Install Terraform

Visit the following website to download a version of terraform for you local computer: https://www.terraform.io/downloads.html

After you download the appropriate zip, copy the files to a folder in your PATH, in the example below we use `~/bin`

```bash
mv ~/Downloads/terraform_0/* ~/bin
```

Open a new terminal window and run:
```
terraform -v
```

You should get the following output:
```
Terraform v0.4.2
```

# Exercise 2 - Configure Install

TODO: Now that we have our AWS keys we can supply these values to Terraform to deploy Cloud Foundry.

## Clone the Repo
On your local computer run the following:
```bash
git clone https://github.com/cloudfoundry-community/terraform-openstack-cf-install
cd terraform-openstack-cf-install
cp terraform.tfvars.example terraform.tfvars
```


## Edit Variable File
This will copy a terraform configuration to your local computer.  Using your favorite text editor (in the examples we will use vi) edit the terraform.tfvars file and supply the the terraform paramters:

```
vi terraform.tfvars
```
The values for each of the parameters are available [here](TODO)

After editing your file should look like:

```
network = "192.168"
auth_url="http://10.2.95.20:5000/v2.0"
tenant_name="chris_cf"
tenant_id="69f43a6776344a338b9cafdea088aca4"
username="chris"
password="chris"
public_key_path="/Users/chris/.ssh/id_rsa.pub"
key_path="/Users/chris/.ssh/id_rsa"
floating_ip_pool="net04_ext"
region="RegionOne"
network_external_id="bb89edee-e639-43b9-b27e-db2fcab833b2"
```

Now you are ready to deploy, run:
```
make plan
make apply
make provision
```

Go get something to drink. It will take about an hour to deploy everything to OpenStack. Don’t panic if an error occurs during `make apply` or `make provision`, run the command again as AWS resources aren’t always available when requested.

When the installation has completed, your screen should output a series of values you will need to connect to the Cloud Foundry deployment, in our example we see:

TODO: replace the screenshot below
```bash
aws_instance.bastion (remote-exec): Deployed 'cf-aws-tiny.yml' to 'bosh-vpc-885f0bed'
aws_instance.bastion: Creation complete

Apply complete! Resources: 9 added, 2 changed, 0 destroyed.

The state of your infrastructure has been saved to the path
below. This state is required to modify and destroy your
infrastructure, so keep it safe. To inspect the complete state
use the 'terraform show' command.

State path: terraform.tfstate

Outputs:

  aws_internet_gateway_id              = igw-23b12546
  aws_key_path                         = ~/.ssh/bosh.pem
  aws_route_table_private_id           = rtb-5d8cad38
  aws_route_table_public_id            = rtb-558cad30
  aws_subnet_bastion                   = subnet-17a6204e
  aws_subnet_bastion_availability_zone = us-west-1a
  aws_vpc_id                           = vpc-885f0bed
  bastion_ip                           = 54.175.138.250
  cf_admin_pass                        = c1oudc0wc1oudc0w
  cf_api                               = api.run.52.0.125.51.xip.io
  cf_domain                            = XIP

```
The three fields which we will need to connect to Cloud Foundry and the Bastion server have been noted above.


# Exercise 3 - Connect to Cloud Foundry

In order to connect to Cloud Foundry, you will need to do the following locally (note that you can just use the Bastion server which will have these tools already installed):
 - Install the CF CLI Tool
 - Use the CF CLI Tool to connect to Cloud Foundry

## Install CF CLI - Local

There are two ways to push CF apps: from your laptop or the bastion server.  If will only be using the bastion server you can skip this step as it is already installed on the bastion server.  If you would like to push CF apps from your laptop:

Open a terminal window on your laptop and run:
```bash
curl -s https://raw.githubusercontent.com/cloudfoundry-community/traveling-cf-admin/master/scripts/installer | bash
```
## Install CF CLI - Bastion Server
The CF CLI and other tools are already installed on the bastion server.

## Connect
Now that the CF CLI tool has been installed, connect to Cloud Foundry using the TODO: 2 noted values outputted from Step 2:
```bash
cf login --skip-ssl-validation -a api.run.52.0.125.51.xip.io -u admin -p c1oudc0wc1oudc0w
```

Thats it!  You can now write your application locally and then push it to Cloud Foundry.


# Exercise 4 - Deploy Services

Now that we have Cloud Foundry deployed to OpenStack let’s deploy some services which the Cloud Foundry applications can consume, such as PostgreSQL and Redis.

## Modify terraform.tfvars

Add the following line to the terraform.tfvars file:
```bash
install_docker_services = "true"
```

Now you are ready to deploy, run:
```bash
make plan
make apply
make provision
```

After 20 or so minutes the Docker Services VM will be deployed. Even after the VM and jobs are running it will take about 10 minutes for all of the docker images to be imported and available for the Service Broker.
Register Services with Service Broker

In Exercise 3 you installed the CF CLI tools which we will use now to create a service broker.  It may take up to 20 minutes for all of the unicorn web server to fetch all of the docker images. Execute the following on the bastion server:
```bash
cf create-service-broker docker containers containers http://cf-containers-broker.run.52.0.125.51.xip.io
```

###Note 1
If you get an error indicating “cf” is not installed, run the following:
```bash
curl -s https://raw.githubusercontent.com/cloudfoundry-community/traveling-cf-admin/master/scripts/installer | bash
```

### Note 2
If you forgot to login in Exercise 3 you will get the message ”No API endpoint set. Use 'cf login' or 'cf api' to target an endpoint.”  If so, execute the following:
```bash
cf login --skip-ssl-validation -a api.run.52.0.125.51.xip.io -u admin -p c1oudc0wc1oudc0w
```

Once the service broker is created you can list the services available:
```bash
cf service-access
cf enable-service-access mysql56  # Enables a mysql 5.6 instance
```

# Exercise 5 - Tear Down

Terraform does not yet quite cleanup after itself. You can run make destroy to get quite a few of the resources you created, but you will probably have to manually track down some of the bits and manually remove them. Once you have done so, run make clean to remove the local cache and status files, allowing you to run everything again without errors. Locally run:
```bash
make destroy
sleep 10
make destroy
make clean
```

# Resources

 - The primary repository is located [here](https://github.com/cloudfoundry-community/terraform-openstack-cf-install)
 - [Installing RVM](https://rvm.io/rvm/install)
 - [Installing Git](http://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
 - [CF CLI Releases](https://github.com/cloudfoundry/cli/releases)
 - [Traveling CF CLI Install](https://github.com/cloudfoundry-community/traveling-cf-admin)
