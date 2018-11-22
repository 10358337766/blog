---
layout: post
title:  在 CentOS 7 安装 Nginx
categories: ["linux"]
tags: ["centos", "nginx"]
---

Nginx（Engine X 的缩写）是一个免费、开源、功能强大的 Web 服务器，基于事件驱动（异步）架构的反向代理。使用 C 编程语言编写，可在类 Unix 操作系统和 Windows 系统上运行。

它还可用作反向代理、邮件和 TCP/UDP 代理服务器，也可以用来配置为负载均衡。它为互联网上大多数站点提供支持；以高性能、稳定性和功能丰富而闻名。

在本文中，我们将介绍如何在 CentOS 7 服务器上安装、配置和管理 Nginx Web 服务器。

# 准备条件

- 一台 CentOS 7 的服务器
- 提供一个可访问的静态 IP

# 安装 Nginx

首先将系统软件包更新到最新版本

```shell
yum -y update
```

接下来，使用 `yum` 包管理器从 `EPEL` 存储库安装 `Nginx`，如下所示。

```shell
yum install epel-release
yum install nginx 
```

![](/assets/images/2018/11/centos_install_nginx.png)

# 管理 Nginx 服务

安装好 Nginx 后，在第一次启动的时候可以配置让它开机自动启动。

```shell
systemctl start nginx
systemctl enable nginx
systemctl status nginx
```

![](/assets/images/2018/11/centos_nginx_status.png)

# 配置 firewalld 以允许 Nginx 流量

默认情况下，CentOS 7 内置防火墙设置不开放 http 端口，所以 nginx 暴露的服务无法访问。要允许 Nginx 上的流量，使用以下命令更新系统防火墙规则，来允许 HTTP 和 HTTPS 上的入站数据包。

```shell
firewall-cmd --zone=public --permanent --add-service=http
firewall-cmd --zone=public --permanent --add-service=https
firewall-cmd --reload
```

![](/assets/images/2018/11/centos_firewall_reload.png)

如果服务器厂商有 Web 界面可以调整防火墙策略，你也可以在服务器里禁用防火墙

```shell
systemctl stop firewalld
systemctl disable firewalld
```

# 测试 Nginx

现在你可以打开 URL 来验证 Nginx 是否成功运行，不出意外你会看到默认的 Nginx 页面。

```shell
http://SERVER_DOMAIN_NAME_OR_IP
```

![](/assets/images/2018/11/centos_nginx_welcome.png)

# Nginx 核心文件和目录

- 默认服务器根目录（包含配置文件的顶级目录）：`/ etc/nginx`
- 主要的 Nginx 配置文件：`/etc/nginx/nginx.conf`。
- 可以在 `/etc/nginx/conf.d` 中添加服务器块（虚拟主机）配置。
- 默认服务器文档根目录（包含Web文件）：`/usr/share/nginx/html`。
