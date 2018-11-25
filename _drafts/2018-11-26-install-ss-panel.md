---
layout: post
title:  做回机场主，搭建自己的 SSPanel
categories: ["vps"]
---

`ss-panel-v3-mod` 是一款专为 `shadowsocks` 设计的 web 前端面板，如果你想搭建属于自己的飞机场，赚大钱、迎娶白富美、走上人生巅峰，赶紧来试试吧！（假装是真的，大雾...）

今天我们使用的代码是：[ss-panel-v3-mod_Uim](https://github.com/NimaQu/ss-panel-v3-mod_Uim)

# 有啥新特性

- **支付系统集成**：集成 支付宝当面付 黛米付 易付通 码支付等多种支付系统
- UI ：修改为圆角、并自定义了几个图标的显示，节点列表等級0可见等級1节点但无法看见节点详情，增加了国家图标显示
- **商店**：商品增加同时连接设备数，用户限速属性
- 从肥羊那里抄来的：新用户注册现金奖励|高等级节点体验|设备数量限制
- **优化**：css 和 js等置入本地提升加载速度
- 增加 v2Ray 功能

演示站: [demo.nimaqu.com]() 账号和密码都是 admin 对接节点的 `mukey=NimaQu`

# 准备环境

- 一台 VPS 主机
- LNMP 环境
- ss-panel 源码

本次教程是基于 CentOS 系统，如果要看 Ubuntu 的可以参考 [这里](https://github.com/NimaQu/ss-panel-v3-mod_Uim/wiki/%E5%89%8D%E7%AB%AF%E5%AE%89%E8%A3%85) 的文档。

# 安装 LNMP

```shell
yum -y install wget git screen python
wget http://mirrors.linuxeye.com/lnmp-full.tar.gz
tar xzf lnmp-full.tar.gz
cd lnmp
screen -S lnmp
./install.sh
```

# 安装 sspanel 前端

修改 nginx 配置

```conf
root /data/wwwroot/example.com/public;

location / {
    try_files $uri $uri/ /index.php$is_args$args;
}
```

进入虚拟主机所在位置

```shell
cd /data/wwwroot/example.com
git clone -b master https://github.com/NimaQu/ss-panel-v3-mod_Uim.git tmp && mv tmp/.git . && rm -rf tmp && git reset --hard
chown -R root:root *
chmod -R 755 *
chown -R www:www storage
```

导入数据库，创建数据库 `ss` 导入 [glzjin_all.sql](https://github.com/NimaQu/ss-panel-v3-mod_Uim/blob/dev/sql/glzjin_all.sql)


```shell
30 22 * * * php /data/wwwroot/ss.2333.blog/xcat sendDiaryMail
0 0 * * * php -n /data/wwwroot/ss.2333.blog/xcat dailyjob
*/1 * * * * php /data/wwwroot/ss.2333.blog/xcat checkjob
*/1 * * * * php /data/wwwroot/ss.2333.blog/xcat syncnode
```

# 安装 sspanel 后端

```shell
yum -y install vim git python-devel libffi-devel openssl-devel python-setuptools && easy_install pip
```

安装libsodium

```shell
yum -y groupinstall "Development Tools"
wget https://github.com/jedisct1/libsodium/releases/download/1.0.10/libsodium-1.0.10.tar.gz
tar xf libsodium-1.0.10.tar.gz && cd libsodium-1.0.10
./configure && make -j2 && make install
echo /usr/local/lib > /etc/ld.so.conf.d/usr_local_lib.conf
ldconfig
```

安装后端

```shell
git clone -b manyuser https://github.com/glzjin/shadowsocks.git
cd shadowsocks
pip install -r requirements.txt
```

接着来设置程序的配置文件

```shell
cp apiconfig.py userapiconfig.py
cp config.json user-config.json
```

`vim userapiconfig.py`


```shell
NODE_ID = 3
API_INTERFACE = 'glzjinmod'

```

# 走你
