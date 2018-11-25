---
layout: post
title:  AriaNg + Google Drive 实现离线下载
categories: ["vps"]
tags: ["aria2", "下载"]
---

有朋友发邮件咨询如何搭建 Google Drive 的离线下载，这样的需求一般是因为我们的 VPS 硬盘容量有限，而老司机们的资源是非常多的。。。所以一般采取的思路是将资源通过 VPS 下载后上传到云存储，但普通的下载肯定不行啊，速度跟不上、可下载的格式也有限，下面我们介绍非常实用的方式搞定这一切。

本文介绍的是在 Google Drive 上操作，当然在 OneDrive 也是可以的，至于怎么撸 Google Drive 无限空间这个博主不做介绍，请自行搜索。

# 准备环境

- 一台 vps 主机，Ubuntu、CentOS 都可以
- 内存 >= 512MB
- 硬盘最好 >= 10G
- KVM 架构

# 安装 AriaNg

Aria2是一个命令行下运行、多协议、多来源下载工具，支持磁力链接、BT种子、HTTP、FTP等下载协议，当然因为它是命令行下载工具，所以我们想下载一个东西还需要去敲命令自然是不方便，于是就有一些人根据Aria2的API开发了一些在线管理面板，可以直接在网页上面添加管理任务。

## 安装 aria2

先安装 aria2 服务端，使用逗比大佬的脚本

```shell
wget -N --no-check-certificate https://raw.githubusercontent.com/ToyoDAdoubi/doubi/master/aria2.sh && chmod +x aria2.sh && bash aria2.sh
```

运行脚本后会出现脚本操作菜单，选择并输入 `1` 就会开始安装。

```shell
Aria2 一键安装管理脚本 [vx.x.x]
-- Toyo | doub.io/shell-jc4 --
 
 0. 升级脚本
————————————
 1. 安装 Aria2
 2. 更新 Aria2
 3. 卸载 Aria2
————————————
 4. 启动 Aria2
 5. 停止 Aria2
 6. 重启 Aria2
————————————
 7. 修改 配置文件
 8. 查看 配置信息
 9. 查看 日志信息
10. 配置 自动更新 BT-Tracker服务器
————————————
 
当前状态: 已安装 并 已启动
 
请输入数字 [0-10]:
```

安装成功后配置文件在 `/root/.aria2/aria2.conf`，内容项：

```shell
dir=/usr/local/caddy/www/aria2/Download
rpc-listen-port=6800
rpc-secret=codesofun
```

**其他操作**

- 启动：/etc/init.d/aria2 start
- 停止：/etc/init.d/aria2 stop
- 重启：/etc/init.d/aria2 restart
- 查看状态：/etc/init.d/aria2 status
- 配置文件：/root/.aria2/aria2.conf （配置文件包含中文注释，但是一些系统可能不支持显示中文）
- 令牌密匙：随机生成（可以自己修改配置文件）
- 下载目录：/usr/local/caddy/www/aria2/Download

## 安装 AriaNg

AriaNg是一个前端(HTML+JS静态)控制面板，不需要和 Aria2(后端/服务端)放在一个服务器或者设备中，你甚至可以只在服务器上面搭建Aria2后端，然后访问别人建好的 AriaNg前端面板，也可以远程操作Aria2后端！

Github 源码地址：https://github.com/mayswind/AriaNg

不需要和 Aria2(后端/服务端)放在一个服务器或者设备中，你甚至可以只在服务器上面搭建 Aria2 后端，然后访问别人建好的 AriaNg 前端面板，也可以远程操作 Aria2 后端！

这里安装 Nginx Web 服务器，安装方式参考：

- []()
- []()

```shell
wget -N --no-check-certificate https://raw.githubusercontent.com/ToyoDAdoubi/doubi/master/caddy_install.sh && chmod +x caddy_install.sh && bash caddy_install.sh install http.filemanager
```

# 配置 Google Drive

## 挂载 Google Drive 到 VPS

安装 Rclone

```shell
yum -y install unzip
curl https://rclone.org/install.sh | sudo bash
```

配置

