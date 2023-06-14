---
weight: 7
keywords:
- Ansible Playbook Optimization
title: "5种途径加速Ansible模块"
date: 2023-06-11
lastmod: 2023-06-11
draft: false
authors: ["wffger"]
description: ""

tags: ["Ansible","翻译"]
categories: ["DevOps"]
lightgallery: true
---

<!--more-->
# 5种途径加速Ansible模块
> 原文[5 ways to make your Ansible modules work faster](https://www.redhat.com/sysadmin/faster-ansible-modules)

## 1. 在单模块中使用多任务，避免模块环路
人们通常都是线性思考的。例如，当你想安装多个软件包时，你可能在终端这样写代码：
```sh
# Multiple `dnf` commands
sudo dnf install httpd
sudo dnf install firewalld
sudo dnf install git
```

其实你可以更有效率：
`sudo dnf install -y httpd firewalld git`

同样，你在Ansible剧本也可以使用相同的策略。你可以传递多个软件包给一个yum任务。  
旧方法：
```yml
- name: Install httpd
  ansible.builtin.yum:
    name: httpd
 
- name: Install firewalld
  ansible.builtin.yum:
    name: firewalld
 
- name: Install git
  ansible.builtin.yum:
    name: git
```
新方法：
```yml
- name: Install Pacakages
  ansible.builtin.yum:
    name: "{{ item }}"
    state: latest
  loop:
    - httpd
    - firewalld
    - git
```
更好的方法：
```yml
- name: Install httpd and firewalld
  ansible.builtin.yum:
    name: 
      - httpd
      - firewalld
      - git
    state: latest
```

## 2.避免复制环路，使用同步模块
当你复制多个文件到同一目录时，同步模块优于使用多个复制模块或者环路：
```yml
- name: Copy application data
  synchronize:
    src: app_data/
    dest: /opt/web_app/data
```

## 3. 使用最新版本Ansbile和模块
多数情况下，新版本的Ansible在性能和优化上有提升。如果可以，使用最新版本软件以使用最新特性。同时，更新模块或角色以便获得最新特性和缺陷修复。

在推送到生产环境之前，确保你测试过最新版本的模块和Ansible。

## 4. 使用配置模板
你可能已经在单个文件中使用多个lineinfile和blockinfile去管理配置。这种方法会创建一个超长的剧本。当配置漂移发生时，你必须使用正则表达式修改lineinfile任务。其实，你可以用Jinja2模板去创建不同复杂度的文件，并且使用template
模块（或者filter）去管理节点。

举个例子，你在Jinja2模板中使用template模块去复制一个复杂的Nginx网站服务器配置。
你的Jinjia2模板：nginxd.conf.j2
```
# nginx configuration for wp-test
server {
    root /var/www/{{ website_root_dir }};
    index index.php index.html index.htm;
    server_name {{ website_name }}.com www.{{ website_name }}.com;
    access_log /var/log/nginx/access_{{ website_name }}-com.log;
    error_log /var/log/nginx/error_{{ website_name }}-com.log;
...<output removed>...
    ssl_certificate /etc/letsencrypt/live/{{ website_name }}.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/{{ website_name }}.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}
server {
    if ($host = www.{{ website_name }}.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot
...<output removed>...
    server_name {{ website_name }}.com www.{{ website_name }}.com;
    return 404; # managed by Certbot
}
```

template模块会用剧本中的值替换变量（代码中的{{  website_root_dir }}）。

```yml
---
- name: Configure the nginx Web Server
  hosts: web_servers
  become: True 
  vars:
    website_name: myawesomeblog
    website_root_dir: /var/www/myawesomeblogdata
  tasks:
    - name: Copy nginx configuration
      template:
        src: nginxd.conf.j2
        dest: /etc/nginx/sites-enabled/{{ website_name }}.conf
```

## 5. 使用适合的模块，避免使用shell或command模块
你可以使用shell或者command模块去执行一些基础的Linux命令。但我强烈建议你使用另外合适的模块，万不得已才使用shell或command模块。因为后者只是简单地执行命令，而没有相应的检验，多数情况下你都需要注意幂等性和执行错误检查。

例如，下面的第一个任务只是简单地覆盖了文件内容，并没有检查或者验证。但第二个任务仅在需要时改变文件，并且更新权限和属主信息。
```yml
    - name: Create file using shell module
      shell: 'echo "Hello" > /tmp/foo.conf'
      
    - name: Create file with permission using file module
      ansible.builtin.copy:
        content: "Hello"
        dest: /tmp/foo.conf
        owner: root
        group: root
        mode: '0644'
```









