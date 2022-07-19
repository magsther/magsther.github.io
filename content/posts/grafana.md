---
title: "Grafana dashboards as configMaps"
date: 2022-07-19T11:53:02+02:00
draft: true
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

### Create a Grafana Dashboard
This is the last part of our mission to monitor an Elastic cluster using Kube-Prometheus.

If you remember from the previous blog post, we mentioned that we need to take five steps to do this:

1. Deploy Elasticsearch Exporter to Kubernetes.
2. Verify that the exporter can scrape metrics from our Elastic cluster.
3. Make Kube-Prometheus to **scrape the exporter.** 
4. Add Prometheus as **data source** to Grafana
5. Add a **Grafana Dashboard** to visualise the data.

In this blog post we will do step 5.

### Prerequisite
- Existing Kubernetes cluster 
- Kube-Prometheus deployed.
- Servicemonitor for elastic-exporter

You can find the blog post [here](https://magsther.github.io/posts/kubeprometheus/) if you want to see how we did it.

### What is Grafana?
Grafana is an open source solution for running data analytics, pulling up metrics that make sense of the massive amount of data & to monitor our apps with the help of cool customizable dashboards.

### Grafana dashboards as configmaps
Usuually we add Grafana dashboards using the UI. 
This is however not the preferred method when we want to automatic the infrastructure. 

To manage our Grafana dashboard in our declarative way, these are the steps we will take:

1. Create a (Kubernetes) configmap
2. Find and add a Grafana dashboard json file.
3. Add a `kubernetes manifest` resource to our Terraform configuration

**Let's start**

### Configmap
`vim elastic-dashboard-configmap.yaml`


```
apiVersion: v1
kind: ConfigMap
metadata:
  name: grafana-elastic
  namespace: monitoring
  labels:
    grafana_dashboard: "true"
data:
  elastic.json: |
    ${indent(4, data)}
```

### Find and add a Grafana dashboard json file.
Grafna provides pre-created dashboards on their website for all kind of use-cases.
We are interested in an `Elastic` dashboard, so we go to the website and search for that.

Browse through the [result](https://grafana.com/grafana/dashboards/?search=elastic) and pick one that you like.

Tip: you can look at the number of `Downloads` to find the most popular ones.

Save the dashbaord as a json file.

In my case, I saved in as `grafana-elastic-dashboard`.

Note: always read through the json file to understand what it does. You can edit the file as you see fit.

### Add a kubernetes manifest resource to Terraform
In the `main.tf` file, add the following resource.

```
resource "kubernetes_manifest" "elastic_dashboard" {
  manifest = yamldecode(templatefile("${path.module}/templates/elastic-dashboard-configmap.yaml", {
   name      = "grafana-elastic"
   data      = file("${path.module}/files/grafana-elastic-dashboard.json")
  }))
}
```

This will load the `configmap` that we created earlier and add it to Grafana.

### Deploying using Terraform
Run `terraform plan`, to verify the changes that Terraform detected.

Apply the terraform plan with `terraform apply`

After a little while **grafana dashboard** is deployed to our **Kubernetes** cluster.

Open the `Grafana UI` with : `kubectl port-forward svc/kube-prometheus-stackr-grafana 3000:80 --namespace monitoring`
and check the **elastic** dashbaord is created.

### Conclusion
In this blog post, we deployed an elastic `grafana dashboard` to display the metrics from our Elastic instance in dashboard.

> All code for this blog can be found [here[](https://github.com/magsther/code)
