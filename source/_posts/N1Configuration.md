---
title: N1配置
comments: true
tags:
  - N1
categories:
  - Device
date: 2020-05-13 09:35:44
updated: 2021-03-19 13:55:12
---

N1配置

<!--more-->

# Armbian

## 刷入系统

刷入5.9.16-flippy-51+
插网线
设置语言，编码，更新源

## docker安装

运行/root目录下的安装docker脚本  
更改docker国内镜像

``` bash
mkdir -p /etc/docker
tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://2wfkcpg0.mirror.aliyuncs.com","https://dockerhub.azk8s.cn","https://reg-mirror.qiniu.com"]
}
EOF
systemctl daemon-reload
systemctl restart docker
```

### 安装dockerui

``` bash
docker volume create portainer_data
#docker pull portainer/portainer-ce
docker run -d -p 8000:8000 -p 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce
```

### 安装mysql8

``` bash
docker pull mysql/mysql-server
docker run --name=mysql8 --restart=always -p 3306:3306 -d mysql/mysql-server
#使用以下命令监视容器的输出：
docker logs mysql8
#初始化完成后，命令的输出将包含为root用户生成的随机密码。使用以下命令检查密码(若没出现密码再输一次)：
docker logs mysql8 2>&1 | grep GENERATED
#从容器内连接到MySQL Server
docker exec -it mysql8 mysql -uroot -p
#第一次登录要修改root密码
ALTER USER 'root'@'localhost' IDENTIFIED BY 'passowrd';
#root远程登录
update user set host='%' where user='root';
flush privileges;
#退出bash
exit
```

### 安装redis

``` bash
docker pull redis:latest
docker run -itd --name redis --restart=always -p 6379:6379 redis:latest
#清空数据
docker exec -it redis FLUSHALL
#进入终端
docker exec -it redis /bin/bash
```

### 安装qiandao

``` bash
docker pull asdaragon/qiandao
docker run -d --name qiandao --restart=always -p 12345:80 -v $(pwd)/qiandao/config:/usr/src/app/config   asdaragon/qiandao
```


### 安装subconverter

``` bash
docker pull tindy2013/subconverter
docker run -d --name subconverter --restart=always -p 25500:25500 tindy2013/subconverter
```


### 安装netdata

``` bash
docker pull netdata/netdata
docker run -d --name=netdata \
  -p 19999:19999 \
  -v netdataconfig:/etc/netdata \
  -v netdatalib:/var/lib/netdata \
  -v netdatacache:/var/cache/netdata \
  -v /etc/passwd:/host/etc/passwd:ro \
  -v /etc/group:/host/etc/group:ro \
  -v /proc:/host/proc:ro \
  -v /sys:/host/sys:ro \
  -v /etc/os-release:/host/etc/os-release:ro \
  --restart unless-stopped \
  --cap-add SYS_PTRACE \
  --security-opt apparmor=unconfined \
  netdata/netdata
```

## OMV安装

科学上网或设置 raw github host;  
参考这个安装omv   
https://github.com/OpenMediaVault-Plugin-Developers/docs/blob/master/Adden-A-Installing_OMV5_on_Armbian.pdf  
运行ovm什么start ,初始化网卡，插网线  

``` bash
#由于network被omv删除了，每次重启都很慢，所以要干掉校验的一个service。
vim /lib/systemd/system/systemd-networkd-wait-online.service
# 找到ExecStart那行，将其替换为ExecStart=echo '1'，然后 :wq! 退出编辑文本模式
systemctl daemon-reload
```

可以在omv的网络里面设置静态ip和dns,导入ssl证书，开启https  
可以安装Cockpit,直接在omv-Extras里面装

## 组RAID

https://yaozhijin.gitee.io/linux%E4%B8%8B%E4%BD%BF%E7%94%A8mdadm%E7%BB%84%E8%BD%AFraid.html
https://www.bilibili.com/video/BV1P4411D7K7
https://blog.csdn.net/weixin_43515220/article/details/102874410