```shell

[root@vultr ~]# rclone config
2018/11/25 09:31:54 NOTICE: Config file "/root/.config/rclone/rclone.conf" not found - using defaults
No remotes found - make a new one
n) New remote
s) Set configuration password
q) Quit config
n/s/q> n
name> codesofun
Type of storage to configure.
Enter a string value. Press Enter for the default ("").
Choose a number from below, or type in your own value
 1 / A stackable unification remote, which can appear to merge the contents of several remotes
   \ "union"
 2 / Alias for a existing remote
   \ "alias"
 3 / Amazon Drive
   \ "amazon cloud drive"
 4 / Amazon S3 Compliant Storage Providers (AWS, Ceph, Dreamhost, IBM COS, Minio)
   \ "s3"
 5 / Backblaze B2
   \ "b2"
 6 / Box
   \ "box"
 7 / Cache a remote
   \ "cache"
 8 / Dropbox
   \ "dropbox"
 9 / Encrypt/Decrypt a remote
   \ "crypt"
10 / FTP Connection
   \ "ftp"
11 / Google Cloud Storage (this is not Google Drive)
   \ "google cloud storage"
12 / Google Drive
   \ "drive"
13 / Hubic
   \ "hubic"
14 / JottaCloud
   \ "jottacloud"
15 / Local Disk
   \ "local"
16 / Mega
   \ "mega"
17 / Microsoft Azure Blob Storage
   \ "azureblob"
18 / Microsoft OneDrive
   \ "onedrive"
19 / OpenDrive
   \ "opendrive"
20 / Openstack Swift (Rackspace Cloud Files, Memset Memstore, OVH)
   \ "swift"
21 / Pcloud
   \ "pcloud"
22 / QingCloud Object Storage
   \ "qingstor"
23 / SSH/SFTP Connection
   \ "sftp"
24 / Webdav
   \ "webdav"
25 / Yandex Disk
   \ "yandex"
26 / http Connection
   \ "http"
Storage> 12
** See help for drive backend at: https://rclone.org/drive/ **

Google Application Client Id
Leave blank normally.
Enter a string value. Press Enter for the default ("").
client_id>
Google Application Client Secret
Leave blank normally.
Enter a string value. Press Enter for the default ("").
client_secret>
Scope that rclone should use when requesting access from drive.
Enter a string value. Press Enter for the default ("").
Choose a number from below, or type in your own value
 1 / Full access all files, excluding Application Data Folder.
   \ "drive"
 2 / Read-only access to file metadata and file contents.
   \ "drive.readonly"
   / Access to files created by rclone only.
 3 | These are visible in the drive website.
   | File authorization is revoked when the user deauthorizes the app.
   \ "drive.file"
   / Allows read and write access to the Application Data folder.
 4 | This is not visible in the drive website.
   \ "drive.appfolder"
   / Allows read-only access to file metadata but
 5 | does not allow any access to read or download file content.
   \ "drive.metadata.readonly"
scope> 1
ID of the root folder
Leave blank normally.
Fill in to access "Computers" folders. (see docs).
Enter a string value. Press Enter for the default ("").
root_folder_id>
Service Account Credentials JSON file path
Leave blank normally.
Needed only if you want use SA instead of interactive login.
Enter a string value. Press Enter for the default ("").
service_account_file>
Edit advanced config? (y/n)
y) Yes
n) No
y/n> n
Remote config
Use auto config?
 * Say Y if not sure
 * Say N if you are working on a remote or headless machine or Y didn't work
y) Yes
n) No
y/n> n
If your browser doesn't open automatically go to the following link: https://accounts.google.com/o/oauth2/auth?access_type=offline&client_id=202264815644.apps.googleusercontent.com&redirect_uri=urn%3Aietf%3Awg%3Aoauth%3A2.0%3Aoob&response_type=code&scope=https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fdrive&state=0fbdd9de765facd73e06e605a3b00e23
Log in and authorize rclone for access
Enter verification code> 4/oACeTswSxMqE2wPveA0h9MAIHEBRDc2HtEhEVX_yBcr--iTAFJ7-fPg

Configure this as a team drive?
y) Yes
n) No
y/n> y
Fetching team drive list...
No team drives found in your account--------------------
[codesofun]
type = drive
scope = drive
token = {"access_token":"ya29.GltfBn_YA8e0lgJNKeMAHZ3b_IeIBXHHrJxMO7wfW0AsY6v_Nso8YczhZafVe8UIIgK6ft1dn6BqP-UWp-W2YXBtcf6zbLuIZgKcPqwnhsVAkx3f7QcO5m0EUvAv","token_type":"Bearer","refresh_token":"1/hFSVnEeJ8CkFmrpdd7IGNx69mhkj8Cpny6JUktv3xcY","expiry":"2018-11-25T10:42:09.400295503Z"}
--------------------
y) Yes this is OK
e) Edit this remote
d) Delete this remote
y/e/d> y
Current remotes:

Name                 Type
====                 ====
codesofun            drive

e) Edit existing remote
n) New remote
d) Delete remote
r) Rename remote
c) Copy remote
s) Set configuration password
q) Quit config
e/n/d/r/c/s/q> q
```

完成后挂载为磁盘

```shell
# 新建本地文件夹，路径自己定
mkdir -p /data/GoogleDrive

#挂载为磁盘
rclone mount codesofun:share /data/GoogleDrive --allow-other --allow-non-empty --vfs-cache-mode writes &

#格式为[name]:[google drive dir] [mount dir]
#[name]就是配置文件是输入的name，例如我的就是ct
#[google drive dir] 这个是谷歌云盘的目录，根目录的花直接空开就可以了
#[mount dir]就是本地挂载位置，/data/GoogleDrive
```

> Fatal error: failed to mount FUSE fs: fusermount: exec: "fusermount": executable file not found in $PATH
> 
> 没有安装 fuse ,自行安装就可以了 `yum -y install fuse`

挂载完成后看看效果

```shell
[root@vultr ~]# df -h
Filesystem       Size  Used Avail Use% Mounted on
/dev/vda1         20G  1.3G   18G   7% /
devtmpfs         234M     0  234M   0% /dev
tmpfs            244M     0  244M   0% /dev/shm
tmpfs            244M  8.5M  236M   4% /run
tmpfs            244M     0  244M   0% /sys/fs/cgroup
tmpfs             49M     0   49M   0% /run/user/0
codesofun:share   15G  3.8G   12G  26% /data/GoogleDrive
```

设置开机自动挂载谷歌云

```shell
wget http://ctnmb.com/dl/rcloned && vim rcloned
#然后修改文件内如下内容
NAME=""  #[name]
REMOTE=''  #[google drive dir]
LOCAL=''  #[mount dir]
```

```shell
mv rcloned /etc/init.d/rcloned
chmod +x /etc/init.d/rcloned
vim /etc/rc.d/rc.local #在末尾加入 bash /etc/init.d/rcloned start
chmod +x /etc/rc.d/rc.local
```

# 请开始你的表演

