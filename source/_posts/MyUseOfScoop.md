---
title: Scoop常用
comments: true
tags:
  - Scoop
  - Windows
categories:
  - Device
date: 2020-12-21 12:24:00
updated: 2021-02-16 18:36:11
---

search	搜索软件名
install	安装软件
update	更新软件
status	查看软件状态
uninstall	卸载软件
info	查看软件详情

<!--more-->

## 安装Scoop

``` powershell
#设置程序安装目录
$env:SCOOP='D:\Program\Scoop'
[environment]::setEnvironmentVariable('SCOOP',$env:SCOOP,'User')
#开始安装
iwr -useb get.scoop.sh | iex
```

## 常用软件

### 安装aria2

scoop install aria2

好多软件是从官网，github，amazon下载的，网速比较渣，可以先安装aria2 (scoop install aria2)，安装后 scoop 会优先调用 aria2 来下载软件；
aria2 一些配置：
aria2-enabled (default: true)
aria2-retry-wait (default: 2)
aria2-split (default: 5)
aria2-max-connection-per-server (default: 5)
aria2-min-split-size (default: 5M)
scoop config aria2-enabled false

### 安装notepad3

scoop install notepad3

### 安装git
scoop install git
配置
git config --global user.name "Nannan"
git config --global user.email "1041836312@qq.com"
ssh-keygen -t rsa -C "1041836312@qq.com"

### 安卓android-sdk
 scoop install android-sdk

### 安装node
scoop install nodejs-lts
淘宝源
npm install -g cnpm --registry=https://registry.npm.taobao.org
cnpm install -g @vue/cli
平常使用
npm install -g @vue/cli --registry=https://registry.npm.taobao.org
npm install -g typescript
npm install -g yarn

### 安装jdk
scoop bucket add java
scoop install openjdk11

### 安装JetBrains-toolbox
scoop bucket add extras
scoop install jetbrains-toolbox

### 安装cmder
scoop install cmder

### 安装tomcat
scoop bucket add extras
scoop install tomcat

### 安装lua
scoop install lua-for-windows

### 安装chrome
scoop install googlechrome

### 安装notepad++
scoop install notepadplusplus

### 安装ffmpeg
scoop install ffmpeg

### 安装mysql
scoop install mysql
更改my.ini D:\Development\mysql\data
mysqld --initialize --console
可以删除scoop里面mysql下data文件夹
管理员 mysqld --install mysql8
net start mysql8
x-%c;Qdu5Br-
mysql -u root -p
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456';  
mysqld --standalone

### 安装python/anaconda
scoop bucket add extras
scoop install anaconda3

### 安装ssh stfp
scoop install mobaxterm

### 安装hugo
scoop install hugo-extended

### 安装Typora
scoop install Typora

### 安装Windows Terminal 
https://zhuanlan.zhihu.com/p/137251716

### 安装mariadb
scoop install mariadb