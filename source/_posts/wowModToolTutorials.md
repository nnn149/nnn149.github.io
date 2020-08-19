---
title: wow工具使用教程
comments: true
tags:
  - wow
categories:
  - game
date: 2020-08-19 23:59:17
updated: 2020-08-19 23:59:17
---

这里以修改 wow3.3.5为例
<!--more--> 

需要准备wow4.x-8.x的客户端，我用的是4.x的
使用M2ModRedux_8.2.3版本的m2转换器
blender脚本使用 blander_scipts_8.2.0版本
m2降级工具用 MultiConverter_3.1.1版本
模型预览工具我用的是WoWModelViewer7.x,高于7的不能预览wow3.x和wow4.x  (更高版本未测试)
blender使用2.8以前，我使用 blender-2.79b-windows64

步骤

用WoWModelViewer导出m2和skin文件 (从wow4.x以上版本导出)
M2ModRedux把m2文件转成m2i文件
blender导入m2i编辑模型，然后导出为m2i文件
M2ModRedux把刚才导出的m2i转成m2文件
用MultiConverter降级m2文件
用MPQmaster放到mpq文件内

链接
https://bitbucket.org/%7Bdd60bb38-a5e5-4c9b-b635-64391cafe8cf%7D/
https://github.com/notagain/Tools
http://www.zezula.net/en/mpq/main.html

