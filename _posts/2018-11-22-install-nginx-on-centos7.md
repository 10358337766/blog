---
layout: post
title:  在 CentOS 7 安装 Nginx
categories: ["linux"]
tags: ["centos", "nginx"]
---

NGINX（Engine X的缩写）是一个免费的，开源的，功能强大的 HTTP Web 服务器和具有事件驱动（异步）架构的反向代理。它使用 C 编程语言编写，可在类 Unix 操作系统和 Windows 操作系统上运行。

它还可用作反向代理，标准邮件和 TCP/UDP 代理服务器，还可以配置为负载均衡器。它为网络上的许多网站提供动力; 以其高性能，稳定性和功能丰富的设备而闻名。

在本文中，我们将介绍如何使用命令行在 CentOS 7 或 RHEL 7 服务器上安装，配置和管理 Nginx HTTP Web 服务器。

# 先决条件：

- CentOS 7服务器最小安装
- RHEL 7 服务器最小安装
- 具有静态 IP 地址的 CentOS/RHEL 7 系统

# 安装 Nginx Web 服务器

1.首先将系统软件包更新到最新版本。

```shell
yum -y update
```

2.接下来，使用YUM包管理器从EPEL存储库安装Nginx HTTP服务器，如下所示。

```shell
yum install epel-release
yum install nginx 
```

![](https://www.tecmint.com/wp-content/uploads/2017/07/Install-Nginx-on-CentOS-7.png)

# 在CentOS 7上管理Nginx HTTP Server

安装Nginx Web服务器后，您可以第一次启动它并使其在系统引导时自动启动。

```shell
systemctl start nginx
systemctl enable nginx
systemctl status nginx
```

![](https://www.tecmint.com/wp-content/uploads/2017/07/Start-and-Enable-Nginx-at-Boot.png)

# 配置firewalld以允许Nginx流量

默认情况下，CentOS 7内置防火墙设置为阻止Nginx流量。要允许Nginx上的Web流量，请使用以下命令更新系统防火墙规则以允许HTTP和HTTPS上的入站数据包。

```shell
＃firewall-cmd --zone = public --permanent --add-service = http
＃firewall-cmd --zone = public --permanent --add-service = https
＃firewall-cmd --reload
```

![](https://www.tecmint.com/wp-content/uploads/2017/08/Allow-Nginx-on-Firewalld.png)

# 在 CentOS 7 上测试 Nginx 服务器

现在您可以通过转到以下URL来验证Nginx服务器，将显示默认的nginx页面。

```shell
HTTP：// SERVER_DOMAIN_NAME_OR_IP 
```

# Nginx重要文件和目录

- 默认服务器根目录（包含配置文件的顶级目录）：`/ etc/nginx`。
- 主要的Nginx配置文件：`/etc/nginx/nginx.conf`。
- 可以在 `/etc/nginx/conf.d` 中添加服务器块（虚拟主机）配置。
- 默认服务器文档根目录（包含Web文件）：`/usr/share/nginx/html`。

