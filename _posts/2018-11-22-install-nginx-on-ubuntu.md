---
layout: post
title:  在 Ubuntu 安装 Nginx
categories: ["linux"]
tags: ["ubuntu", "nginx"]
---

# 介绍

Nginx是世界上最受欢迎的网络服务器之一，负责托管互联网上一些规模最大，流量最高的网站。在大多数情况下，它比Apache更加资源友好，可以用作Web服务器或反向代理。

在本指南中，我们将讨论如何在Ubuntu 16.04服务器上安装Nginx。

# 先决条件

在开始本指南之前，您应该有一个sudo在服务器上配置了权限的常规非root用户。您可以按照我们的Ubuntu 16.04初始服务器设置指南了解如何配置常规用户帐户。

如果您有可用的帐户，请以非root用户身份登录以开始。

# 第1步：安装Nginx

Nginx可以在Ubuntu的默认存储库中使用，因此安装非常简单。

由于这是我们apt在此会话中与包装系统的第一次互动，我们将更新我们的本地包索引，以便我们可以访问最新的包列表。之后，我们可以安装nginx：

```shell
sudo apt-get update
sudo apt-get install nginx
```

接受该过程后，apt-get将Nginx和任何所需的依赖项安装到您的服务器。

# 第2步：调整防火墙

在我们测试Nginx之前，我们需要重新配置防火墙软件以允许访问该服务。ufw在安装时，Nginx将自己注册为防火墙服务。这使得允许Nginx访问变得相当容易。

我们可以ufw通过键入以下内容列出知道如何使用的应用程序配置：

```shell
sudo ufw app list
```

您应该获得应用程序配置文件的列表：

```shell
Output
Available applications:
  Nginx Full
  Nginx HTTP
  Nginx HTTPS
  OpenSSH
```

如您所见，Nginx有三种配置文件：

- **Nginx Full**：此配置文件打开端口80（正常，未加密的Web流量）和端口443（TLS / SSL加密流量）
- **Nginx HTTP**：此配置文件仅打开端口80（正常，未加密的Web流量）
- **Nginx HTTPS**：此配置文件仅打开端口443（TLS / SSL加密流量）

建议您启用限制性最强的配置文件，该配置文件仍允许您配置的流量。由于我们尚未为我们的服务器配置SSL，因此在本指南中，我们只需要允许端口80上的流量。

您可以输入以下命令启用此功能

```shell
sudo ufw allow 'Nginx HTTP'
```

您可以键入以下内容来验证更改：

```shell
sudo ufw status
```

您应该在显示的输出中看到允许的HTTP流量：

```shell
Output
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere                  
Nginx HTTP                 ALLOW       Anywhere                  
OpenSSH (v6)               ALLOW       Anywhere (v6)             
Nginx HTTP (v6)            ALLOW       Anywhere (v6)
```

# 第3步：检查您的Web服务器

在安装过程结束时，Ubuntu 16.04启动Nginx。Web服务器应该已经启动并运行。

我们可以systemd通过键入以下内容来检查init系统以确保服务正在运行：

```shell
systemctl status nginx
Output
● nginx.service - A high performance web server and a reverse proxy server
   Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
   Active: active (running) since Mon 2016-04-18 16:14:00 EDT; 4min 2s ago
 Main PID: 12857 (nginx)
   CGroup: /system.slice/nginx.service
           ├─12857 nginx: master process /usr/sbin/nginx -g daemon on; master_process on
           └─12858 nginx: worker process
```

如您所见，该服务似乎已成功启动。但是，测试它的最佳方法是从Nginx实际请求页面。

您可以访问默认的Nginx登录页面以确认软件正常运行。您可以通过服务器的域名或IP地址访问它。

如果您没有为服务器设置域名，可以在此处了解如何使用DigitalOcean设置域名。

如果您不想为服务器设置域名，则可以使用服务器的公共IP地址。如果您不知道服务器的IP地址，可以从命令行获得几种不同的方法。

尝试在服务器的命令提示符下键入：

```shell
ip addr show eth0 | grep inet | awk '{ print $2; }' | sed 's/\/.*$//'
```

你会回来几行。您可以在Web浏览器中尝试各自以查看它们是否有效。

另一种方法是键入此内容，它应该为您提供从Internet上其他位置看到的公共IP地址：

```shell
sudo apt-get install curl
curl -4 icanhazip.com
```

获得服务器的IP地址或域后，将其输入浏览器的地址栏：

```shell
http://server_domain_or_IP
```

你应该看到默认的Nginx登陆页面，它应该是这样的：

![](https://assets.digitalocean.com/articles/nginx_1604/default_page.png)

此页面仅包含在Nginx中，以向您显示服务器正在正常运行。

# 第4步：管理Nginx流程

现在您已启动并运行Web服务器，我们可以查看一些基本的管理命令。

要停止Web服务器，可以键入：

```shell
sudo systemctl stop nginx
```

要在Web服务器停止时启动它，请键入：

```shell
sudo systemctl start nginx
```

要停止然后再次启动该服务，请键入：

```shell
sudo systemctl restart nginx
```

如果您只是进行配置更改，Nginx通常可以在不丢弃连接的情况下重新加载。为此，可以使用此命令：

```shell
sudo systemctl reload nginx
```

默认情况下，Nginx配置为在服务器引导时自动启动。如果这不是您想要的，您可以通过键入以下内容来禁用此行为：

```shell
sudo systemctl disable nginx
```

要重新启用服务以在启动时启动，您可以键入：

```shell
sudo systemctl enable nginx
```

# 第5步：熟悉重要的Nginx文件和目录

既然您已经知道如何管理服务本身，那么您应该花几分钟时间熟悉一些重要的目录和文件。

## 内容

- `/var/www/html`：实际的Web内容（默认情况下仅包含您之前看到的默认Nginx页面）是从/var/www/html目录中提供的。这可以通过更改Nginx配置文件来更改。

## 服务器配置

- `/etc/nginx`：Nginx配置目录。所有Nginx配置文件都驻留在此处。
- `/etc/nginx/nginx.conf`：主要的Nginx配置文件。可以对此进行修改以更改Nginx全局配置。
- `/etc/nginx/sites-available/`：可以存储每个站点“服务器块”的目录。Nginx不会使用此目录中的配置文件，除非它们链接到sites-enabled目录（见下文）。通常，所有服务器块配置都在此目录中完成，然后通过链接到其他目录来启用。
- `/etc/nginx/sites-enabled/`：存储每个站点“服务器块”启用的目录。通常，这些是通过链接到sites-available目录中的配置文件来创建的。
- `/etc/nginx/snippets`：此目录包含可以包含在Nginx配置中其他位置的配置片段。可能可重复的配置段是重构为片段的良好候选者。

## 服务器日志

- `/var/log/nginx/access.log`：除非Nginx配置为执行其他操作，否则对Web服务器的每个请求都将记录在此日志文件中。
- `/var/log/nginx/error.log`：任何Nginx错误都将记录在此日志中。

# 结论

现在您已经安装了Web服务器，您可以选择要提供的内容类型以及要用于创建更丰富体验的技术。
