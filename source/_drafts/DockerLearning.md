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

MODE	cluster/standalone	cluster/standalone default cluster

http://192.168.2.211:8848/nacos

nacos nacos
