---
title: Docker笔记
comments: true
tags:
  - Docker
categories:
  - linux
date: 2021-08-11 17:13:16
updated:
---

Docker笔记

<!--more-->

## 安装

### Nacos Server

```bash
docker run -itd --name nacos -e MODE=standalone -p 8848:8848 -d nacos/nacos-server:latest
```

MODE    cluster/standalone    cluster/standalone default cluster

http://192.168.2.211:8848/nacos

nacos nacos
