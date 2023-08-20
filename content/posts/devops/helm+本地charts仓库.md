---
weight: 7
keywords:
- helm charts
title: "helm+本地charts仓库"
date: 2023-08-20
lastmod: 2023-08-20
draft: false
authors: ["wffger"]
description: ""

tags: ["k8s"]
categories: ["devops"]
lightgallery: true
---

<!--more-->
# helm+本地charts仓库
## 创建本地仓库
### [ChartMuseum](https://chartmuseum.com/docs/#installation)
```bash
podman run --privileged \
  --rm -it \
  -p 8000:8080 \
  -e DEBUG=1 \
  -e STORAGE=local \
  -e STORAGE_LOCAL_ROOTDIR=/charts \
  -v $(pwd)/charts:/charts \
  chartmuseum/chartmuseum:latest
```
 [Container permission denied: How to diagnose this error](https://www.redhat.com/sysadmin/container-permission-denied-errors)
### python web服务器
```bash
mkdir -p repo_py/web/charts
cd repo_py
vi app.py

## 
py app.py
```

```python
import http.server
import socketserver

PORT = 8001
DIRECTORY = "web"
 
class YamlHandler(http.server.SimpleHTTPRequestHandler):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, directory=DIRECTORY, **kwargs)

YamlHandler.extensions_map={
    '.yaml': 'text/x-yaml',
    '.manifest': 'text/cache-manifest',
    '.html': 'text/html',
    '.png': 'image/png',
    '.jpg': 'image/jpg',
    '.svg':	'image/svg+xml',
    '.css':	'text/css',
    '.js':	'application/x-javascript',
    '': 'application/octet-stream', # Default
}

with socketserver.TCPServer(("", PORT), YamlHandler) as httpd:
    print("serving at port", PORT)
    httpd.allow_reuse_address=1
    httpd.serve_forever()
```
## helm配置
```bash
helm repo list 
helm repo add chartmuseum http://localhost:8000
helm repo add repo_py http://localhost:8001/charts
helm repo list
```
## chart
### 创建chart
```bash
mkidr charts
cd charts
helm create wffgerapp
# 编辑内容

# 打包
helm package wffgerapp/
# 产出wffgerapp-x.x.x.tgz
```
### 推送chart

#### ChartMuseum
```bash
cd charts
helm cm-push wffgerapp/ chartmuseum
helm repo update
```
#### Python web server
```bash
# 手动推送软件包
cp wffgerapp-x.x.x.tgz /repo_py/web/charts

# 更新索引
cd repo_py/web/
helm repo index charts/ --url http://localhost:8001/charts
```

## 部署chart
启动minikube后，helm会从~/.kube/config找到k8s集群信息。<br />https://<INTERNAL-IP>:8443/version?timeout=32s
```bash
helm install test-wffgerapp chartmuseum/wffgerapp
kubectl get pod
```
