---
title: "How to monitor Elasticsearch with Kube-Prometheus"
date: 2022-07-22T20:05:23+02:00
draft: false
toc: false
images:
tags:
  - elastic
  - prometheus
  - kubernetes
  - helm
  - terraform
---

### How to monitor Elasticsearch with Kube-Prometheus
In the previous [post](https://magsther.github.io/posts/elastic/) we deployed elastic cloud into our Kubernetes cluster.

To monitor our Elastic deployment, we can use [Elasticsearch Exporter[(https://github.com/prometheus-community/elasticsearch_exporter)]

There are basically five steps that we need to take:
1. Deploy **Elasticsearch Exporter** to Kubernetes.
2. Verify that the exporter can **scrape metrics** from our Elastic cluster.
3. Make Kube-Prometheus to **scrape the exporter.** 
4. Add Prometheus as **data source** to Grafana
5. Add a **Grafana Dashboard** to visualise the data.

In this blog post we will begin with step 1 and 2, and in the next posts we will do rest of the steps.

### What is Elasticsearch Exporter?
Elasticsearch Exporter is a **Prometheus** exporter for various metrics about Elasticsearch. 
It fetches information from an Elasticsearch cluster on every scrape,

I recommend that you read through the [documentation](https://github.com/prometheus-community/elasticsearch_exporter) to find out how to configure the exporter to best fit with your use case.

Please make a note of this `es.uri` argument, as this is the address of the Elasticsearch node we should connect to.
It's also here we need to add our credentials if `basic auth` is needed.


### Elasticsearch Exporter - Helm
We will deploy our exporter using a helm chart that can be found in the [prometheus-community charts repository](https://github.com/prometheus-community/helm-charts/tree/main/charts/prometheus-elasticsearch-exporter)

Installing the `helm` chart manually in the `Kubernetes` cluster is as simple as running:
`helm repo add prometheus-community https://prometheus-community.github.io/helm-charts`

`helm repo update`

`helm install [RELEASE_NAME] prometheus-community/prometheus-elasticsearch-exporter`

### Elasticsearch Exporter and Terraform
To deploy the [exporter](https://github.com/prometheus-community/elasticsearch_exporter) using `Terraform`, we will create a new file in our `elastic-cloud` module. 

`vim es-exporter.tf`

First we add a new resource (`helm_release`) and name it `es_exporter`.

In our resource we add:
- repository
- name 
- chart
- version
- timeout

```
resource "helm_release" "elasticsearch_exporter" {
  repository = var.helm_chart_repository
  name       = "prometheus-elasticsearch-exporter"
  chart      = "prometheus-elasticsearch-exporter"
  version    = var.elasticsearch_exporter_chart_version
  timeout    = 3600
}
```

If you are running your Elastic cluster locally (and accessible on `localhost:9200`), then this is all you need to do.

If you have a remote Elastic cluster and that requires `basic auth`, then you will need to change the value of `es.uri` in your code. To set values in the `helm_relase` in **Terraform**, you use `set`.

```
  set {
    name  = "es.uri"
    value = replace(ec_deployment.elastic_deployment.elasticsearch.0.https_endpoint, "https://", "https://${ec_deployment.elastic_deployment.elasticsearch_username}:${ec_deployment.elastic_deployment.elasticsearch_password}@")
  }
```


Add your to the `variables.tf` file like this:

```
variable "helm_chart_repository" {
  type = string
  default = "https://prometheus-community.github.io/helm-charts"
  description = "Helm Chart repository"
}
```

```
variable "elasticsearch_exporter_chart_version" {
  type        = string
  description = "The version of the elasticsearch-exporter chart in the Helm repository."
  default     = "4.13.0"
}
```

Since we are using the [Helm](https://registry.terraform.io/providers/hashicorp/helm/latest) provider from the Terraform Registry, we will need to add this to our `providers.tf` file like this:

```
    helm = {
      source  = "hashicorp/helm"
      ersion  = "~> 2.6.0"
    }
```

### Deploying Elasticsearch Exporter using Terraform

Since we added a new provider, we need to run `terraform init` to install this since its required by the configuration.

If we now run `terraform plan`, it should detect our changes.

Apply the terraform plan with `terraform apply`

After a little while **elastic-exporter** is deployed to our **Kubernetes** cluster.

### Interacting with the Elasticsearch Exporter
Elasticsearch Exporter is now fetching information from our Elasticsearch cluster.

Let's have a look of what was deployed in the previous step.

Check that the **endpoint** exposes metrics: 
`k port-forward prometheus-elasticsearch-exporter-xxx 9108:9108`

### Conclusion
In this blog post, we deployed an `Elastic-exporter` (which scrapes metrics from our Elastic clsuter) using the `Helm` resource from `Terraform` to our `Kubernetes` cluster.

In the next post we will add a `servicemonitor` to our exporter to have so that our `Kube-Prometheus` can **scrape the metrics** and display it in a Grafana Dashboard.

> All code for this blog can be found [here](https://github.com/magsther/code)
