---
weight: 7
keywords:
- Hashicorp Vault Entity
title: "为什么Vault使用实体概念"
date: 2023-10-09
lastmod: 2023-10-09
draft: false
authors: ["wffger"]
description: ""

tags: ["Hashicorp","Vault"]
categories: ["DevOps"]
lightgallery: true
---

<!--more-->
# 为什么Vault使用实体概念

## 我的理解
用户可能有多种身份，创建实体时绑定一个基本策略，然后创建多个别名，每个别名绑定不同的认证方法。认证方法可以绑定单独的策略。当用户使用不同的方法入口登录，具有的权限不一样，可以访问不通的秘密。这有点像切换角色。


## 官方解释
[Why create entities?](https://developer.hashicorp.com/vault/tutorials/auth-methods/identity?variants=vault-deploy%3Aselfhosted#why-create-entities)，官方解释把用户换成应用的概念，那么应用可以有不同的认证方式，例如用户名密码，OIDC，LDAP等。无论是哪种方式登录都应该视作同一个客户端，这样Hashicorp就不会重复计数，因为收费版本可能是按照客户端数量收费的。



## 认证方法
[Vault 1.15.0](https://www.github.com/hashicorp/vault/blob/main/CHANGELOG.md#)提供了14中认证方法。 

{{< image src="/images/vaultAuthMethod.png" caption="Vault Authentication Method" >}}

### Generic
1. AppRole
2. JWT
3. OIDC
4. TLS Certificates
5. Username & Password
   
### Cloud
1. AliCloud
2. AWS
3. Azure
4. Google Cloud
5. GitHub


### Infra
1. Kubernetes
2. LDAP
3. Okta
4. RADIUS



## 实践
### 获取令牌

```Bash
curl --request POST \
   --data '{"password": "wffgerqa"}' \
   $VAULT_ADDR/v1/auth/userpass-qa/login/wffgerqa \
   | jq -r ".auth.client_token" > qa_token.txt


curl --request POST \
   --data '{"password": "wffgertest"}' \
   $VAULT_ADDR/v1/auth/userpass-test/login/wffgertest \
   | jq -r ".auth.client_token" > test_token.txt
   
   ```


### 准备载荷
```Bash
tee payload_test.json <<EOF
{
   "data": {
   "designer": "ydx"
   }
}
EOF
```


### 测试
```Bash
curl --header "X-Vault-Token: $(cat test_token.txt)" \
	--request GET \
	$VAULT_ADDR/v1/secret/data/test


curl --header "X-Vault-Token: $(cat qa_token.txt)" \
	--request GET \
	$VAULT_ADDR/v1/secret/data/test
  
curl --header "X-Vault-Token: $(cat test_token.txt)" \
	--silent --output /dev/null --write-out "%{http_code}\n" \
	--request POST \
	--data @payload_test.json \
	$VAULT_ADDR/v1/secret/data/test

```


### 结果

```Bash
[wffger@ydx vault]$ cat test_token.txt 
hvs.CAESIM4UBwF-QeQSZn4N_zNloj06RTzuFOsCcMEseq2bAmVTGh4KHGh2cy5CY3FDVWxaeXphVXludWNERndiOTlKd2w
[wffger@ydx vault]$ curl --header "X-Vault-Token: $(cat qa_token.txt)" \
   --request GET \
   $VAULT_ADDR/v1/secret/data/test
{"errors":["permission denied"]}

[wffger@ydx vault]$ 
curl --header "X-Vault-Token: $(cat test_token.txt)" \
   --request GET \
   $VAULT_ADDR/v1/secret/data/test
{"request_id":"ebd2ff41-dcdf-1858-82bd-a8f522a90fe2","lease_id":"","renewable":false,"lease_duration":0,"data":{"data":{"product":"mate70","release_date":"20231012"},"metadata":{"created_time":"2023-10-09T10:31:34.664826713Z","custom_metadata":null,"deletion_time":"","destroyed":false,"version":1}},"wrap_info":null,"warnings":null,"auth":null}

[wffger@ydx vault]$ curl --header "X-Vault-Token: $(cat test_token.txt)"    --silent --output /dev/null --write-out "%{http_code}\n"    --request POST    --data @payload_test.json    $VAULT_ADDR/v1/secret/data/test
200

[wffger@ydx vault]$ curl --header "X-Vault-Token: $(cat test_token.txt)"    --request GET    $VAULT_ADDR/v1/secret/data/test                        
{"request_id":"74172cfc-7e54-39cf-264f-c6808e750fb5","lease_id":"","renewable":false,"lease_duration":0,"data":{"data":{"designer":"ydx"},"metadata":{"created_time":"2023-10-09T10:44:31.249698334Z","custom_metadata":null,"deletion_time":"","destroyed":false,"version":3}},"wrap_info":null,"warnings":null,"auth":null}
》
```