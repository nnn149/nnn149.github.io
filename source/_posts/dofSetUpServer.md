---
title: dof服务端配置
comments: true
tags:
  - DNF
categories:
  - Game
date: 2020-09-02 23:40:21
updated: 2020-09-02 23:40:21
---

搭建dof双库端


<!--more-->

## 快速开始

my.ini
```
[mysqld]
sql-mode="NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION"
```

```
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY'123456' WITH GRANT OPTION;

修改虚拟机ip
vi /etc/sysconfig/network-scripts/ifcfg-eth0


设置数据库ip地址
UPDATE db_connect SET db_ip='192.168.2.200';
设置允许ip
INSERT INTO geo_allow (allow_ip) VALUES ('192.168.2.0');
INSERT INTO geo_allow (allow_ip,allow_c_code) VALUES ('192.168.2.200','CN');


更改服务端cfg配置的ip。 160为原虚拟机ip
cd /home/dxf


sed -i "s/你在虚拟机里面用到的ip/你的外网ip/g"  `find . -type f -name "*.tbl"`
sed -i "s/你在虚拟机里面用到的ip/你的外网ip/g"  `find . -type f -name "*.cfg"`

sed -i "s/192.168.1.160/192.168.2.200/g"  `find . -type f -name "*.tbl"`
sed -i "s/192.168.1.160/192.168.2.200/g"  `find . -type f -name "*.cfg"`

sed -i "s/192.168.1.200/192.168.2.207/g"  `find . -type f -name "*.tbl"`
sed -i "s/192.168.1.200/192.168.2.207/g"  `find . -type f -name "*.cfg"`
```



https://docs.qq.com/doc/DRFhlek14RmZiZ3Jl