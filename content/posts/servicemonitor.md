---
title: "Kubernetes Servicemonitor"
date: 2022-07-22T20:20:24+02:00
draft: false
toc: false
images:
tags:
  - kubernetes
  - terraform
  - prometheus
  - kube-prometheus
  - helm
  - grafana
---

### Kubernetes ServiceMonitor
`ServiceMonitor` describes the set of targets to be monitored by Prometheus.  

In the previous post we deployed [Elasticsearch Exporter](https://github.com/prometheus-community/elasticsearch_exporter) to scrape metrics from our Elastic cluster.

We defined five steps to take to monitor our Elastic deployment.

There are basically five steps that we need to take:
1. Deploy **Elasticsearch Exporter** to Kubernetes.
2. Verify that the exporter can **scrape metrics** from our Elastic cluster.
3. Make Kube-Prometheus to **scrape the exporter.** 
4. Add Prometheus as **data source** to Grafana
5. Add a **Grafana Dashboard** to visualise the data.

In this blog post we will continue with steps 3 and 4. 

### Prerequisite
Have an existing Kubernetes cluster with Kube-Prometheus deployed.

We did all that (using Terraform). 
You can find the blog post [here](https://magsther.github.io/posts/kubeprometheus/)

### Create the servicemonitor
We will create a serviceresource in Kubernetes to allow our existing `Kube-Prometheus` to scrape the metrics from the elastic-exporter.

Create a new file called `servicemonitor.yaml` with the following content.

```
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: elastic-exporter
  namespace: monitoring
  labels:
    release: kube-prometheus-stack
spec:
  endpoints:
  - interval: 5s
    port: http
  namespaceSelector:
    matchNames:
      - monitoring
  selector:
    matchLabels:
      app: prometheus-elasticsearch-exporter
```

Add the following to the `main.tf` file in the `elastic-cloud` module

```
resource "kubernetes_manifest" "elastic_servicemonitor" {
  manifest = yamldecode(templatefile("${path.module}/templates/servicemonitor.yaml", {
    name      = var.ec_instance_name
    appName   = local.elastic_cloud.app
  }))
}
```

Here we add a `kubernetes_manifest` resource and name it `elastic_servicemonitor`.

We specify where `servicemonitor.yaml` can be found.

The `appName` is important and needs to map the `matchLabels` in our yaml file.


### Deploying Elasticsearch Exporter using Terraform
Run `terraform plan`, to verify the changes that Terraform detected.

Apply the terraform plan with `terraform apply`

After a little while **servicemonitor** is deployed to our **Kubernetes** cluster.

Open the `prometheus UI` with: 

`kubectl port-forward svc/kube-prometheus-stackr-prometheus 9090:9090 --namespace monitoring`

and check the **target** and verify that its there.

Prometheus is now scraping metrics from the elastic-exporter.

### Conclusion
In this blog post, we deployed a `servicemonitor` so that our Prometheus instance can scrape metrics from the elastic-exporter.

In the next post we will create a Grafana Dashboard to display the data.

> All code for this blog can be found [here](https://github.com/magsther/code)

