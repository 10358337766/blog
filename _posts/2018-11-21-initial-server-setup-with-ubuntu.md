---
layout: post
title:  安装 Ubuntu 后要做的设置
categories: ["linux"]
tags: ["ubuntu"]
---

我们第一次安装好 Ubuntu 服务器后应该做一些基本的设置，保证服务器的安全性和可用性。

# 第一步 - Root 登录

要登录服务器，需要知道服务器的外网 IP，还有登录密码。或者你使用 SSH 密钥进行身份验证，则需要 `root` 用户的私钥。

如果你没有登录到服务器，使用 `root` 用户登录到服务器，参考以下命令：

```shell
ssh root@your_server_ip
```

登录过程中可能会弹出主机真实性的警告信息，然后输入 root 密码。如果是第一次登录，系统还会提示你修改密码。

## 关于 Root

`root` 用户是 Linux 系统中的超级用户，它可是什么权限都有的。正因为它权限比较高，所以我们不推荐你长期的直接使用它，因为密码万一泄露服务器就 GG 了。所以我们需要设置一个替代的账户，使用它来进行日常操作。

# 第二步 - 创建一个新用户

当你使用 root 登录后，我们就可以添加一个新账户了。

这个例子中我会创建一个名为 `“codesofun”` 的新用户，你可以使用你喜欢的用户名：

```shell
adduser codesofun
```

![](/assets/images/2018/11/ubuntu_adduser.png)

添加账户的时候系统会询问你一些信息，设置一个比较复杂的密码，其他信息可以一路回车跳过。

# 第三步 - Root 权限

现在我们拥有一个普通用户 `codesofun`，有时候我们希望用它可以执行一些管理任务。

为了避免退出普通用户后使用 root 身份重新登录，我们可以为普通帐户设置所谓的 “超级用户” 或 `root` 权限。这将允许普通用户通过在每个命令前添加 `sudo` 来运行带有管理权限的命令。

要将这些权限添加到新用户，我们需要将新用户添加到 `sudo` 组里面。默认情况下，在 `Ubuntu 16.04` 上，允许属于 `sudo` 组的用户使用该 `sudo` 命令。

在 root 登录回话下，执行该命令将新用户添加到 `sudo` 组：

```shell
usermod -aG sudo codesofun
```

现在 `codesofun` 这个用户就可以使用超级用户权限来运行命令了！切换到 `codesofun` 使用 `su - codesofun` 即可。

想要提高服务器的安全性，请执行本教程中的其他步骤。

# 第四步 - 添加公钥认证（推荐）

保护服务器的下一步是为新用户设置公钥身份验证。设置后将使用 SSH 密钥登录来提高服务器的安全性。

## 生成密钥对

如果你本地还没有 SSH 密钥对（包含公钥和私钥），则需要生成一个密钥对。如果已经有要使用的密钥，请跳至 _复制公钥_ 步骤。

要生成新密钥对，请在本地终端输入以下命令：

```shell
ssh-keygen
```

假设你本地用户名为 “biezhi”，你将看到如下所示的输出：

```shell
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/biezhi/.ssh/id_rsa):
```

回车将配置保存在该路径中。接下来，系统会提示你输入密码来保护密钥，你可以输入密码也可以将它留空。

> 注意：如果将密码留空，在使用私钥进行身份验证的时候不用输入密码。如果输入密码，则需要输入私钥密码才能登录。使用密码保护密钥更安全，但这两种方法都有其用途，并且都比基本的密码方式验证要安全。

这会在 `biezhi` 的家目录生成私钥 `id_rsa` 和公钥 `id_rsa.pub`。需要切记的是，不要和任何人共享你的私钥！

## 复制公钥

生成 SSH 密钥对后，需要将公钥复制到新服务器。我们将介绍两种简单的方法。

**使用 ssh-copy-id**

如果本机安装了 `ssh-copy-id` 脚本，则可以使用它将公钥安装到你具有登录凭证的任何用户。

使用 `ssh-copy-id` 要指定服务器的用户和 IP，如下所示：

```shell
ssh-copy-id codesofun@your_server_ip
```

运行后会提示你输入密码，验证成功后你本地的公钥将被添加到远程用户（在这里是 `codesofun`）的 `~/.ssh/authorized_keys` 文件中。现在就可以使用私钥登录到服务器了！

**手动安装密钥**

假设你使用上一步生成了 SSH 密钥对，请在本地终端上使用以下命令来查看你的公钥（`id_rsa.pub`）：

```shell
cat ~/.ssh/id_rsa.pub
```

它的输出应该类似于以下内容：

