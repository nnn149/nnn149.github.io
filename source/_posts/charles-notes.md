---
title: Charles抓包使用
comments: true
tags:
  - Charles
categories:
  - tool
date: 2022-01-15 16:24:00
updated: 2022-01-15 16:24:00
---

使用Charles分析http协议

抓包java和.net程序

<!--more-->

## 安装Charles

[官网](https://www.charlesproxy.com/https:/)

## 配置Charles

### SSL

菜单: Proxy -> SSL Proxying Settings -> SSL Proxying选项卡 -> Enable SSL Proxying  勾选
底下Include添加需要拦截的域名

菜单: Proxy ->
Windows Proxy 勾选
Start SSL Proxying
Start Recording

## 抓包C#程序

### 解决SSL报错

添加代码
` ServicePointManager.ServerCertificateValidationCallback = (sender, certificate, chain, sslPolicyErrors) => true; `

## 抓包Java程序

```java
        System.setProperty("https.proxyHost", "127.0.0.1");
        System.setProperty("https.proxyPort", "8888");
```
---