``` bash
sudo mdadm -C /dev/md0 -a yes -l 1 -n 2 /dev/sda1 /dev/sdb1
# 格式化
sudo mkfs.ext4 /dev/md0
# 写入配置文件
sudo mdadm -Ds /dev/md0 > mdadm.conf
sudo mv mdadm.conf /etc/
#挂载 也可在OMV里面挂载
mkdir /raid1
mount /dev/md0 /raid1
#永久挂载
vim /etc/fstab
UUID=77d1b97b-0681-49e7-99d6-3f51fde77308 /raid1 ext4 deafults 0 2
```

## kodbox有道云

https://hub.docker.com/r/kodcloud/kodbox/tags?page=1&ordering=last_updated
http://doc.kodcloud.com/v2/#/start

拉取镜像 `docker pull kodcloud/kodbox:v1.15`

创建mysql用户

``` mysql
CREATE DATABASE kodbox;  
CREATE USER 'kodbox'@'%' IDENTIFIED BY 'password';
GRANT ALL ON kodbox.* TO 'kodbox'@'%';
FLUSH PRIVILEGES; 
```

启动docker

`docker run -d -p 8666:80 --name=kodbox --restart=always -v /srv/dev-disk-by-uuid-77d1b97b-0681-49e7-99d6-3f51fde77308/kodbox:/var/www/html kodcloud/kodbox:v1.15`

mysql和redis的ip填写docker里分别对应的ip  
若要配置https: 下载腾讯云证书，解压nginx的证书,crt改名fullchain.pem,key改名privkey.pem

## docker-openwrt

https://www.right.com.cn/forum/forum.php?mod=viewthread&tid=1905062&extra=page%3D6%26filter%3Dtypeid%26typeid%3D21

上传docker镜像

``` bash
docker import /tmp/openwrt-armvirt-64-default-rootfs.tar.gz openwrt:acc
docker network create -d macvlan --subnet=192.168.2.0/24 --gateway=192.168.2.1 -o parent=eth0 macnet
#查看已创建了哪些
docker network ls 

docker run --restart always -d --network macnet --privileged --name openwrt openwrt:acc /sbin/init

#f大版本
docker pull unifreq/openwrt-aarch64:latest
docker run --restart always -d --network macnet --privileged --name openwrt unifreq/openwrt-aarch64:latest /sbin/init

#进入openwrt容器修改network设置
docker exec -it openwrt /bin/bash
vi /etc/config/network
#    option ipaddr '192.168.2.11'
#    option gateway '192.168.2.1'
#    option dns '192.168.2.1'
reboot
exit
```

访问 192.168.2.11，禁止dhcp，ipv6，取消桥接？



## nginx配置

接管OMV的nginx生成的配置

注释`/etc/nginx/nginx.conf` 里面的 ` include /etc/nginx/sites-enabled/*;`  
拷贝  `/etc/nginx/sites-enabled/openmediavault-webgui`  到  `/etc/nginx/conf.d/openmediavault-webgui.conf`

cockpit需要添加一条按这个配置的location 

https://github.com/cockpit-project/cockpit/wiki/Proxying-Cockpit-over-nginx  
可以配置kodbox为另一个域名访问  

完整conf如下:  

