---
weight: 7
keywords: 
- Try Out Argo Wrokflow
title: "Try Out Argo Workflow"
date: 2023-06-05
lastmod: 2023-06-05
draft: false
authors: ["wffger"]
description: ""

tags: ["Argo"]
categories: ["DevOps"]
lightgallery: true
---

Try to deploy Argo Workflow in local and run a simple workflow.

<!--more-->

## Installation and Configuration
```
# Installation
kubectl create namespace argo
kubectl apply -n argo -f https://github.com/argoproj/argo-workflows/releases/latest/download/install.yaml

# Enable server authentication
kubectl patch deployment \
  argo-server \
  --namespace argo \
  --type='json' \
  -p='[{"op": "replace", "path": "/spec/template/spec/containers/0/args", "value": [
  "server",
  "--auth-mode=server"
]}]'


# Set port forward
kubectl -n argo port-forward deploy/argo-server --address 0.0.0.0 2746:2746
```

## Start
Open UI [https://localhost:2746](https://localhost:2746)
{{< figure src="/images/argoworkflow-start.png" title="argoworkflow (figure)" >}}


## Creaet WorkflowTemplate