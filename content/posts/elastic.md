---
title: "Elastic Cloud on Kubernetes"
date: 2022-07-14T15:47:55+02:00
draft: false
toc: false
images:
tags:
  - kubernetes
  - elastic
  - terraform
---

### What is Elastic Cloud?
*Elastic Cloud is the best way to consume all of Elastic's products across any cloud. Easily deploy in your favorite public cloud, or in multiple clouds, and extend the value of Elastic with cloud-native features.*

### What is Kubernetes?
*Kubernetes, often abbreviated as “K8s”, orchestrates containerized applications to run on a cluster of hosts. The K8s system automates the deployment and management of cloud native applications using on-premises infrastructure or public cloud platforms.*

### Elastic Cloud on Kubernetes
We will use our [Kind](https://magsther.github.io/posts/kind/) that we deployed earlier.

Elastic has created a good [quickstart](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-quickstart.html) on how to **manually** deploy **Elastic Cloud on Kubernetes** (short ECK).

We are all for **automation** ;) so we will do it with [Terraform](https://www.terraform.io/)

In a previous [post](https://magsther.github.io/posts/kubeprometheus/), we deployed [Kube-Prometheus](https://github.com/prometheus-operator/kube-prometheus) by creating a Terraform module.

Let's create another module for **Elastic Cloud**.

### Elastic Cloud - Provider

As always, when working with **Terraform providers**, we will first take a look at the [Terraform Registry](https://registry.terraform.io/providers/elastic/ec/latest).

### Authentication
We need to tell Terraform how to authenticate with Elastic.

The [documentation](https://registry.terraform.io/providers/elastic/ec/latest/docs) mentions that the provider *offers two methods of authentication against the remote API: apikey or a combination of username and password.*

Using an **API key** is recommended, so that's what we will use here. 

*After you've generated your API Key, you can make it available to the Terraform provider by exporting it as the environment variable EC_API_KEY (recommended)*

`export TF_VAR_ec_api_key="<apikey value>"`

### Elastic Module

Create a new folder in the **modules** folder. 
`mkdir modules/elastic-cloud`

First we will add the Elastic Cloud provider to our `providers.tf` file

```
terraform {
  required_providers {
    ec = {
      source = "elastic/ec"
      version = "0.4.1"
    }
  }
}
```

### Adding Elastic resources

Create a `main.tf` file.
In this file, we will create the **Elastic resources** that we want to use. 

You can find some examples [here](https://registry.terraform.io/providers/elastic/ec/latest/docs/resources/ec_deployment)

In our case, we will use **elasticsearch** and **kibana**.
This deployment will be created in **Azure**.

The `ec_deployment` creates  **elasticsearch** and **kibana**.

```
resource "ec_deployment" "elastic_deployment" {
  name                   = var.ec_instance_name
  region                 = var.region
  deployment_template_id = var.deployment_template_id
  version                = var.es_version

  elasticsearch {
    autoscale = var.autoscale
  }

  kibana {}
}
```

As you can see from the snippet above, we make use of **input variables**, so that we can customize behavior.

The `deployment_template_id` and the `region` can be found [here](https://www.elastic.co/guide/en/cloud/current/ec-regions-templates-instances.html)

`Autocaling` is set to false by default, but can be enabled by setting the value in `variables.tf` to `true`.


### Adding variables

We will define the values in the `variables.tf` file. 
Using variables allows us to write configuration that is flexible and easier to re-use.

```
variable "ec_instance_name" {
  type        = string
  default     = "my-test-instance"
  description = "Name of Elastic deployment"
}

variable "region" {
  type        = string
  default     = "azure-westeurope"
  description = "Region of Elastic deployment"
}

variable "es_version" {
  type        = string
  default     = "8.2.3"
  description = "Version of Elastic deployment"
}

variable "deployment_template_id" {
  type        = string
  default     = "azure-memory-optimized"
  description = "Azure SKU of Elastic deployment"
}

variable "autoscale" {
  default     = false
  type        = bool
  description = "Autocale enabled of Elastic deployment"
}
```

We also add an `output.tf` file. This will make it easier for us to connect to the cluster once it is deployed.

```
output "elasticsearch_endpoint" {
  value = ec_deployment.elastic_deployment.elasticsearch[0].https_endpoint
}

output "elasticsearch_username" {
  value = ec_deployment.elastic_deployment.elasticsearch_username
}

output "elasticsearch_password" {
  value     = ec_deployment.elastic_deployment.elasticsearch_password
  sensitive = true
}
```

In the **root module** update the `main.tf` file with our new module

```
module "elastic" {
  source     = "./modules/elastic-cloud"
}
```

### Deploy time 
Let's see if this works.

Run `terraform init` in your terminal.
This will download our new provider (elastic cloud)

Run `terraform plan` to see what will be deployed.

Run `terraform apply` to apply the plan. Respond to the confirmation prompt with a yes.

After a little while, the Elastic cluster is deployed and can be found in the **Elastic Cloud portal**.


### Additional resources
https://registry.terraform.io/providers/elastic/ec/latest/docs