``` nginx
server {
    server_name nas.nicenan.cn;
    listen [::]:80;
      if ($scheme = http) {
          # Force redirection to HTTPS.
          return 301 https://$host:443$request_uri;
      }
    listen [::]:443 ssl;
    ssl_certificate /etc/nginx/certificate/1_nas.nicenan.cn_bundle.crt;
    ssl_certificate_key /etc/nginx/certificate/2_nas.nicenan.cn.key;
    error_log /var/log/nginx/kodbox_error.log error;
    access_log /var/log/nginx/kodbox_access.log combined;
    client_max_body_size 500M;
    client_header_timeout 3600s;
    client_body_timeout 3600s;
    fastcgi_connect_timeout 3600s;
    fastcgi_send_timeout 3600s;
    fastcgi_read_timeout 3600s;
    location / {
        # Required to proxy the connection to Cockpit
        proxy_pass http://127.0.0.1:8666;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Required for web sockets to function
        proxy_http_version 1.1;
        proxy_buffering off;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        # Pass ETag header from Cockpit to clients.
        # See: https://github.com/cockpit-project/cockpit/issues/5239
        gzip off;
    }
}

server {
    server_name n1.nicenan.cn;
    root /var/www/openmediavault;
    index index.php;
    autoindex off;
    server_tokens off;
    sendfile on;
    large_client_header_buffers 4 32k;
    client_max_body_size 25M;
    error_log /var/log/nginx/openmediavault-webgui_error.log error;
    access_log /var/log/nginx/openmediavault-webgui_access.log combined;
    error_page 404 = /404.php;
    location /cockpit/ {
        # Required to proxy the connection to Cockpit
        proxy_pass https://127.0.0.1:9090/cockpit/;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Required for web sockets to function
        proxy_http_version 1.1;
        proxy_buffering off;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        # Pass ETag header from Cockpit to clients.
        # See: https://github.com/cockpit-project/cockpit/issues/5239
        gzip off;
    }
    location /404.html {
        internal;
    }
    location /extjs6/ {
        alias /usr/share/javascript/extjs6/;
        expires 2d;
    }
    location ~ ^/(css|js|images)/ {
        expires 2d;
    }
    location /favicon {
        expires 14d;
    }
    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/run/php/php7.3-fpm-openmediavault-webgui.sock;
        fastcgi_index index.php;
        fastcgi_read_timeout 60s;
        include fastcgi.conf;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PHP_ADMIN_VALUE session.cookie_secure=On;
    }
    listen [::]:80 default_server ipv6only=on;
      if ($scheme = http) {
          # Force redirection to HTTPS.
          return 301 https://$host:443$request_uri;
      }
    listen [::]:443 default_server ipv6only=on ssl deferred;
    ssl_certificate /etc/ssl/certs/openmediavault-70b9aedb-7a5f-4286-b53f-bf0bbc500cc8.crt;
    ssl_certificate_key /etc/ssl/private/openmediavault-70b9aedb-7a5f-4286-b53f-bf0bbc500cc8.key;
    include /etc/nginx/openmediavault-webgui.d/*.conf;
}
```

# Openwrt

https://www.right.com.cn/forum/thread-2983767-1-1.html

https://www.right.com.cn/forum/thread-4046582-1-1.html

网络-接口-全局网络选项-IPv6 ULA 前缀 这个直接填写小米路由器里面的LAN IPv6地址

用ShadowSocksR Plus+，绕过国内 ？ 

password 默认 添加 `upos-hz-mirrorakam.akamaized.net` 和 `bilibili.com` 到直连

实测发现 5.9.8-flippy-49+ 版本 配AX6，走的是AX6的DNS，n1上接口和ssr和Turbo acc加速这三个地方设置dns都没有用，电脑必须设置8.8.8.8才能科学上网。

解决办法是吧AX6宽带拨号的地方dns1设为8.8.8.8，n1的dhcp/dns的dns转发 192.168.2.1 (AX6的ip,没测试要不要设置这个)，电脑dns可以设置为192.168.2.11 （n1 ip）

看自带文档启用docker ce?

# Openwrt Openclash 

接口删掉只剩 lan，全剧网络ipv6前缀删掉

地址192.168.2.11，子网掩码255.255.255.0，网关192.168.2.1，dns自定+主路由ip，内置ipv6管理取消，桥接不要关。

dhcp忽略，ipv6都关掉。不用加防火墙代码

openclash 运行模式 Fake-IP(TUN-混合)，网络栈类型Gvisor，内核编译版本armv8，劫持和缓存dns启用。

最后添加配置文件，开整。

需要富强的客户端网关改为192.168.2.11，dns 192.168.2.11

没提到的都不用管

# 旧

./install-aml
apt-get update



apt install ipset tcpdump net-tools git tcptraceroute iftop -y
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh --mirror Aliyun


mkdir -p /etc/docker
tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://2wfkcpg0.mirror.aliyuncs.com","https://dockerhub.azk8s.cn","https://reg-mirror.qiniu.com"]
}
EOF
systemctl daemon-reload
systemctl restart docker

docker volume create portainer_data
docker pull portainer/portainer:latest
上传public
docker run -d -p 9000:9000 --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data -v /public:/public --name portainer portainer/portainer:latest 
docker run -d -p 9000:9000 --name portainer --restart always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer


docker pull unifreq/openwrt-aarch64:r20.02.15

ip link set eth0 promisc on

