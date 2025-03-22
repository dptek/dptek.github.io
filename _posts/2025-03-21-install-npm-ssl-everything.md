---
layout: post
title: "让所有家庭内网的服务都用上 https SSL 证书，从外网都可以用域名访问内网服务，无需在路由器上开放端口，也不需要添加 DNS 记录！！！"
author: DPTEK
date: 2025-03-21 07:07:07 -0700
categories: 服务器
image: assets/images/2025-03-21/npm无需开端口改DNS记录让你的内网网站都用上小锁头https.png
---

## 目的
我们的目的是让我们内网的服务器都用上安全证书，并有 SSL 加密，不再看到难看的此网站不安全的提示，更重要的是我们在不添加内网穿透的情况下就可以从外网访问我们的内网服务。

## 前提
你需要一个虚拟机平台，我假设你已经准备了一个。我是用的是 Proxmox 8.3。其他平台一样可以实现。

## 安装 Nginx Proxy Manager（NPM）这个不是node.js 的那个 npm 包管理器

* 我们采用Alpine Linux 的虚拟机专用版 iso 镜像，它只有 63M。在它的基础上我们会安装docker 和 docker-compose，并使用 docker-compose 来安装 NPM。

* Alpine Linux 镜像的默认登录用户名：root 密码为空，直接回车便可登录。

* 使用 setup-alpine 照我的视频回答提示问题完成安装。

* 安装时可能需要手动配置静态 IP 地址, 请按如下样板按需更改：

```bash
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
        address 192.168.1.11
        netmask 255.255.255.0
        gateway 192.168.1.254
```

* 重启 Alpine Linux

* ssh 进入 Alpine Linux 更新系统



```bash
su
apk update 
apk upgrade
apk add nano
```
前往 /etc/apk/

编辑： nano repositories

打开 community 源 - 去掉第三行前面的井号并保存退出。

```bash
apk add docker docker-compose
rc-update add docker boot
service docker start
```
回到主目录

```bash
cd ~
touch docker-compose.yml
nano docker-compose.yml
```
黏贴以下内容

```yml
services:
  app:
    image: 'jc21/nginx-proxy-manager:latest'
    restart: unless-stopped
    ports:
      # These ports are in format <host-port>:<container-port>
      - '80:80' # Public HTTP Port
      - '443:443' # Public HTTPS Port
      - '81:81' # Admin Web Port
      # Add any other Stream port you want to expose
      # - '21:21' # FTP
    environment:
      # Mysql/Maria connection parameters:
      DB_MYSQL_HOST: "db"
      DB_MYSQL_PORT: 3306
      DB_MYSQL_USER: "npm"
      DB_MYSQL_PASSWORD: "npm"
      DB_MYSQL_NAME: "npm"
      # Uncomment this if IPv6 is not enabled on your host
      # DISABLE_IPV6: 'true'
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
    depends_on:
      - db

  db:
    image: 'jc21/mariadb-aria:latest'
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: 'npm'
      MYSQL_DATABASE: 'npm'
      MYSQL_USER: 'npm'
      MYSQL_PASSWORD: 'npm'
      MARIADB_AUTO_UPGRADE: '1'
    volumes:
      - ./mysql:/var/lib/mysql
```

```bash
docker compose up -d
```
## 访问并设置 NPM
使用你的ip地址访问NPM，端口号81
我的是：
https://192.168.1.11:81

```bash
Email:    admin@example.com
Password: changeme
```
