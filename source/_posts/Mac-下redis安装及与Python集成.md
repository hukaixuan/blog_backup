---
title: "Mac下redis安装及与Python集成"
date: 2017.04.17 11:20
tags:
 - Mac
 - Redis
 - Python
---

1. 终端执行`brew install redis` 来安装redis

2. 安装完成后，执行 `redis-server` 启动redis服务，出现如下内容则安装成功：
![redis服务启动成功](http://upload-images.jianshu.io/upload_images/3340699-f73f704239ea43d7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3. `command + T ` 新建一个标签页，执行 `pip install redis`

4. 测试redis:
    `python`    
![Hello world](http://upload-images.jianshu.io/upload_images/3340699-7b34edf0c0c99dab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