docker network create -d macvlan --subnet=192.168.2.0/24 --gateway=192.168.2.1 -o parent=eth0 macnet
docker network ls查看已创建了哪些
docker run --restart always -d --network macnet --privileged --name openwrt unifreq/openwrt-aarch64:latest /sbin/init

进入openwrt容器修改network设置
vi /etc/config/network
option ipaddr 192.168.2.11
reboot






dns
218.104.128.106
58.22.96.66

6.1.openwrt旁路由：关闭dhcp，网关填主路由ip，dns可以为主路由ip或114等公共dns，且将其【物理设置】中的【桥接接口】取消掉。还有最重要的1点：务必在防火墙添加规则设置：
iptables -t nat -I POSTROUTING -j MASQUERADE 并重启防火墙
iptables -t nat -I POSTROUTING -o eth0 -j MASQUERADE

6.2.主路由：开启dhcp，网关和dns填旁路由ip。
DHCP 选项
3,192.168.2.11
6,192.168.2.11
https://www.right.com.cn/forum/thread-1347921-1-1.html

auto eth0
iface eth0 inet manual
up ip link set eth0 promisc on
auto macvlan
iface macvlan inet static
   address 192.168.12.111
   netmask 255.255.255.0
   gateway 192.168.12.111
   dns-nameservers 192.168.12.111
   pre-up ip link add macvlan link eth0 type macvlan mode bridge
   post-down ip link del macvlan link eth0 type macvlan mode bridge

cp /etc/network/interfaces /root/interfaces.bak
2. 宿主机无法上网的问题，总的思路是将macvlan与eth0桥接，大概是这个意思；
具体，修改/etc/network/interfaces, 修改成如下，然后重启就可以了。修改过程中有报错，可以忽略。这个方法参考Raymond博客  https://raymondtech.win/2019/08/%E5%9C%A8docker%E4%B8%AD%E8%BF%90%E8%A1%8Copenwrt/
一次性方案，重启失效，粘贴到ssh里面执行，注意换行符号：
# 添加macvlan桥接网卡，并将192.168.2.8分配到该网卡

ip addr del 192.168.2.111/24 dev eth0; \
ip link add macvlan link eth0 type macvlan mode bridge; \
ip addr add 192.168.2.111/24 dev macvlan; \
ip link set macvlan up; \
ip route del 192.168.2.0/24 dev eth0; \
ip route del default; \
ip route add 192.168.2.0/24 dev macvlan; \
ip route add default via 192.168.2.1 dev macvlan;


永久
nano /etc/network/interfaces
#
auto eth0
iface eth0 inet manual

auto macvlan
iface macvlan inet static
  address 192.168.2.111
  netmask 255.255.255.0
  gateway 192.168.2.1
  dns-nameservers 192.168.2.1
  pre-up ip link add macvlan link eth0 type macvlan mode bridge
  post-down ip link del macvlan link eth0 type macvlan mode bridge
#
------------***可以用
auto eth0
iface eth0 inet manual
up ip link set eth0 promisc on
auto macvlan
iface macvlan inet static
   address 192.168.2.111
   netmask 255.255.255.0
   gateway 192.168.2.1
   dns-nameservers 192.168.2.1
   pre-up ip link add macvlan link eth0 type macvlan mode bridge
   post-down ip link del macvlan link eth0 type macvlan mode bridge
-----------

service docker start#启动docker
service docker stop#停止docker
service docker restart#重启docker
systemctl enable docker.service
systemctl disable docker.service
N1 ARMBIAN变成只读文件系统 Read-only file system
用U盘启动，运行一次 e2fsck /dev/mmcblk1p2 ，修复了。谢谢！
https://www.right.com.cn/forum/forum.php?mod=viewthread&tid=366836




https://www.right.com.cn/forum/thread-1375877-1-1.html

签到
docker pull asdaragon/qiandao
docker run -d --name qiandao -p 8765:80 --restart=always -v $(pwd)/qiandao/config:/usr/src/app/config   asdaragon/qiandao


移动硬盘
fdisk -l

mount /dev/sda1 /media/nnhd


