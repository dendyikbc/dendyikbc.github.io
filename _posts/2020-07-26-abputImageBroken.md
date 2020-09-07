---
layout: post
title: '关于本站图裂影响显示的解决办法'
subtitle: '等同于 github图片裂开问题'
date: 2020-07-26
author: Dave
tags: GitHub 
---


![](https://raw.githubusercontent.com/dendyikbc/PicGoBed/master/img/light-city-boy-piccolo.jpg)

## 关于本站图裂影响显示的解决办法

由于使用的开源图床PicGo，如果图裂，解决办法等同于 github图片裂开问题

正确解决办法

- 1.获取**raw.githubusercontent.com** 最新IP

    进入[IPAddress.com](https://www.ipaddress.com/)，查询**raw.githubusercontent.com**的最新IP
    ```java
    //2020/07/26 最新IP记录
    199.232.68.133
    ```
- 2.修改主机hosts文件
    - Windows路径 **C:\Windows\System32\drivers\etc\hosts**
    - Linux路径 **/etc/hosts**

    添加
    ```java
    199.232.28.133 raw.githubusercontent.com
    ```
    
    当然也可以有细化的版本
    
    
    ```java
    # GitHub Host Start
    185.199.108.154 github.githubassets.com
    199.232.68.133 camo.githubusercontent.com
    199.232.68.133 github.map.fastly.net
    199.232.69.194 github.global.ssl.fastly.net
    140.82.113.3 github.com
    140.82.112.5 api.github.com
    199.232.68.133 raw.githubusercontent.com
    199.232.68.133 user-images.githubusercontent.com
    199.232.68.133 favicons.githubusercontent.com
    199.232.68.133 avatars5.githubusercontent.com
    199.232.68.133 avatars4.githubusercontent.com
    199.232.68.133 avatars3.githubusercontent.com
    199.232.68.133 avatars2.githubusercontent.com
    199.232.68.133 avatars1.githubusercontent.com
    199.232.68.133 avatars0.githubusercontent.com
    # GitHub Host End
    ```

    - 或者使用hosts切换工具 [SwitchHosts](https://github.com/oldj/SwitchHosts)直接添加
    ![](https://raw.githubusercontent.com/oldj/SwitchHosts/master/screenshots/sh_light.png)
