---
layout: post
title: 使用 Aria2 + rclone 遇到的一些问题
categories: ["vps"]
tags: ["rclone", "离线下载"]
---

在发布了 [Aria2 + rclone 实现 Google Drive 离线下载](https://www.youtube.com/watch?v=whAAyKd58gg){:target="_blank"} 视频后发现很多朋友遇到了一些问题，博主在演示的时候也忽略了 VPS 上的文件未删除占用磁盘问题，在评论中看到有位叫 `@陈潮潮` 的网友给出一个不错的思路，在本文中我将优化它们。

# 什么问题？

我们使用 aria2 下载完文件后在 `/data/Download` 目录，然后执行了一个脚本将文件移动到 `/data/GoogleDrive` 目录（该目录是挂载目录）。这时候文件会自动的同步到谷歌云盘，但是问题来了，假设我们的 VPS 只有 10G 硬盘，这个挂载目录不能一直存文件吧，否则的话不就 GG 了。当时博主确实没考虑到这个问题，所以我们可以换一种思路去解决它。

# rclone 命令

先来看看 rclone 这个工具的常用命令：

- `rclone config` - 以控制会话的形式添加rclone的配置，配置保存在.rclone.conf文件中。
- `rclone copy` - 将文件从源复制到目的地址，跳过已复制完成的。
- `rclone sync` - 将源数据同步到目的地址，只更新目的地址的数据。
- `rclone move` - 将源数据移动到目的地址。
- `rclone delete` - 删除指定路径下的文件内容。
- `rclone purge` - 清空指定路径下所有文件数据。
- `rclone mkdir` - 创建一个新目录。
- `rclone rmdir` - 删除空目录。
- `rclone check` - 检查源和目的地址数据是否匹配。
- `rclone ls` - 列出指定路径下所有的文件以及文件大小和路径。
- `rclone lsd` - 列出指定路径下所有的目录/容器/桶。
- `rclone lsl` - 列出指定路径下所有文件以及修改时间、文件大小和路径。
- `rclone md5sum` - 为指定路径下的所有文件产生一个md5sum文件。
- `rclone sha1sum` - 为指定路径下的所有文件产生一个sha1sum文件。
- `rclone size` - 获取指定路径下，文件内容的总大小。.
- `rclone version` - 查看当前版本。
- `rclone cleanup` - 清空remote。
- `rclone dedupe` - 交互式查找重复文件，进行删除/重命名操作。

我们可以看使用 `rclone move` 就可以直接将 aria2 下载的文件上传上去，这里 `@陈潮潮` 把他写的脚本开源出来，放在 Github 了，我们可以拿过来使用，非常感谢这位朋友。

# 取消挂载

因为这个思路是直接移动，所以之前的挂载就不用了，我们取消挂载。

```shell
rm -f /etc/init.d/rcloned
fusermount -u /data/GoogleDrive
```

然后删除 `vim /etc/rc.d/rc.local` 中的 `bash /etc/init.d/rcloned start` 这一行。

这样就完成了取消挂载。

# 修改脚本

这个时候就不能用之前的脚本了，我们直接使用上面这位朋友提供的脚本。

```shell
> /root/rcloneupload.sh
vim /root/rcloneupload.sh
```

将之前的上传脚本清空后重新编辑

```shell
#!/bin/bash

filepath=$3	 #取文件原始路径，如果是单文件则为/Download/a.mp4，如果是文件夹则该值为文件夹内第一个文件比如/Download/a/1.mp4
path=${3%/*}	 #取文件根路径，如把/Download/a/1.mp4变成/Download/a
downloadpath='/data/Download'	#Aria2下载目录
name='codesofun' #配置Rclone时的name
folder='/share'	 #网盘里的文件夹，如果是根目录直接留空
MinSize='10k'	 #限制最低上传大小，默认10k，BT下载时可防止上传其他无用文件。会删除文件，谨慎设置。
MaxSize='15G'	 #限制最高文件大小，默认15G，OneDrive上传限制。

if [ $2 -eq 0 ]; then exit 0; fi

while true; do
if [ "$path" = "$downloadpath" ] && [ $2 -eq 1 ]	#如果下载的是单个文件
    then
    rclone move -v "$filepath" ${name}:${folder} --min-size $MinSize --max-size $MaxSize
    rm -vf "$filepath".aria2	#删除残留的.aria.2文件
    exit 0
elif [ "$path" != "$downloadpath" ]	#如果下载的是文件夹
    then
    while [[ "`ls -A "$path/"`" != "" ]]; do
    rclone move -v "$path" ${name}:/${folder}/"${path##*/}" --min-size $MinSize --max-size $MaxSize --delete-empty-src-dirs
    rclone delete -v "$path" --max-size $MinSize	#删除多余的文件
    rclone rmdirs -v "$downloadpath" --leave-root	#删除空目录，--delete-empty-src-dirs 参数已实现，加上无所谓。
    done
    rm -vf "$path".aria2	#删除残留的.aria2文件
    exit 0
fi
done
```

需要修改几个地方：

- `downloadpath`：填写 aria2 的下载目录
- `name`：填写 rclone 配置的名称
- `folder`：填写谷歌云盘的文件夹名称，根目录填 `/` 即可

# FAQ

在评论中还看到很多朋友出了一些问题，这里我们列出一些常见问题如何解决。

## 如何实现多个网盘之间文件拷贝？

我们使用 `rclone config` 可以创建一个名为 `codesofun` 的配置，当然也能创建多个，你需要再创建第二个网盘的配置，即便是 Google Drive 同步到 OneDrive 也是可以的，假设第二个名字为 `hello`。

执行命令 `rclone copy codesofun:share hello:share` 即可。

> `rclone copy source:sourcepath dest:destpsth`
> 
> rclone copy 复制总是指定路径下的数据；而不是当前目录。
> –no-traverse 标志用于控制是否列出目的地址目录。

如果文件比较多或者较大的话建议使用 `screen` 命令开启一个会话执行，这里使用 `share` 目录的原因是一个习惯而已，不加目录就是整个网盘咯。

## 浏览器访问 ip 出现 403

原因可能有 3 种

**权限不够**

解决方法 `chmod -R 755 /data/www`

**开启了 SELinux**

使用 `/usr/sbin/sestatus -v` 查看是否开启，输出 `enable` 则为开启。

解决方法 修改 `/etc/selinux/config` 将 `SELINUX=enforcing` 改为 `SELINUX=disabled`，然后使用 `reboot` 重启系统。

## AriaNg 一直连接不成功

AriaNg 是一个网页程序，它的配置主要是 RPC 相关的，这配置里也就是一个 IP、端口和密钥。

配置后打开浏览器的开发者工具可以在 `network` 选项里看到不停的会有请求发送，正常会返回 `200` 状态码。一般如果连接不上很可能就是防火墙拦截或者 aria2 根本没开启，因为只有这样才会导致网站端无法访问到，其他更复杂的原因要看具体情况。

我们可以在自己本地使用 `telnet` 命令测试一下端口是否暴露：

```shell
telnet ip 6800
```

这里的 `ip` 是你 VPS 的外网ip，端口默认是 `6800` 如果修改了就用自己修改的。正常情况会得到这样的输出：

```shell
telnet 66.42.xxx.232 6800
Trying 66.42.xxx.232...
Connected to 66.42.xxx.232.vultr.com.
Escape character is '^]'.
```

我们随便写个端口肯定不通，会得到这样的输出：

```shell
telnet 66.42.xxx.232 6899
Trying 66.42.xxx.232...
telnet: connect to address 66.42.xxx.232: Connection refused
telnet: Unable to connect to remote host
```

解决方法是在 VPS 上开放这个端口或关闭防火墙

```shell
systemctl stop firewalld
```

## Mac 怎么安装使用 aria2 本地下载？

下载 [aria2gui](https://github.com/yangshun1029/aria2gui){:target="_blank"} 配合插件使用即可，可参考使用说明。

# 视频版教程

<iframe width="680" height="415" src="https://www.youtube.com/embed/whAAyKd58gg" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

# 参考资料

- [Rclone: 超好用的网盘 / VPS 数据同步、备份工具](https://www.zrj96.com/post-520.html){:target="_blank"}
- [aria2 完美配置](https://github.com/P3TERX/aria2_perfect_config){:target="_blank"}
