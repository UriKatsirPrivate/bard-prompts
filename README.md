Welcome to the "Bard Prompts" repository! This is a collection of prompt examples to be used with the Bard model.

The [Bard](https://bard.google.com/) model is a large language model trained by Google that is capable of generating human-like text. By providing it with a prompt, it can generate responses that continue the conversation or expand on the given prompt.

In this repository, you will find a variety of prompts that can be used with Bard.

### To get started, simply clone this repository and use the prompts in the README.md file as input for Bard. You can also use the prompts in this file as inspiration for creating your own.
---
# Prompts

## Explain Commands
<!-- Contributed by: [@f](https://github.com/f) -->
<!-- Reference: https://www.engraved.blog/building-a-virtual-machine-inside/ -->

> gcloud compute networks create demo-llm --project=landing-zone-demo-341118 --subnet-mode=custom </br>
Convert this command to terraform, use parameters when possible.
Convert this command to pulumi, use parameters when possible
What will this command do?

> gcloud compute networks create demo-llm --project=landing-zone-demo-341118 --subnet-mode=custom --mtu=1460 --bgp-routing-mode=regional </br>
What will this command do?
What would be the command to reverse it?

## Create/Modify IaC Modules
### Create Services with Terraform
> Create a terraform module with the following properties: </br>
Services will be created on GCP </br>
create a VPC network with 3 subnets (us-central1, me-west1 and me-central1) </br>
create a firewall rule that allows all traffic on port 22 from 77.125.87.220/32 </br>
create a GCE instance in one of the subnets and assign the firewall rule to the instance </br>
Create a GCS bucket in us-east1 region

>> Break the file into modules

### Modify .tf Files
>Given the terraform below, perform the following:
1.   replace the values with parameters
2. Add the code to create a gcs bucket to the current file. use parameters when possible.
```yaml
provider "google" {
  project = "my-project-id"
  region  = "us-central1"
}

resource "google_compute_instance" "my-instance" {
  name         = "my-instance"
  machine_type = "n1-standard-1"
  zone         = "us-central1-a"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
    network = "default"
    access_config {
    }
  }
}
```
3. split the .tf file into modules
4. what will be created when I run the code?

## Reverse Engineering
### TF State → TF Module
1. Write a simple terraform state file
2. Create a .tf file based on the state file
3. Split the .tf file into a more flexible structure. For example, move the variables into a separate file.
4. Rewrite the .tf files to use GCP equivalent

### TF Module → TF State
>given the .tf file below, create the state file that will be created.
###
```json 
  provider "google" {
  project = demo-project
  region = us-east1
}

resource "google_storage_bucket" "example" {
  name = demo-bucket-name
  location = us-east1
  storage_class = STANDARD
}
###
```
### TF State + TF Module → Plan
> I am going to provide two files. a terraform state file and a .tf file. Tell me what would be the result of applying the .tf file.
The state file:
```json
terraform {
  backend "gcs" {
    bucket = "my-bucket"
    path = "terraform.tfstate"
  }
}

resource "google_compute_instance" "example" {
  name = "my-instance"
  machine_type = "n1-standard-1"
  zone = "us-central1-a"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-9"
    }
  }
}

resource "google_compute_firewall" "example" {
  name = "my-firewall"
  network = "default"

  allow {
    protocol = "tcp"
    ports = ["80", "443"]
  }
}
```

The .tf file:
```json
provider "google" {
  project = var.project_id
  region = var.region
}

resource "google_storage_bucket" "example" {
  name = var.bucket_name
  location = var.region
  storage_class = var.storage_class
}
```
## Switch Between IaC Providers
### Pulumi to Terraform
> convert the Pulumi below to terraform:
``` python
import * as pulumi from "@pulumi/pulumi";
import * as gcp from "@pulumi/gcp";

const config = new pulumi.Config();
const project_id = config.require("project_id");
const region = config.require("region");
const zone = config.require("zone");
const instance_name = config.require("instance_name");
const machine_type = config.require("machine_type");
const image = config.require("image");
const network = config.require("network");
const bucket_name = config.require("bucket_name");
const bucket_location = config.require("bucket_location");

const provider = new gcp.Provider("provider", {
  project: project_id,
  region: region,
});

const myInstance = new gcp.compute.Instance("my-instance", {
  name: instance_name,
  machineType: machine_type,
  zone: zone,

  bootDisk: {
    initializeParams: {
      image: image,
    },
  },

  networkInterfaces: [{
    network: network,
    accessConfigs: [{}],
  }],
}, {
  provider: provider,
});
```
###  Terraform to Pulumi
>convert the terraform below to pulumi
```json
provider "google" {
  project = var.project_id
  region  = var.region
}

resource "google_compute_instance" "my-instance" {
  name         = var.instance_name
  machine_type = var.machine_type
  zone         = var.zone

  boot_disk {
    initialize_params {
      image = var.image
    }
  }

  network_interface {
    network = var.network
    access_config {
    }
  }
}
```
###  Terraform to gCloud
> Write a .tf file to create a gce instance. Use demo values as needed. </br>
Replace the .tf with gCloud commands

## Switch Between Cloud Providers
### AWS SDK to GCP SDK
>Given the aws SDK code below, convert it into a GCP equivalent:
```python
import sys
import boto3
from botocore.exceptions import ClientError

instance_id = sys.argv[2]
action = sys.argv[1].upper()

ec2 = boto3.client('ec2')

if action == 'ON':
    # Do a dryrun first to verify permissions
    try:
        ec2.start_instances(InstanceIds=[instance_id], DryRun=True)
    except ClientError as e:
        if 'DryRunOperation' not in str(e):
            raise

    # Dry run succeeded, run start_instances without dryrun
    try:
        response = ec2.start_instances(InstanceIds=[instance_id], DryRun=False)
        print(response)
    except ClientError as e:
        print(e)
else:
    # Do a dryrun first to verify permissions
    try:
        ec2.stop_instances(InstanceIds=[instance_id], DryRun=True)
    except ClientError as e:
        if 'DryRunOperation' not in str(e):
            raise

    # Dry run succeeded, call stop_instances without dryrun
    try:
        response = ec2.stop_instances(InstanceIds=[instance_id], DryRun=False)
        print(response)
    except ClientError as e:
        print(e)
```
### AWS CDK to Terraform
>Convert the CDK python code below to terraform:
```python
import os.path

from aws_cdk.aws_s3_assets import Asset

from aws_cdk import (
    aws_ec2 as ec2,
    aws_iam as iam,
    App, Stack
)

from constructs import Construct

dirname = os.path.dirname(__file__)


class EC2InstanceStack(Stack):

    def __init__(self, scope: Construct, id: str, **kwargs) -> None:
        super().__init__(scope, id, **kwargs)

        # VPC
        vpc = ec2.Vpc(self, "VPC",
            nat_gateways=0,
            subnet_configuration=[ec2.SubnetConfiguration(name="public",subnet_type=ec2.SubnetType.PUBLIC)]
            )

        # AMI
        amzn_linux = ec2.MachineImage.latest_amazon_linux(
            generation=ec2.AmazonLinuxGeneration.AMAZON_LINUX_2,
            edition=ec2.AmazonLinuxEdition.STANDARD,
            virtualization=ec2.AmazonLinuxVirt.HVM,
            storage=ec2.AmazonLinuxStorage.GENERAL_PURPOSE
            )

        # Instance Role and SSM Managed Policy
        role = iam.Role(self, "InstanceSSM", assumed_by=iam.ServicePrincipal("ec2.amazonaws.com"))

        role.add_managed_policy(iam.ManagedPolicy.from_aws_managed_policy_name("AmazonSSMManagedInstanceCore"))

        # Instance
        instance = ec2.Instance(self, "Instance",
            instance_type=ec2.InstanceType("t3.nano"),
            machine_image=amzn_linux,
            vpc = vpc,
            role = role
            )

        # Script in S3 as Asset
        asset = Asset(self, "Asset", path=os.path.join(dirname, "configure.sh"))
        local_path = instance.user_data.add_s3_download_command(
            bucket=asset.bucket,
            bucket_key=asset.s3_object_key
        )

        # Userdata executes script from S3
        instance.user_data.add_execute_file_command(
            file_path=local_path
            )
        asset.grant_read(instance.role)

app = App()
EC2InstanceStack(app, "ec2-instance")

app.synth()
```
### CloudFormation to Terraform
>translate the cloudformation template below to terraform:
```json
{
  "AWSTemplateFormatVersion" : "2010-09-09",
  "Description" : "AWS CloudFormation Sample Template EC2Instance.",
  "Parameters" : {
    "KeyName": {
      "Description" : "Name of an existing EC2 KeyPair to enable SSH access to the instance",
      "Type": "AWS::EC2::KeyPair::KeyName",
      "ConstraintDescription" : "must be the name of an existing EC2 KeyPair."
    },
    "InstanceType" : {
      "Description" : "WebServer EC2 instance type",
      "Type" : "String",
      "Default" : "t2.small",
      "AllowedValues" : [ "t1.micro", "t2.nano", "t2.micro", "t2.small"]
,
      "ConstraintDescription" : "must be a valid EC2 instance type."
    },
    "SSHLocation" : {
      "Description" : "The IP address range that can be used to SSH to the EC2 instances",
      "Type": "String",
      "MinLength": "9",
      "MaxLength": "18",
      "Default": "0.0.0.0/0",
      "AllowedPattern": "(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})/(\\d{1,2})",
      "ConstraintDescription": "must be a valid IP CIDR range of the form x.x.x.x/x."
   }
  },
  "Resources" : {
    "EC2Instance" : {
      "Type" : "AWS::EC2::Instance",
      "Properties" : {
        "InstanceType" : { "Ref" : "InstanceType" },
        "SecurityGroups" : [ { "Ref" : "InstanceSecurityGroup" } ],
        "KeyName" : { "Ref" : "KeyName" },
        "ImageId" : { "Fn::FindInMap" : [ "AWSRegionArch2AMI", { "Ref" : "AWS::Region" },
                          { "Fn::FindInMap" : [ "AWSInstanceType2Arch", { "Ref" : "InstanceType" }, "Arch" ] } ] }
      }
    },
    "InstanceSecurityGroup" : {
      "Type" : "AWS::EC2::SecurityGroup",
      "Properties" : {
        "GroupDescription" : "Enable SSH access via port 22",
        "SecurityGroupIngress" : [ {
          "IpProtocol" : "tcp",
          "FromPort" : "22",
          "ToPort" : "22",
          "CidrIp" : { "Ref" : "SSHLocation"}
        } ]
      }
    }
  },
  "Outputs" : {
    "InstanceId" : {
      "Description" : "InstanceId of the newly created EC2 instance",
      "Value" : { "Ref" : "EC2Instance" }
    }}}
```
>create a gcp equivalent from the terraform

### AWS Terraform to GCP Terraform
>Given the aws terraform below, create the gcp equivalent:
```json
# Importing required AWS provider
provider "aws" {
  region = "us-east-1"
}

# Creating a VPC
resource "aws_vpc" "my_vpc" {
  cidr_block = "10.0.0.0/16"

  tags = {
    Name = "my_vpc"
  }
}

# Creating a public subnet
resource "aws_subnet" "public" {
  vpc_id = aws_vpc.my_vpc.id
  cidr_block = "10.0.1.0/24"
  availability_zone = "us-east-1a"
}

# Creating an IAM role for EC2 instance
resource "aws_iam_role" "instance_role" {
  name = "InstanceSSM"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Principal = {
          Service = "ec2.amazonaws.com"
        }
      }
    ]
  })
}

# Adding the AmazonSSMManagedInstanceCore managed policy to the IAM role
resource "aws_iam_role_policy_attachment" "instance_role_policy" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore"
  role = aws_iam_role.instance_role.name
}

# Creating a security group for the EC2 instance
resource "aws_security_group" "instance_sg" {
  name_prefix = "instance_sg_"
  vpc_id = aws_vpc.my_vpc.id

  ingress {
    from_port = 0
    to_port = 65535
    protocol = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port = 0
    to_port = 65535
    protocol = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# Creating an EC2 instance
resource "aws_instance" "my_instance" {
  ami = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.nano"
  key_name = "my_key_pair"
  subnet_id = aws_subnet.public.id
  vpc_security_group_ids = [aws_security_group.instance_sg.id]

  user_data = <<-EOF
              #!/bin/bash
              sudo yum update -y
              sudo yum install -y httpd
              sudo service httpd start
              sudo chkconfig httpd on
              EOF

  iam_instance_profile = aws_iam_instance_profile.instance_profile.name
}

# Creating an instance profile for the EC2 instance
resource "aws_iam_instance_profile" "instance_profile" {
  name = "my_instance_profile"
  role = aws_iam_role.instance_role.name
}

# Creating an S3 bucket to store the script
resource "aws_s3_bucket" "script_bucket" {
  bucket = "my-bucket"
  acl = "private"
}

# Adding the script as an object to the S3 bucket
resource "aws_s3_bucket_object" "script_object" {
  key = "configure.sh"
  source = "/path/to/configure.sh"
  bucket = aws_s3_bucket.script_bucket.id
}
```