---
layout: post
title: 安装 Docker
categories: ["docker"]
tags: ["docker"]
---

Docker 在 Windows、Linux、Mac 下的安装。

# 在 Windows 安装 Docker

## Windows 7

在 Win7 系统安装 Docker 需要使用 [docker toolbox](https://docs.docker.com/toolbox/toolbox_install_windows/){:target="_blank"}。

Docker Toolbox 包括以下 Docker 工具：

- Docker CLI：客户端，用于运行 Docker Engine 以创建映像和容器
- Docker Machine：你可以从 Windows 终端运行 Docker Engine 命令
- Docker Compose：用于运行 `docker-compose` 命令
- Kitematic：Docker GUI
- 为 Docker 命令行环境预配置的 `Docker QuickStart shell`
- `Oracle VM VirtualBox`

## Windows 10

下载 [Docker CE for Windows](https://store.docker.com/editions/community/docker-ce-desktop-windows){:target="_blank"} 双击运行即可。

# 在 Mac 安装 Docker

下载 [Docker CE for Mac](https://store.docker.com/editions/community/docker-ce-desktop-mac){:target="_blank"} 双击运行即可。

# 在 Linux 安装 Docker

## 在 Ubuntu 安装

**卸载旧版本**

```shell
sudo apt-get remove docker docker-engine docker.io
```

**使用脚本安装**

```shell
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

如果想将 Docker 用作非 root 用户，可以将用户添加到 `docker` 组，如：

```shell
sudo usermod -aG docker your-user
```

注销重新登录才会生效！

> 将用户添加到 `docker` 组可以运行容器，该容器可用于获取 docker 主机上的 root 权限。

**启动、停止、重启**

- `sudo service docker start`
- `sudo service docker stop`
- `sudo service docker restart`

**卸载 Docker CE**

1. 卸载 Docker CE 软件包：

```shell
sudo apt-get purge docker-ce
```

2. 主机上的镜像，容器，数据卷或自定义配置文件不会自动删除。要删除所有镜像，容器和数据卷：

```shell
sudo rm -rf /var/lib/docker
```

你需要手动删除任何已编辑的配置文件。

## 在 CentOS 安装

**卸载旧版本**

```shell
sudo yum remove docker \
                docker-client \
                docker-client-latest \
                docker-common \
                docker-latest \
                docker-latest-logrotate \
                docker-logrotate \
                docker-selinux \
                docker-engine-selinux \
                docker-engine
```

**使用脚本安装**

```shell
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

如果想将 Docker 用作非 root 用户，可以将用户添加到 `docker` 组，如：

```shell
sudo usermod -aG docker your-user
```

注销重新登录才会生效！

> 将用户添加到 `docker` 组可以运行容器，该容器可用于获取 docker 主机上的 root 权限。

**卸载 Docker CE**

1. 卸载 Docker CE 软件包：

```shell
sudo yum remove docker-ce
```

2. 主机上的镜像，容器，数据卷或自定义配置文件不会自动删除。要删除所有镜像，容器和数据卷：

```shell
sudo rm -rf /var/lib/docker
```

你需要手动删除任何已编辑的配置文件。

# 参考资料

- [Docker Docs](https://docs.docker.com/install/){:target="_blank"}