---
layout: post
title: Docker 核心概念
categories: ["docker"]
tags: ["docker"]
---

# Image

Docker 镜像是用于创建 Docker 容器的一个模板，可以将它类比为编程中 **类和对象** 的关系。Docker 镜像是一个特殊的文件系统，内部包含了运行应用所需要的所有依赖。

**内容**

Docker 镜像的内容主要包含两个部分：第一，镜像层文件内容；第二，镜像 json 文件。

**存储位置**

Docker 镜像层的内容一般在 Docker 根目录的 `aufs` 路径下，为 `/var/lib/docker/aufs/diff/`，具体情况如下：



# Container
# Docker Client
# Docker Host
# Docker Registry
# Docker Hub
# Docker Machine

# 参考资料

- [Docker 容器概念](http://jm.taobao.org/2016/05/12/introduction-to-docker/){:target="_blank"}
- [深入分析 Docker 镜像原理](http://blog.daocloud.io/principle-of-docker-image/){:target="_blank"}
- [10张图带你深入理解Docker容器和镜像](http://dockone.io/article/783){:target="_blank"}