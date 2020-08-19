---
title: wowAddMount
comments: true
tags:
  - wow
categories:
  - game
date: 2020-08-19 00:06:58
updated: 2020-08-19 00:06:58
---

新建一个mpq文件 patch-zhCN-7.MPQ 选择兼容性

里面文件夹 Character  DBFilesClient  Interface

导入模型包到对应文件夹



## dbc修改

需要修改dbc文件 spell (技能)  SpellIcon(图标)		CreatureModelData	CreatureDisplayInfo



先修改spell

搜索飞毯或华丽的飞毯 虚空幼龙 28828，复制行，取一个最大的id    96000



然后修改生物模型CreatureModelData	dbc  ,随便复制一个 id 6000

第2列标识为2代表坐骑，第三列mpq文件内m2文件路径 Character\WhimsyshireCloudMount\WhimsyshireCloudMount.m2    保存



修改  CreatureDisplayInfo dbc   起到数据库和dbc相连作用. 随便找一行复制 id 50000 

第二列对应刚才CreatureModelData添加的id(6000)

第五列是模型尺寸倍数,7-9 把模型包mpq，里的blp文件名，不带后缀 WhimsyshireCloudMount_Angry

还需要新的贴图就新建一行，第二列id一样7-9填新的贴图名

保存



继续做技能spell

72为6 73为6 是陆地坐骑 74为6为飞行坐骑 (效果123)

效果基础值 81水 82陆=309 83空=99 速度

效果光环96 97 98列代表是否是飞行坐骑  78 32 0代表陆地坐骑  78 207 32 代表飞行坐骑

效果复合值 111列 填写数据库 entry 字段填写4000

 creature_template 表 插入一行 entry字段填写4000,modelid1 字段 填写 CreatureDisplayInfo 的id  50000 , name字段 筋斗云

视觉效果 132列 ，根据AnimationData.dbc 中定义的来填写。  12884蹲伏的动作

修改 SpellIcon.dbc 复制一行 id 4376 第二列填写 Interface\Icons\INV_Mount_WhimsyshireCloudMount01

修改 spell里的技能图标134列为刚才的技能图标id

141列修改名字，

171还是175 修改描述

保存



把修改过的dbc添加到创建的mpq的 DBFilesClient 文件夹内



数据库 creature_model_info表 添加一条

modelid或者DisplayID为 CreatureDisplayInfo的id  50000，gender为2，应为坐骑没性别 



开启游戏，gm指令 .Learn id  学习法术