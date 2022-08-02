---
title: Frp配合Nginx暴露内网服务
comments: true
tags:
  - Frp
  - Nginx
  - Acme
  - WinSW
categories:
  - tool
date: 2022-02-13 10:59:15
updated: 2022-02-13 10:59:15
---

Frp暴露内网服务、RDP和SSH等，配合Nginx

xxxxxxxxxx159 1import java.awt.*;2import java.awt.image.BufferedImage;3import java.util.Arrays;4import java.util.Random;5​6/**7 * @author nicenan8 */9public class VerificationCodeUtils {10    private int width = 100;11    private int height = 50;12    private int length = 4;13    private int interLine = 10;14    private float noiseRate = 0.1f;15    private String verificationCode;16​17    public int getInterLine() {18        return interLine;19    }20​21    public void setInterLine(int interLine) {22        this.interLine = interLine;23    }24​25    public float getNoiseRate() {26        return noiseRate;27    }28​29    public void setNoiseRate(float noiseRate) {30        this.noiseRate = noiseRate;31    }32​33    public int getWidth() {34        return width;35    }36​37    public void setWidth(int width) {38        this.width = width;39    }40​41    public int getHeight() {42        return height;43    }44​45    public void setHeight(int height) {46        this.height = height;47    }48​49    public int getLength() {50        return length;51    }52​53    public void setLength(int length) {54        this.length = length;55    }56​57    public String getVerificationCode() {58        return verificationCode;59    }60​61    public char[] getVerificationCodeArrary() {62        return verificationCodeArrary;63    }64​65    public void setVerificationCodeArrary(char[] verificationCodeArrary) {66        this.verificationCodeArrary = verificationCodeArrary;67    }68​69    private char[] verificationCodeArrary = {'0', '1', '2', '3', '4', '5', '6', '7', '8', '9',70            'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm', 'n', 'o', 'p', 'q', 'r', 's', 't', 'u', 'v', 'w', 'x', 'y', 'z',71            'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M', 'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z'72    };73​74    //饿汉单例模式75    private VerificationCodeUtils() {76    }77​78    private static VerificationCodeUtils verificationCodeUtils;79​80    public static VerificationCodeUtils getInstance() {81        if (verificationCodeUtils == null) {82            synchronized (VerificationCodeUtils.class) {83                if (verificationCodeUtils == null) {84                    verificationCodeUtils = new VerificationCodeUtils();85                    return verificationCodeUtils;86                }87            }88        }89        return verificationCodeUtils;90    }91​92    @Override93    public String toString() {94        return "VerificationCodeUtils{" +95                "width=" + width +96                ", height=" + height +97                ", length=" + length +98                ", interLine=" + interLine +99                ", noiseRate=" + noiseRate +100                ", verificationCode='" + verificationCode + '\'' +101                ", verificationCodeArrary=" + Arrays.toString(verificationCodeArrary) +102                '}';103    }104​105    public BufferedImage createVerificationCodeImage() {106        BufferedImage bufferedImage = new BufferedImage(width, height, BufferedImage.TYPE_INT_RGB);107        Graphics graphics = bufferedImage.getGraphics();108        Random random = new Random();109​110        graphics.setColor(getRandomColor());111        graphics.fillRect(0, 0, width, height);112        //设置乱七八糟横线113        if (interLine > 0) {114            for (int i = 0; i < interLine; i++) {115                int x1 = random.nextInt(width);116                int y1 = random.nextInt(height);117                int x2 = random.nextInt(width);118                int y2 = random.nextInt(height);119                graphics.setColor(getRandomColor());120                graphics.drawLine(x1, y1, x2, y2);121            }122​123        }124​125        //最终的验证码126        StringBuffer sb = new StringBuffer();127        //写入验证码128        int fontsize = (int) (height * 0.9);129        graphics.setFont(new Font(Font.SANS_SERIF, Font.BOLD, fontsize));130        //要画的字符的X坐标131        int codex = 0;132        int codey = 0;133        for (int i = 0; i < length; i++) {134            codey = (int) ((Math.random() * 0.3 + 0.7) * height);135            graphics.setColor(getRandomColor());136            int anInt = random.nextInt(verificationCodeArrary.length - 1);137            sb.append(verificationCodeArrary[anInt]);138            graphics.drawChars(verificationCodeArrary, anInt, 1, codex, codey);139            codex += (width / length) * (Math.random() * 0.3 + 0.9);140        }141​142        //插入噪点143        int noiseNum = (int) (width * height * noiseRate);144        for (int i = 0; i < noiseNum; i++) {145            int x = random.nextInt(width);146            int y = random.nextInt(height);147            graphics.setColor(getRandomColor());148            graphics.drawLine(x, y, x, y);149        }150        graphics.dispose();151        verificationCode = sb.toString();152        return bufferedImage;153    }154​155    private Color getRandomColor() {156        Random random = new Random();157        return new Color(random.nextInt(255), random.nextInt(255), random.nextInt(255));158    }159}java VerificationCodeUtils

## 资源

[frp官方仓库](https://github.com/fatedier/frp)

[frps脚本](https://github.com/MvsCode/frps-onekey)

## 配置远程桌面连接

### frps服务端配置

```ini
bind_addr = 0.0.0.0
bind_port = 7864
kcp_bind_port = 7864
dashboard_port = 18657
dashboard_user = admin
dashboard_pwd = xxxxxx
vhost_http_port = 9569
log_file = ./frps.log
# debug, info, warn, error
log_level = info
log_max_days = 3
token = xxxxx
max_pool_count = 50
tcp_mux = true
bind_udp_port = 9856
```

### frpc客户端配置

#### 被远程控制的主机1

frpc.ini

```ini
[common]
#frps服务器公网地址
server_addr = xxx.xxx.xxx.xxx
server_port = 7864
protocol = kcp
token = xxxxx

#另一方不需要使用frp，而是通过服务器转发tcp
[5600x_rdp_tcp]
type = tcp
local_ip = 127.0.0.1
local_port = 3389
remote_port = 6292

[5600x_rdp_udp]
type = udp
local_ip = 127.0.0.1
local_port = 3389
remote_port = 6292

#p2p配置，无法打洞则使用stcp
[5600x_rdp_xtcp]
type = xtcp
sk = xxx
local_ip = 127.0.0.1
local_port = 3389

[5600x_rdp_sudp]
type = sudp
sk = xxxx
local_ip = 127.0.0.1
local_port = 3389
```

##### 设置Frp为服务开机自启

[下载WinSW](https://github.com/winsw/winsw/releases)  

创建WinSW同名的xml文件放在一起

```xml
<service>
  <!-- 服务的唯一标识 -->
  <id>frpc</id>
  <name>frpc</name>
  <description>frp客户端</description>
  <executable>D:\Tool\frp\frpc.exe</executable>
  <arguments>-c D:\Tool\frp\frpc.ini</arguments>
  <startmode>Automatic</startmode>
  <!-- 第一次启动失败 120秒重启 -->
  <onfailure action="restart" delay="120 sec"/>
  <!-- 第二次启动失败 600秒后重启 -->
  <onfailure action="restart" delay="600 sec"/>
  <log mode="roll"></log>
  <logpath>D:\Tool\frp\logs</logpath>
</service>
```

安装frpc服务:`.\WinSW.exe install frpc.xml`

更新服务配置:`.\WinSW.exe refresh frpc.xml`



#### 发起控制的主机2

frpc.ini

```ini
[common]
#frps服务器公网地址
server_addr = xxx.xxx.xxx.xxx
server_port = 7864
protocol = kcp
token = xxxxx

#尝试p2p连接
[5600x_rdp_visitor]
type = xtcp
role = visitor
server_name = 5600x_rdp_xtcp
sk = xxx
bind_addr = 127.0.0.1
bind_port = 6000

[5600x_rdp_udp_visitor]
type = sudp
role = visitor
server_name = 5600x_rdp_sudp
sk = xxxx
bind_addr = 127.0.0.1
bind_port = 6000
```

#### 开始使用

先开启服务器的frps，在开启主机1的frpc

不想通过服务器转发需要在主机2开启frpc，通过 `127.0.0.1:6000` 尝试p2p连接主机1

主机2没有frp时可以通过 `服务器公网IP:6292`连接主机1

## 配合Nginx暴露内网服务

### Docker安装Nginx

[参考教程](https://www.exception.site/docker/docker-install-nginx)

```bash
#搞个临时复制出配置
#mkdir /root/nginx/nginx.conf
docker run --name tmp-nginx-container -p 80:80 -d nginx
#docker cp tmp-nginx-container:/etc/nginx/nginx.conf /root/nginx/nginx.conf
#docker cp tmp-nginx-container:/etc/nginx/conf.d/default.conf /root/nginx/conf.d/ default.conf
mkdir /root/nginx
docker cp tmp-nginx-container:/etc/nginx /root
docker rm -f tmp-nginx-container


#nginx.conf和conf.d/default.conf和证书文件提前放好

#运行
docker run -d \
-p 80:80 \
-p 443:443 \
--name nginx \
--restart always \
-v /root/nginx:/etc/nginx \
-v /root/nginx-logs:/var/log/nginx \
-v /root/nginx-html:/usr/share/nginx/html \
nginx:alpine 
```

### Frp配置

#### frps

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

#### frpc

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
token = xxxx

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

### Nginx配置

#### default.conf

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

### Acme自动ssl证书

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
--key-file       /root/nginx/ssl/nicenan.cn.key  \
--fullchain-file /root/nginx/ssl/nicenan.cn.cer \
--reloadcmd     "docker restart nginx" \
--ecc


### 三级域名版本

acme.sh --issue --dns dns_dp -d n1.nicenan.cn -d *.n1.nicenan.cn --keylength ec-256 --server letsencrypt


acme.sh --installcert -d n1.nicenan.cn \
--key-file       /root/nginx/ssl/n1.nicenan.cn.key  \
--fullchain-file /root/nginx/ssl/n1.nicenan.cn.cer \
--reloadcmd     "docker restart nginx" \
--ecc

# 开启通知Bark通知
export BARK_API_URL="https://api.day.app/XXXXXXXXXXXXXXXXXXXXXX"
export BARK_GROUP=ACME
acme.sh --set-notify --notify-hook bark
```

### 支持多个服务

#### Frp配置

frpc.ini

```ini
[common]
server_addr = xxx.xxx.xxx.xxx
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

#### Nginx配置

以下都是docker挂载到nginx容器的目录 /root/nginx

初始配置：  /root/nginx/nginx.conf

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

需要反代的一系列配置：/root/nginx/conf.d/n1.conf

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

证书配置(泛域名证书，配合acme自动更新):/root/nginx/conf.d/n1.ssl

```nginx
ssl_certificate  ssl/n1.nicenan.cn.cer; 
ssl_certificate_key ssl/n1.nicenan.cn.key; 
ssl_session_timeout 5m;
ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
ssl_protocols TLSv1.2 TLSv1.3;
ssl_prefer_server_ciphers on;
```

反向代理buffer配置:/root/nginx/conf.d/fxdl

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
