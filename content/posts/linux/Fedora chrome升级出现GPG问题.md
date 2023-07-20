---
weight: 7
keywords:
- rpm google-chrome gpg
title: "Fedora chrome升级出现GPG问题"
date: 2023-07-20
lastmod: 2023-07-20
draft: false
authors: ["wffger"]
description: ""

tags: ["rpm"]
categories: ["Linux"]
lightgallery: true
---

<!--more-->
# Fedora chrome升级出现GPG问题
## 错误
```
仓库 "google-chrome" 的 GPG 公钥已安装，但是不适用于此软件包。
请检查此仓库的公钥 URL 是否配置正确。. 失败的软件包是：google-chrome-stable-115.0.5790.98-1.x86_64
 GPG密钥配置为：https://dl.google.com/linux/linux_signing_key.pub
下载的软件包保存在缓存中，直到下次成功执行事务。
您可以通过执行 'dnf clean packages' 删除软件包缓存。
错误：GPG 检查失败
```

## 原因

[remove-obsolete-gpg-key-from-dnf](https://gist.github.com/e7d/3b786c7410ca14a5ded61eec36de9874)

```
sudo rpm -q gpg-pubkey --qf '%{NAME}-%{VERSION}-%{RELEASE}\t%{SUMMARY}\n' | grep google
gpg-pubkey-7fac5991-4615767f	Google, Inc. Linux Package Signing Key <linux-packages-keymaster@google.com> public key
gpg-pubkey-d38b4796-570c8cd3	Google Inc. (Linux Packages Signing Authority) <linux-packages-keymaster@google.com> public key
```
安装了两个google密钥，其中7fac5991是废弃的。你也可以两个都移除。
[Google Linux Package Signing Keys ](https://www.google.com/linuxrepositories/)

## 修复
```
# 清除废弃的密钥
sudo rpm -e gpg-pubkey-7fac5991-4615767f

# 升级chrome
sudo dnf update -y

# 检查新的密钥
sudo rpm -q gpg-pubkey --qf '%{NAME}-%{VERSION}-%{RELEASE}\t%{SUMMARY}\n' | grep google
gpg-pubkey-7fac5991-45f06f46	Google, Inc. Linux Package Signing Key <linux-packages-keymaster@google.com> public key
gpg-pubkey-d38b4796-570c8cd3	Google Inc. (Linux Packages Signing Authority) <linux-packages-keymaster@google.com> public key
```
你会发现7fac5991又回来了，但是后面跟着的字符串变了。
难道新的7fac5991-45f06f46就是用d38b4796-570c8cd3创建的吗？Google官文说会周期轮换密钥，难道以后我都要手动删除旧的subkey？

## 延伸
[GPG signing](https://docs.digicert.com/en/software-trust-manager/signing-tools/gpg-signing.html)