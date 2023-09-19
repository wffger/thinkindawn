---
weight: 7
keywords:
- t3 next-auth aws congito
title: "T3App使用AWS Cognito"
date: 2023-09-19
lastmod: 2023-09-19
draft: false
authors: ["wffger"]
description: ""

tags: ["t3","nodejs","next-auth"]
categories: ["Demo"]
lightgallery: true
---

<!--more-->
# T3App使用AWS Cognito

示例仓库：[t3-pretty](https://github.com/wffger/t3-pretty)

## AWS Cognito操作步骤

1. 创建身份池，“联合身份提供商”，“SAML”
2. 在应用程序客户端中配置OAuth，“授权码授权”
3. 按照下图配置，注意回调URL的协议，区分http和https
  

{{< image src="/images/aws-cognito.png" caption="app client setting" >}}

## 修改T3工程

添加供应商 - /server/auth.ts
```
CognitoProvider({
  clientId: process.env.COGNITO_CLIENT_ID,
  clientSecret: process.env.COGNITO_CLIENT_SECRET,
  issuer: process.env.COGNITO_ISSUER,
})
```

配置环境变量 - /.env
```
COGNITO_CLIENT_ID=""
COGNITO_CLIENT_SECRET=""
COGNITO_ISSUER="https://cognito-idp.{region}.amazonaws.com/{PoolId}"
```

## 参考
[next-auth](https://next-auth.js.org/providers/cognito)