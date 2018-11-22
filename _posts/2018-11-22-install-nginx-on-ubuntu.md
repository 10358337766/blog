---
layout: post
title:  在 Ubuntu 安装 Nginx
categories: ["linux"]
tags: ["ubuntu", "nginx"]
---

Nginx 是世界上最受欢迎的 Web 服务器之一，负责托管互联网上一些规模最大、流量最高的网站。在大多数情况下，它比 Apache 的资源利用率更高，可以用作 Web 服务器或反向代理。

在本文中，我们将介绍如何在 `Ubuntu 16.04` 服务器上安装 Nginx。

# 先决条件

在开始阅读本文之前，你应该有一个非 `root` 用户（当然 root 也是可以的）。你可以按照我们的 Ubuntu 16.04 初始服务器设置指南了解如何配置常规用户。

如果你有可用的帐户，尽可能使用非 root 用户身份来登录。

# 第一步：安装 Nginx

在 Ubuntu 的默认存储库中已经有 Nginx 了，所以安装很简单。

因为这是我们登录后的第一次操作，所以可以先更新下本地的包索引，以便于访问到最新的包列表。然后安装 nginx：

```shell
sudo apt-get update
sudo apt-get install nginx
```

![](/assets/images/2018/11/ubuntu_install_nginx.png)

接受该过程后，`apt-get` 将 `Nginx` 和任何所需的依赖项安装到您的服务器。

# 第2步：配置防火墙

在验证 Nginx 之前，先配置一下服务器的防火墙允许我们访问 Nginx 暴露的服务。在安装 nginx 的时候已经安装了 `ufw` 防火墙服务，让我们更容易的配置访问 Nginx。

首先开启 `ufw` 管理防火墙，然后查看当前服务器的应用配置列表：

```shell
sudo ufw enable
sudo ufw app list
```

你应该会看到如下的列表：

```shell
Available applications:
  Nginx Full
  Nginx HTTP
  Nginx HTTPS
  OpenSSH
```

和你看到的一样，在 Nginx 中有这三种防火墙配置文件：

- **Nginx Full**：该配置允许打开 80 端口和 443 端口
- **Nginx HTTP**：该配置仅打开 80 端口
- **Nginx HTTPS**：该配置仅打开 443 端口

简单起见，我们直接暴露 `80` 和 `443` 端口，以免添加了 `SSL` 证书忘记开启此配置。

```shell
sudo ufw allow 'Nginx Full'
```

输入以下命令验证修改：

```shell
sudo ufw status
```

你应该在控制台看到类似这样的输出：

```shell
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere                  
Nginx Full                 ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)             
Nginx Full (v6)            ALLOW       Anywhere (v6)
```

# 第3步：检查 Web 服务器

安装结束后，Ubuntu 会自动启动 Nginx，这时候已经在运行了。我们可以使用 `systemd` 来检查 nginx 服务运行状态：


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

根据上述输出，可以看到 nginx 服务已经成功启动，我们可以打开浏览器访问 IP 试试。

```shell
http://server_domain_or_IP
```

![](/assets/images/2018/11/ubuntu_nginx_welcome.png)

你应该也会看到这样的欢迎页面，这是默认的访问网页，证明 nginx 已经成功运行了！

# 第4步：管理 Nginx 服务

现在 nginx 已经启动并运行，我们还需要了解一下基本的管理命令。

想要停止 nginx 可以输入：

```shell
sudo systemctl stop nginx
```

停止后想启动 nginx 可以输入：

```shell
sudo systemctl start nginx
```

重启 nginx 服务请输入：

```shell
sudo systemctl restart nginx
```

如果你只是修改了配置，Nginx 可以在不关闭连接的情况下重新加载配置，输入命令：

```shell
sudo systemctl reload nginx
```

默认情况，nginx 会在服务器开机的时候自动启动。如果你不想让它自启动，可以输入以下命令禁止：

```shell
sudo systemctl disable nginx
```

反之，需要的话可以输入如下命令：

```shell
sudo systemctl enable nginx
```

# 第5步：Nginx 配置文件和目录

前面我们知道如何管理 nginx 服务了，下面应该了解一下它的重要目录和配置文件。

## 内容

- `/var/www/html`：实际的 Web 内容（默认情况下仅包含您之前看到的默认 Nginx 页面）是从`/var/www/html` 目录中提供的。可以通过修改 nginx 配置来改变这个目录。

## 服务器配置

- `/etc/nginx`：Nginx 配置目录，所有 Nginx 配置文件都存在这里。
- `/etc/nginx/nginx.conf`：Nginx 主配置文件，修改它会导致全局生效。
- `/etc/nginx/sites-available/`：可以存储每个站点 server 的目录。Nginx 不会使用此目录中的配置文件，除非它们链接到 `sites-enabled` 目录（见下文）。一般所有虚拟主机配置都在该目录中完成，然后通过链接到其他目录来启用。
- `/etc/nginx/sites-enabled/`：存储每个站点 server 块的配置目录。一般这个目录中的文件都是通过链接到 `sites-available` 中的配置文件来创建的。
- `/etc/nginx/snippets`：这个目录中包含 nginx 中一些配置的片段，把那些重复性较多的配置放在这里是个好主意。

## 服务器日志

- `/var/log/nginx/access.log`：除非你修改了 Nginx 配置，否则 nginx 的每次请求都会记录在该日止文件中。
- `/var/log/nginx/error.log`：所有的 nginx 错误都会记录在这里。