配合docker版本MariaDB(mysql)数据库软件
因为默认nextcloud默认提醒不提倡SQLITE数据库，如果用到客户端同步的话，推荐使用MYSQL或者MariaDB
SQLITE用起来也没有问题，如果要用MYSQL这里讲一下怎么操作。
本人尝试了下docker版本MariaDB非常好用，作为MYSQL的分支，使用起来没有大的区别。
docker pull mariadb:10.4.1 本人测试该版本在N1上运行非常稳定，最新版有几率出错
部署容器
docker run -v /root/mysql/data:/var/lib/mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=1234 --privileged=true --restart unless-stopped --name mariadbs -d mariadb
这里  /data/mysql/自行修改安装位置，默认用户名root 默认密码root123这个自己改。
之后在nextcloud部署界面选择mariadb数据库，端口3306，数据库名称自己取一个比如nc就可以了，提示下mariadb容器启动需要一两分钟，不要刚启动就安装。
有些人这里会报错的，本人也碰到过，直接删除容器重新创建，多试几次应该就可以了。


第一句 拉取镜像：  docker pull arm64v8/nextcloud         默认最新版本，最新版貌似视频缩略图没有好的解决方案，本人一直使用低版本也没差，需要低端版本的话，加上:具体版本号，比如docker pull arm64v8/nextcloud:16.0.3，下同
                          没问题应该会出现很多文件下载的界面，依据网络环境来，如果没成功重试几次。多次失败建议换成阿里云源，方法自己百度，速度飞快。
		多挂载一个目录 用于aria2 download
                       当所有下载行均显示pull complete的时候，表示下载完成，开始第二句创建容器：
docker run -d -p 80:80 --name nextcloud -v /media/nnhd/nextcloud/:/var/www/html -v /media/nnhd/download/:/media/download --restart=always --privileged=true arm64v8/nextcloud
                        注意 8888端口自行修改，    /media/nnhd为部署目录位置，建议部署到移动硬盘，自行修改，小钢炮系统应该是/media/ABC为外置硬盘，armbian安装OMV后应该是sharedfolders文件夹里面为外置硬盘，前提是OMV里面要先挂载硬盘设置共享。
不出问题一会就完成了，接下来打开portainer 确认nextcloud已经启动的情况下，浏览器输入地址跟端口，应该就会出现安装界面了，剩下的就自己折腾吧，怎么配置网上已经很多教程了。


登陆等很久！！！


docker pull wahyd4/aria2-ui:arm64
docker run -d --name aria2-ui -p 2222:80 -e ARIA2_EXTERNAL_PORT=2222 -e RPC_SECRET=sadf654w -e ENABLE_AUTH=true -e ARIA2_USER=admin -e ARIA2_PWD=sadf654w -v /media/nnhd/download/:/data --restart=always wahyd4/aria2-ui:arm64

文件管理器 admin admin


一直困扰了好几天的问题终于解决了（如果有文字密集恐惧症，请直接执行第四步的2个命令就一步到位了），如果本贴看不懂如何将www-data加入自己文件夹目录的朋友 请按照我说的详细方法操作 我也是自己谷歌好久研究出来的（我也是小白 给小白看的 大神都只提个思路不提供具体步骤）：
第一步：先跟本帖一样先确认自己u盘是否正常挂上了
xshell ssh登录linux deploy配置的Ubuntu 这个会吧？然后盒子电视显示屏幕用es软件查看插上的u盘编号名称比如我的是D897-7DCC（以下内容大家根据自己的u盘识别编码替换我的） ，我的目录路径是/storage/D897-7DCC，然后deploy那边挂载目录我设置的是/mnt/media_rw/D897-7DCC（好像说填的就是/storage/D897-7DCC也可以 大家试试 ）

第二步：查看u盘是否成功挂载
输入命令：sudo df -h
比如我的显示如下：
Filesystem      Size  Used Avail Use% Mounted on
/dev/loop0      3.9G  1.3G  2.5G  34% /
tmpfs           892M  492K  891M   1% /dev
tmpfs           892M     0  892M   0% /dev/shm
/dev/block/sda   16G  300M   15G   2% /mnt/media_rw/D897-7DCC   #这里说明U挂载成功 如果你deploy那边设置挂载路径是/storage/D897-7DCC，这里也会显示/storage/D897-7DCC


再输入命令查看是否具备读写U盘权限
输入命令：sudo mkdir /mnt/media_rw/D897-7DCC/1122   #其中1122是我要创建的名字为1122的文件夹，大家可以随便取名
如果没返回任何错误，那就应该是成功了。大家也可以用winscp或者其他途径去查看/mnt/media_rw/D897-7DCC路径下的U盘是否成功创建了了一个1122的文件夹

