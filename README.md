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
