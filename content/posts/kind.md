---
title: "Kind & Terraform"
date: 2022-07-09T19:50:17+02:00
draft: false
toc: false
images:
tags:
  - untagged
---

## Kind
*[kind](https://kind.sigs.k8s.io/) is a tool for running local Kubernetes clusters using Docker container “nodes”.
kind was primarily designed for testing Kubernetes itself, but may be used for local development or CI.*

We still start a series of articles of how to manage **Kubernetes** resources via **Terraform**.

First we need a **Kubernetes** cluster running locally. There are many tools to pick from (**minikube**, **K3s**, **MicroK8s** etc..) , and the tool we will use is **Kind**.

To use kind, you will also need to install **docker**.

Even though we can easily create a new cluster by running `kind create cluster` we will be doing this by using **Terraform**