第三步：用命令查看1122以及u盘下（如果还有其他文件夹的话）所有文件夹分别属于什么组和什么用户
输入命令：cd /mnt/media_rw/D897-7DCC    #意思就是进入u盘的文件夹目录，注意cd后面有个小空格别错了，命令后没返回错误就是正常进入了
再输入命令：ls -l    #注意中间空格   然后我的返回如下界面
wmc@localhost:/mnt/media_rw/D897-7DCC$ ls -l     #wmc是我创建的用户名 你的跟我一定不一样  这里/mnt/media_rw/D897-7DCC$的显示代表我进入u盘目录正常
total 9968
drwxrwx---. 2 aid_media_rw aid_media_rw     8192 Aug  6 21:18 1122        #第一个aid_media_rw代表1122的文件夹的用户，第二个位置的aid_media_rw代表1122组名。你的名字跟可能不一样
-rwxrwx---. 1 aid_media_rw aid_media_rw 10183060 Jul 31 18:32 linuxdeploy-2.1.0-237.apk   #我自己的U盘的文件不用管
drwxrwx---. 2 aid_media_rw aid_media_rw     8192 Aug  5 21:46 LOST.DIR


第四步：知道U盘下1122文件夹组名后，直接输入以下命令将www-data加入到aid_media_rw用户组就行，就是这个命令我找了好几天，主要是不理解原理什么感觉很难，其实就一条命令
输入命令：sudo usermod -a -G aid_media_rw www-data       #无返回错误，完美，其实前面都不用管直接输入第四步命令，一步到位，前面三步都是便于大家理解是什么情况。
然后输入命令：sudo service apache2 restart     #重启web服务后，看看是不是外挂U盘可以同步文件进去啦！哈哈哈






小钢炮:::

# 下载汉化包
curl -O -# http://www.iyuu.cn/usr/uploads/beikeyun.zip
# 解压并安装
unzip ./beikeyun.zip -o -d /usr/local/apps/dashboard
# 重启管理面板
/etc/init.d/S99dashboard restart

# 如果你的小钢炮是N1，再多执行一条命令：
cp /usr/local/apps/dashboard/aria2.html /usr/local/apps/dashboard/theme/darkmatter/templates/appcfg/aria2.html
# 重启管理面板
/etc/init.d/S99dashboard restart

更改docker源

vi /etc/docker/daemon.json
"https://2wfkcpg0.mirror.aliyuncs.com"


docker volume create portainer_data
docker pull portainer/portainer:latest
上传public
docker run -d -p 9000:9000 --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data -v /public:/public --name portainer portainer/portainer:latest


docker pull unifreq/openwrt-aarch64:r20.03.05

ip link set eth0 promisc on

docker network create -d macvlan --subnet=192.168.2.0/24 --gateway=192.168.2.1 -o parent=eth0 macnet

docker run --restart always -d --network macnet --privileged --name openwrt unifreq/openwrt-aarch64:r20.03.05 /sbin/init

进入openwrt容器修改network设置
vi /etc/config/network
option ipaddr 192.168.2.11
reboot



手动穿docker ui
docker import openwrt-armvirt-64-h-rootfs.tar.gz lff-openwrt-hw:20.03.08
docker run --restart always -d --network macnet --privileged --name openwrt lff-openwrt-hw:20.03.08 /sbin/init

docker import openwrt-armvirt-64-s-rootfs.tar.gz lff-openwrt-ss:20.03.08
docker run --restart always -d --network macnet --privileged --name openwrt lff-openwrt-ss:20.03.08 /sbin/init

ip link set eth0 promisc on

docker network create -d macvlan --subnet=192.168.2.0/24 --gateway=192.168.2.1 -o parent=eth0 macnet



docker pull 80x86/baidupcs

docker run -d \
--name=baidupcs \
--restart always \
-v /media/nnhd/n1/baidupcs:/app/Downloads \
-v /media/nnhd/n1/baidupcs-config:/app/.config/BaiduPCS-Go \
-p 5299:5299 \
80x86/baidupcs:latest



--最新

https://wiki.omv-extras.org/