```shell
id_rsa.pub contents
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDBGTO0tsVejssuaYR5R3Y/i73SppJAhme1dH7W2c47d4gOqB4izP0+fRLfvbz/tnXFz4iOP/H6eCV05hqUhF+KYRxt9Y8tVMrpDZR2l75o6+xSbUOMu6xN+uVF0T9XzKcxmzTmnV7Na5up3QM3DoSRYX/EP3utr2+zAqpJIfKPLdA74w7g56oYWI9blpnpzxkEd3edVJOivUkpZ4JoenWManvIaSdMTJXMy3MtlQhva+j9CgguyVbUkdzK9KKEuah+pFZvaugtebsU+bllPTB0nlXGIJk98Ie9ZtxuY3nCKneB+KjKiXrAvXUPCI9mWkYS/1rggpFmu3HbXBnWSUdf biezhi@machine.local
```

将公钥内容复制。

要使用 SSH 密钥作为新远程用户进行身份验证，必须将公钥添加到用户家目录中的特殊文件中。

在服务器上，以 root 用户身份输入以下命令以临时切换到新用户（替换自己的用户名）：

```shell
su - codesofun
```

现在，你将进入新用户的家目录。

创建一个名为 `.ssh` 的新目录，并使用以下命令限制其权限：

```shell
mkdir ~/.ssh
chmod 700 ~/.ssh
```

现在打开一个使用文本编辑器来修改 `~/.ssh/authorized_keys` 文件。我使用 `vi` 来编辑文件：

```shell
vi ~/.ssh/authorized_keys
```

现在把本地复制的公钥粘贴进去。

按下 `ESC` 键后输入 `:wq` 保存修改并退出。

现在需要给 `authorized_keys` 赋予权限：

```shell
chmod 600 ~/.ssh/authorized_keys
```

`exit` 退出到 root 用户。现在已安装公钥，你可以使用 SSH 密钥进行登录了。

接下来，我们来通过禁止密码登录来提高服务器的安全性。

# 第五步 - 禁止密码登录（推荐）

现在你的新用户可以使用 SSH 密钥登录，可以通过禁止使用密码验证来提高服务器的安全性。这样做之后服务器仅允许使用公钥验证。也就是说，登录到服务器（除了控制台）的唯一方法是拥有与已安装的公钥配对的私钥。

> 注意：如果你按照上面的建议为用户安装了公钥，则仅禁用密码验证。如果 SSH 秘钥方式登录不成功不要尝试这么干，会锁定你的服务器！

在服务器上禁止密码验证，需要如下操作。

以 root 身份或新的用户身份（添加 sudo 前缀）打开 SSH 程序配置：

```shell
sudo vi /etc/ssh/sshd_config
```

找到 `PasswordAuthentication` 删除前面的 `#` 取消注释，然后将值修改为 `no`。修改后应该是如下的样子：

```shell
PasswordAuthentication no
```

以下是另外两个对于仅密钥身份验证很重要的设置。如果你之前秘钥修改过该文件，则不要修改这些设置：

```shell
PubkeyAuthentication yes
ChallengeResponseAuthentication no
```

完成修改后，重新加载 SSH 守护程序：

```shell
sudo systemctl reload sshd
```

现在密码验证已经禁用，服务器只能通过 SSH 密钥身份验证登录。

# 第六步 - 测试登录

现在，在退出服务器之前，先来测试一下新配置。在确认可以通过 SSH 成功​​登录之前，请不要断开连接。

在本地新开一个终端，使用我们创建的新帐户登录到服务器。

```shell
ssh codesofun@your_server_ip
```

如果你向用户添加了公钥身份验证，你的私钥将用作身份验证凭据。否则，系统将提示你输入用户密码。

> 关于密钥身份验证的注意事项：如果创建秘钥时候输入了密码，在使用时系统会提示你输入秘钥的密码。否则，你会直接登录到服务器。

一旦向服务器提供身份验证，你将以新用户身份登录。

需要记住，如果要运行具有 root 权限的命令，请在运行命令前加入 “sudo”：

```shell
sudo command_to_run
```

# 第七步 - 设置防火墙

`Ubuntu 16.04` 服务器可以使用 `UFW` 防火墙来确保只允许连接到某些服务。我们可以使用它来简单的配置基本防火墙。

不同的程序可以在安装时使用 `UFW` 注册其配置文件。这些配置文件允许 `UFW` 按名称管理这些程序。`OpenSSH` 是允许我们现在连接到我们服务器的服务，它在 `UFW` 上注册了一个配置文件。

你可以输入以下内容来看到：

```shell
sudo ufw app list

Available applications:
  OpenSSH
```
我们需要确保防火墙允许 SSH 连接，以便我们可以在下次重新登录。我们可以输入以下内容来允许这些连接

```shell
sudo ufw allow OpenSSH
```

之后，我们可以输入以下命令启用防火墙：

```shell
sudo ufw enable
```

输入 “y” 并按回车键继续。你可以通过输入以下命令来查看 SSH 连接：

```shell
sudo ufw status

Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)
```

如果你要安装其他的服务，就需要调整防火墙的配置运行端口暴露接收流量。