---
layout: post
title: 架设属于你的「萌娘百科」维基站点
categories: ["vps"]
tags: ["wiki"]
---

# 准备环境

- 内存 >= 512MB
- 硬盘 >= 5GB

```shell
sudo apt-get -y update
```

# 安装 nginx

```shell
sudo apt-get -y install nginx
```

# 安装 PHP

```shell
sudo apt-get -y install php7.0-fpm
```

# 安装 DokuWiki

```shell
wget https://download.dokuwiki.org/src/dokuwiki/dokuwiki-stable.tgz
tar -xzvf dokuwiki-stable.tgz
sudo mkdir /var/www/dokuiki
sudo mv dokuwiki-2018-04-22a/* /var/www/dokuiki
```

给目录授权

```shell
cd /var/www/dokuwiki
sudo chmod -R 775 data/
sudo chown -R www-data:www-data data/
```

## 修改 nginx 配置

```shell
vim /etc/nginx/sites-available/wiki.conf
```

```shell
server {
    server_name <IP_OR_DOMAIN>;

    # Maximum file upload size is 4MB - change accordingly if needed
    client_max_body_size 4M;
    client_body_buffer_size 128k;

    root /var/www/dokuwiki;
    index doku.php;

    #Remember to comment the below out when you're installing, and uncomment it when done.
    location ~ /(data/|conf/|bin/|inc/|install.php) { deny all; }
 
    location ~ ^/lib.*\.(js|css|gif|png|ico|jpg|jpeg)$ {
        expires 365d;
    }
 
    location / { try_files $uri $uri/ @dokuwiki; }
 
    location @dokuwiki {
        # rewrites "doku.php/" out of the URLs if you set the userwrite setting to .htaccess in dokuwiki config page
        rewrite ^/_media/(.*) /lib/exe/fetch.php?media=$1 last;
        rewrite ^/_detail/(.*) /lib/exe/detail.php?media=$1 last;
        rewrite ^/_export/([^/]+)/(.*) /doku.php?do=export_$1&id=$2 last;
        rewrite ^/(.*) /doku.php?id=$1&$args last;
    }
    
    location ~ \.php$ {
        try_files $uri $uri/ /doku.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param REDIRECT_STATUS 200;
        fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
    }

}
```

```shell
sudo ln -s /etc/nginx/sites-available/wiki.conf /etc/nginx/sites-enabled/
sudo systemctl reload nginx
```

# 设置 Wiki

# 视频版

