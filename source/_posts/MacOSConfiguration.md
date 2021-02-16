---
title: MacOS配置
comments: true
tags:
  - MacOS
categories:
  - Device
date: 2021-02-15 03:23:34
updated: 2021-02-15 03:23:34
---

自用MacOS设置

<!--more-->

### 挂载EFI分区

`sudo diskutil list`

`sudo diskutil mount disk0s1`

### 系统偏好设置

- 系统偏好设置
  - 网络
    - 以太网-高级-硬件-速度：1000baseT
  - 声音-在菜单栏中显示音量
  - 通用
    - 显示滚动条：始终
    - 在滚动条中点按：跳到点按位置
  - 程序坞
    - 放大
    - 将窗口最小化为应用程序图标
    - 自动隐藏
      - defaults write com.apple.Dock autohide-delay -float 0 && killall Dock   # 取消延迟
      - defaults delete com.apple.Dock autohide-delay && killall Dock   #恢复延迟
  - 调度中心
    - 触发角
      - 左上：调度中心
      - 右上：应用程序窗口
      - 左下：启动台
      - 右下：桌面
  - 软件中心：全部关闭
  - 键盘
    - 修饰健
      - Control键：Control
      - Option键：Option
      - Command键：Command

- 访达
  - defaults write com.apple.finder _FXShowPosixPathInTitle -bool YES    #顶上显示路径

### 软件

- homebrew
  - mos
  - Maccy
  - openjdk@11 ? AdoptOpenJDK
  - node@14
  - dbeaver-community
  - bob
  - github  (github desktop)
  - tencent-lemon
  - postman
  - another-redis-desktop-manager
  - hexo
  - iTerm2
    - homebrew/cask-fonts/font-hack-nerd-font
    - https://blog.biezhi.me/2018/11/build-a-beautiful-mac-terminal-environment.html
- Typora
- clash x
- MacPass
- Onedrive
- IINA
- Logitech
- AMD Power Gadget
- AppCleaner
- SteelSeries ExactMouse Tool
- AppStore
  - CotEditor

