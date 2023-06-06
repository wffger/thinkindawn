---
weight: 7
title: "试用Argo Workflow"
date: 2023-06-05
lastmod: 2023-06-05
draft: false
authors: ["wffger"]
description: ""

tags: ["Argo"]
categories: ["DevOps"]

lightgallery: true
---

初次尝试本地部署Argo Workflow.

<!--more-->

## 安装配置
```
# 安装
kubectl create namespace argo
kubectl apply -n argo -f https://github.com/argoproj/argo-workflows/releases/latest/download/install.yaml

# 配置免除登录
kubectl patch deployment \
  argo-server \
  --namespace argo \
  --type='json' \
  -p='[{"op": "replace", "path": "/spec/template/spec/containers/0/args", "value": [
  "server",
  "--auth-mode=server"
]}]'


# 配置端口转发
kubectl -n argo port-forward deploy/argo-server --address 0.0.0.0 2746:2746
```

## 启动
打开入口地址[https://localhost:1313](https://localhost:1313)
{{< figure src="/images/argoworkflow-start.png" title="argoworkflow (figure)" >}}
