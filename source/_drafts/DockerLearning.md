---
title: DockerLearning
comments: true
tags:
  - Linux
  - Docker
categories:
  - Docker
date: 2021-08-11 17:13:16
updated:
---

Docker

<!--more-->

## 安装

### Nacos Server

```bash
docker run -itd --name nacos -e MODE=standalone -p 8848:8848 -d nacos/nacos-server:latest
```

MODE    cluster/standalone    cluster/standalone default cluster

http://192.168.2.211:8848/nacos

nacos nacos

### Nginx

[参考教程](https://www.exception.site/docker/docker-install-nginx)

```bash
#搞个临时复制出配置
docker run --name tmp-nginx-container -d nginx
docker cp tmp-nginx-container:/etc/nginx/nginx.conf /home/nginx/nginx.conf
docker rm -f tmp-nginx-container
cd /home
nano /home/nginx.conf

#nginx.conf和conf.d/default.conf和证书文件提前放好

#运行
docker run -d \
-p 80:80 \
-p 443:443 \
--name nginx \
--restart always \
-v /home/nginx/nginx.conf:/etc/nginx/nginx.conf \
-v /home/nginx/conf.d:/etc/nginx/conf.d \
-v /home/nginx/logs:/var/log/nginx \
-v /home/nginx/ssl:/etc/nginx/ssl \
nginx:alpine 
```

#### 配合frp 一个域名一个服务

##### frps

[一键脚本](https://github.com/MvsCode/frps-onekey)

```ini
# [common] is integral section
[common]
# A literal address or host name for IPv6 must be enclosed
# in square brackets, as in "[::1]:80", "[ipv6-host]:http" or "[ipv6-host%zone]:80"
bind_addr = 0.0.0.0
bind_port = 7864
# udp port used for kcp protocol, it can be same with 'bind_port'
# if not set, kcp is disabled in frps
kcp_bind_port = 7864
# if you want to configure or reload frps by dashboard, dashboard_port must be set
dashboard_port = 18657
# dashboard assets directory(only for debug mode)
dashboard_user = admin
dashboard_pwd = xxxxxx
# assets_dir = ./static
# nginx要反代这个端口
vhost_http_port = 88
#vhost_https_port = 443
# console or real logFile path like ./frps.log
log_file = ./frps.log
# debug, info, warn, error
log_level = info
log_max_days = 3
# auth token
token = xxxxxx
# It is convenient to use subdomain configure for http、https type when many people use one frps server together.
#subdomain_host = 101.35.4.132
# only allow frpc to bind ports you list, if you set nothing, there won't be any limit
#allow_ports = 1-65535
# pool_count in each proxy will change to max_pool_count if they exceed the maximum value
max_pool_count = 50
# if tcp stream multiplexing is used, default is true
tcp_mux = true
```

##### frpc

```bash
nano /etc/systemd/system/frpc.service

####复制进去
[Unit]
Description=Frp Client Service
After=network.target

[Service]
Type=simple
User=root
Restart=on-failure
RestartSec=5s
ExecStart=/root/frp/frpc -c /root/frp/frpc.ini
####

[Install]
WantedBy=multi-user.target

systemctl enable frpc
systemctl start frpc
systemctl status frpc
```

```ini
[common]
server_addr = frp.nicenan.cn
server_port = 7864
token = pUr1D53V6Oy7Lita

[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 4528

[DOCKER_PORTAINER]
type = http
local_ip = 192.168.2.149
local_port = 9000
custom_domains = qinglong.nicenan.cn
```

##### default.conf

```nginx
server {
    listen 443 ssl;
    #填写绑定证书的域名
    server_name  qinglong.nicenan.cn; 
    #证书文件名称
    ssl_certificate  ssl/qinglong.nicenan.cn.crt; 
    #私钥文件名称
    ssl_certificate_key ssl/qinglong.nicenan.cn.key; 
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;

    location / {
    # 需要代理的容器ip或容器别名，我设置的容器别名是filebrowser，也可以直接这个容器的ip：172.18.0.2
    # http端口默认是80，和filebrowser映射端口相同，所以没加端口，如果filebrowser映射的是其它端口，如8089，那就写：http://filebrowser:8089
    proxy_pass http://qinglong.nicenan.cn:88;

    # 反向代理buffer
    proxy_buffering off;
    proxy_buffer_size 128k;
    proxy_buffers 100 128k;

    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection keep-alive;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_cache_bypass $http_upgrade;
    }
}
server {
    listen 80;
    #填写绑定证书的域名
    server_name qinglong.nicenan.cn; 
    #把http的域名请求转成https
    return 301 https://$host$request_uri; 
}
```
### acme
```bash
curl https://get.acme.sh | sh
#然后重启终端
#进入dnspod设置api密钥，记录id和token(key)
#使用dns验证方式签发证书 ，设置环境变量
export DP_Id="291441"
export DP_Key="7c915c94eb53b0f7cc70425e78aef3fe"
# --keylength ec-256  签发 ECC 证书
acme.sh --issue --dns dns_dp -d nicenan.cn -d *.nicenan.cn --keylength ec-256 --server letsencrypt

#开启自动复制到nginx 并重启nginx容器

acme.sh --installcert -d nicenan.cn \
--key-file       /home/nginx/ssl/nicenan.cn.key  \
--fullchain-file /home/nginx/ssl/nicenan.cn.cer \
--reloadcmd     "docker restart nginx" \
--ecc


### 三级域名版本

acme.sh --issue --dns dns_dp -d n1.nicenan.cn -d *.n1.nicenan.cn --keylength ec-256 --server letsencrypt


acme.sh --installcert -d n1.nicenan.cn \
--key-file       /home/nginx/ssl/n1.nicenan.cn.key  \
--fullchain-file /home/nginx/ssl/n1.nicenan.cn.cer \
--reloadcmd     "docker restart nginx" \
--ecc

```


#### 多个服务frp配合nginx ssl
##### frp配置

frpc.ini
```ini 
[common]
server_addr = frp.nicenan.cn
server_port = 7864
protocol = kcp
token = xxx


[docker_portainer]
type = http
local_ip = 127.0.0.1
local_port = 9000
custom_domains = docker.n1.nicenan.cn

[netdata]
type = http
local_ip = 127.0.0.1
local_port = 19999 
custom_domains = netdata.n1.nicenan.cn
```

frps.ini
```ini
[common]
bind_addr = 0.0.0.0
bind_port = 7864
kcp_bind_port = 7864
dashboard_port = 18657
dashboard_user = admin
dashboard_pwd = xxx
vhost_http_port = 9569
log_file = ./frps.log
log_level = info
log_max_days = 3
token = xxx
max_pool_count = 50

```

##### nginx配置

以下都是docker挂载到nginx容器的目录 /home/nginx

初始配置：  /home/nginx/nginx.conf
```nginx

user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}

```


需要反代的一系列配置：/home/nginx/conf.d/n1.conf

```nginx
server {
    listen 443 ssl;
    #填写绑定证书的域名
    server_name  docker.n1.nicenan.cn; 
    #证书文件配置
    include /etc/nginx/conf.d/n1.ssl;

    location / {
    proxy_pass http://docker.n1.nicenan.cn:9569;

    # 反向代理buffer配置
    include /etc/nginx/conf.d/fxdl;
    }
}
server {
    listen 443 ssl;
    #填写绑定证书的域名
    server_name  netdata.n1.nicenan.cn; 
    #证书文件配置
    include /etc/nginx/conf.d/n1.ssl;

    location / {
    proxy_pass http://netdata.n1.nicenan.cn:9569;

    # 反向代理buffer配置
    include /etc/nginx/conf.d/fxdl;
    }
}
server {
    listen 80;
    #填写绑定证书的域名
    server_name *.n1.nicenan.cn; 
    #把http的域名请求转成https
    return 301 https://$host$request_uri; 
}
```

证书配置(泛域名证书，配合acme自动更新):/home/nginx/conf.d/n1.ssl
```nginx
ssl_certificate  ssl/n1.nicenan.cn.cer; 
ssl_certificate_key ssl/n1.nicenan.cn.key; 
ssl_session_timeout 5m;
ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
ssl_protocols TLSv1.2 TLSv1.3;
ssl_prefer_server_ciphers on;
```

反向代理buffer配置:/home/nginx/conf.d/fxdl
```nginx
proxy_buffering off;
proxy_buffer_size 128k;
proxy_buffers 100 128k;
proxy_http_version 1.1;
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection keep-alive;
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
proxy_cache_bypass $http_upgrade;
```

每次只需增加一条frpc的配置，并在nginx设置反代即可