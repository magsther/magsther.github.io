---
title: "Kube-Prometheus (with Terraform and Helm)"
date: 2022-07-11T20:37:53+02:00
draft: false
toc: false
images:
tags:
  - kubernetes
  - terraform
  - kube-prometheus
  - helm
  - observability
---

### Kube-Prometheus using Terraform (and Helm)

In the previous post we deployed our **Kind** cluster using **Terraform**.

Let's continue and add the [kube-prometheus](https://github.com/prometheus-operator/kube-prometheus) stack to our Kubernetes cluster.

### What is Kube-Prometheus?
**Kube-Prometheus** is magic ;)

You deploy it to your Kubernetes cluster and it will automatically collect metrics from all Kubernetes components.
This means, **prometheus** , some default set of **rules** and pre-configured **Grafana** dashboards!!! 

The only prerequisites we need is a Kubernetes cluster, which we deployed in our previosu post.

### Terraform providers and authentication
While we could deploy **kube-prometheus** by running `helm install my-kube-prometheus-stack prometheus-community/kube-prometheus-stack --version 36.6.2`, we will create a **terraform** modules for this.

Before creating our new modules, we need to add the following providers to our root module. 
- [Helm](https://registry.terraform.io/providers/hashicorp/helm/latest/docs)
- [Kubernetes](https://registry.terraform.io/providers/hashicorp/kubernetes/latest/docs)

> Remember, if you are going to use **providers** (like **helm**, **kubernetes**), you will need to add them to your code.

while you could add this to the `main.tf` file, it's usually better to seperate it into it's own file.

Add this content to `provders.tf`

```
# For authentication, the Helm provider can get its configuration by supplying a path to your kubeconfig file.
provider "helm" {
  kubernetes {
    config_path = "~/.kube/config"
  }
}

# Used to interact with the resources supported by Kubernetes.
# The provider needs to be configured with the proper credentials before it can be used.
provider "kubernetes" {
  config_path = pathexpand(var.kube_config)
}
```

Create a new file called `variables.tf` with this content:

```
variable "kube_config" {
  type    = string
  default = "~/.kube/config"
}
```

### Create the Terraform module
Now, we can start with our module.

Create a new folder called `modules` and inside that a new folder with the name of your module. In our case, this will be `kube-prometheus`. 

Create a new file in that folder and name it `main.tf`

Add this content to that file:

```
resource "helm_release" "kube-prometheus" {
  name       = "kube-prometheus-stackr"
  namespace  = var.namespace
  version    = var.kube-version
  repository = "https://prometheus-community.github.io/helm-charts"
  chart      = "kube-prometheus-stack"
}
```

How did we know which **helm chart** to use? 
Well, we went to [artifacthub](https://artifacthub.io/) which is a web-based application that contains **Kubernetes** packages.

We searched for **kube-prometheus**, so we put that in the search bar, which gave us information how to deploy it using [Helm](https://artifacthub.io/packages/helm/prometheus-community/kube-prometheus-stack)

Next, we went to [Terraform Registry](https://registry.terraform.io/providers/hashicorp/helm/latest/docs) to look up the **Helm** provider.

### Namespaces
Creating namespace with the Kubernetes provider is better than auto-creation in the helm_release.

Create a new file  called `namespaces.tf` with the following content:

```
resource "kubernetes_namespace" "monitoring" {
  metadata {
    name = var.namespace
  }
}
```

### Variables
In the `variables.tf` file we will add our input variables. 

Note, here we set the default value of our namespace to **monitoring**

```
variable "kube_config" {
  type    = string
  default = "~/.kube/config"
}

variable "namespace" {
  type    = string
  default = "monitoring"
}

variable "kube-version" {
}
```

The last step we need to do is to "call" our module from the `main.tf` file in the root module.

```
module "kube" {
  source = "./modules/kube-prometheus"
  kube-version = "36.2.0"
}
```

### Deploying using Terraform

Since we added a new module, we need to run `terraform init` to install this since its required by the configuration.

If we now run `terraform plan`, it should detect our changes.

```
...
....
# module.kube.kubernetes_namespace.monitoring will be created
  + resource "kubernetes_namespace" "monitoring" {
      + id = (known after apply)

      + metadata {
          + generation       = (known after apply)
          + name             = "monitoring"
          + resource_version = (known after apply)
          + uid              = (known after apply)
        }
    }
```

Apply the terraform plan with `terraform apply`

After a little while **kube-prometheus** is installed to our **Kubernetes** cluster.

### Interacting with Kube-Prometheus

Let's have a look of what was deployed in the previous step.

First, check what **pods** are running:

`kubectl get pods -n monitoring`

```
NAME                                                        READY   STATUS    RESTARTS   AGE
alertmanager-kube-prometheus-stackr-alertmanager-0          2/2     Running   0          66m
kube-prometheus-stackr-grafana-7cd9d66c5c-rhxsm             3/3     Running   0          66m
kube-prometheus-stackr-kube-state-metrics-98cbcdf4b-2x8qp   1/1     Running   0          66m
kube-prometheus-stackr-operator-7bbddd8bb4-b9pcv            1/1     Running   0          66m
kube-prometheus-stackr-prometheus-node-exporter-njjx6       1/1     Running   0          66m
kube-prometheus-stackr-prometheus-node-exporter-p8nnq       1/1     Running   0          66m
prometheus-kube-prometheus-stackr-prometheus-0              2/2     Running   0          66m
```

Let's check what **services** are deployed:

`kubectl get svc --namespace monitoring`

```
NAME                                              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
alertmanager-operated                             ClusterIP   None            <none>        9093/TCP,9094/TCP,9094/UDP   69m
kube-prometheus-stackr-alertmanager               ClusterIP   10.96.49.165    <none>        9093/TCP                     70m
kube-prometheus-stackr-grafana                    ClusterIP   10.96.91.188    <none>        80/TCP                       70m
kube-prometheus-stackr-kube-state-metrics         ClusterIP   10.96.38.109    <none>        8080/TCP                     70m
kube-prometheus-stackr-operator                   ClusterIP   10.96.44.188    <none>        443/TCP                      70m
kube-prometheus-stackr-prometheus                 ClusterIP   10.96.168.194   <none>        9090/TCP                     70m
kube-prometheus-stackr-prometheus-node-exporter   ClusterIP   10.96.131.191   <none>        9100/TCP                     70m
prometheus-operated                               ClusterIP   None            <none>        9090/TCP                     69m
```


### Accessing Prometheus
To access **Prometheus**, we can use **port forwarding**.

`kubectl port-forward` allows using resource name, such as a pod name, to select a matching pod to port forward to.

`kubectl port-forward svc/kube-prometheus-stackr-prometheus 9090:9090 --namespace monitoring`

`Forwarding from 127.0.0.1:9090 -> 9090`

Open a browser and go to: `http://localhost:9090/` to see the Prometehus UI.

### Accessing Grafana
Similar, to access **Grafana**, use this command:

`kubectl port-forward svc/kube-prometheus-stackr-grafana 3000:80 --namespace monitoring`

### Accessing Alertmanager
`kubectl port-forward svc/kube-prometheus-stackr-alertmanager 9093:9093 --namespace monitoring`
