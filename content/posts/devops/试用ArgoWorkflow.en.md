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
```sh
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
```yml
metadata:
  name: omniscient-bear
  namespace: pony
  uid: b5452263-dabc-4e5d-a0be-1f2ea4710372
  resourceVersion: '169216'
  generation: 3
  creationTimestamp: '2023-06-07T03:33:11Z'
  labels:
    example: 'true'
    workflows.argoproj.io/creator: system-serviceaccount-argo-argo-server
  annotations:
    author: wffger
  managedFields:
    - manager: argo
      operation: Update
      apiVersion: argoproj.io/v1alpha1
      time: '2023-06-07T03:33:11Z'
      fieldsType: FieldsV1
      fieldsV1:
        f:metadata:
          f:annotations:
            .: {}
            f:author: {}
          f:labels:
            .: {}
            f:example: {}
            f:workflows.argoproj.io/creator: {}
        f:spec: {}
spec:
  templates:
    - name: argosay
      inputs:
        parameters:
          - name: message
            value: '{{workflow.parameters.message}}'
      outputs: {}
      metadata: {}
      container:
        name: main
        image: argoproj/argosay:v2
        command:
          - /argosay
        args:
          - echo
          - '{{inputs.parameters.message}}'
        resources: {}
  entrypoint: argosay
  arguments:
    parameters:
      - name: message
        value: hello argo
  workflowMetadata:
    labels:
      example: 'true'

```