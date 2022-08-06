---
title: ASF挂卡软件
comments: true
tags:
  - Steam
  - ASF
categories:
  - 工具
date: 2022-02-13 19:35:40
updated:
---

Docker部署ASF挂卡

<!--more-->

## 安装

先新建目录`/home/archi/asf`

上传`IPC.config`到目录

```json
{
    "Kestrel": {
        "Endpoints": {
            "HTTP": {
                "Url": "http://*:1242"
            }
        }
    }
}
```

使用 [在线配置文件生成器](https://justarchinet.github.io/ASF-WebConfigGenerator)

选择ASF，填入SteamID64

打开高级设置设置，打开Headless并设置IPCPassword

下载配置文件(ASF.json)上传到目录

```bash
docker run -it -p 1242:1242 -v /home/archi/asf:/app/config --name asf --restart=always --pull always justarchi/archisteamfarm
```

最后浏览器打开 http://192.168.2.149:1242/

输入密码登陆后，新建一个Bot



## 公网访问

### frpc配置

```ini
[asf]
type = http
local_ip = 127.0.0.1
local_port = 1242
custom_domains = asf.n1.nicenan.cn
```

### nginx配置

```ngin
server {
    listen 443 ssl;
    #填写绑定证书的域名
    server_name  asf.n1.nicenan.cn; 
    #证书文件名称
    include /etc/nginx/conf.d/n1.ssl;
    
    location / {
    #反向代理地址
    proxy_pass http://asf.n1.nicenan.cn:9569;

    # 反向代理buffer
    include /etc/nginx/conf.d/fxdl;
    }
}
```



剩余frp和nginx配置见 [Referrence: frp](./frp-notes.md)

