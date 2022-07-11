---
title: "Kind & Terraform"
date: 2022-07-09T19:50:17+02:00
draft: false
toc: false
images:
tags:
  - kubernetes
  - kind
  - terraform
---

## What is Kind?
*[kind](https://kind.sigs.k8s.io/) is a tool for running local Kubernetes clusters using Docker container “nodes”.
kind was primarily designed for testing Kubernetes itself, but may be used for local development or CI.*

This is the beginning of a series of articles on how to manage **Kubernetes** resources via **Terraform**.

## Deploying Kind using Terraform
First we need a **Kubernetes** cluster running locally. 
There are many tools to pick from (**minikube**, **K3s**, **MicroK8s** etc..) , but the tool we will use is **Kind**. 
One of the things that I really like with Kind, is the support of multi-node (including HA) clusters.

Note: To use kind, you will need to have [docker](https://docs.docker.com/get-docker/) installed.

Even though we can easily create a new cluster by running `kind create cluster` we will do this by using **Terraform**

Using [Terraform](https://www.terraform.io/) to manage your Kubernetes resources have benefits like:
- Unified Workflow
- Full Lifecycle Management
- Graph of Relationships

## Terraform Registry
The best place to start when using Terraform **providers** is the **Terraform Registry**.
Since we want to use **Kind**, we search for [Kind](https://registry.terraform.io/providers/tehcyx/kind/0.0.13)

The **Use Provider** button (at the top right corner), shows an example of how to use the provider.

We can use this as a template.

## Setting up the structure
Create a new folder on your workstation:
`mkdir kind-demo`

Create a `main.tf`. This file will hold the configuraton of the **Kind** cluster.

As you can see from the code below, our cluster will contain one node with the role **control-plane** and one node simply called **worker**.

Paste in the following code:

```
terraform {
  required_providers {
    kind = {
      source = "tehcyx/kind"
      version = "0.0.13"
    }
  }
}

provider "kind" {}

  resource "kind_cluster" "default" {
    name = "kindcluster"
    node_image = "kindest/node:v1.24.0"
    wait_for_ready = true
    
    kind_config {
      kind = "Cluster"
      api_version = "kind.x-k8s.io/v1alpha4"

      node {
        role = "control-plane"
      }

      node {
        role = "worker"
      }
    }
  }
  ```

  If you want to add additional nodes, you can just add them to the code like this:
  
  ```
      node {
        role = "worker"
      }
```

Save the `main.tf` file and in the terminal, type `terraform init`

You should see an output similar to this:

```
Initializing the backend...

Initializing provider plugins...
- Finding tehcyx/kind versions matching "0.0.13"...
- Installing tehcyx/kind v0.0.13...
- Installed tehcyx/kind v0.0.13
...
Terraform has been successfully initialized!
```

Now run `terraform plan`. 

The **plan** creates an execution plan, which lets you preview the changes that Terraform plans to make to your infrastructure.

```
Terraform will perform the following actions:

  # kind_cluster.default will be created
  + resource "kind_cluster" "default" {
      + client_certificate     = (known after apply)
      + client_key             = (known after apply)
      + cluster_ca_certificate = (known after apply)
      + completed              = (known after apply)
      + endpoint               = (known after apply)
      + id                     = (known after apply)
      + kubeconfig             = (known after apply)
      + kubeconfig_path        = (known after apply)
      + name                   = "kindcluster"
      + node_image             = "kindest/node:v1.24.0"
      + wait_for_ready         = true

      + kind_config {
          + api_version = "kind.x-k8s.io/v1alpha4"
          + kind        = "Cluster"

          + node {
              + role = "control-plane"
            }
          + node {
              + role = "worker"
            }
        }
    }

Plan: 1 to add, 0 to change, 0 to destroy.
```

Apply the changes by running `terraform apply`

This command executes the actions proposed in a Terraform **plan**.

Once completed, your new **Kubernetes** cluster is created. 

Let's do some interactions with our cluster.

`kubectl cluster-info `

```
Kubernetes control plane is running at https://127.0.0.1:53839
CoreDNS is running at https://127.0.0.1:53839/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

`kubectl get nodes`

```
NAME                        STATUS   ROLES           AGE     VERSION
kindcluster-control-plane   Ready    control-plane   3m48s   v1.24.0
kindcluster-worker          Ready    <none>          3m20s   v1.24.0
```

## Conclusion
In this blog post, we installed Kubernetes (kind) using Terraform.

In the next post, we will create a **Kube-Prometheus stack** using a **Terraform** module.