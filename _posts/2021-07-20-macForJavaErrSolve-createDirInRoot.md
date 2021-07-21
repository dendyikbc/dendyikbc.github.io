---
layout: post
title: 'Mac开发环境问题（1）：Big Sur根目录无法创建文件夹'
subtitle: ''
date: 2021-07-20
author: Dave
cover: ''
tags: MAC开发环境 
---



>问题起源：springboot项目启动报错 在根目录下无法创建/data 存放项目日志 

```powershell
ERROR in ch.qos.logback.core.rolling.RollingFileAppender[APP] - Failed to create parent directories for [/data/apps/项目名/logs/app.log]
```



**试着手动创建去解决，失败**

```powershell
admin@admindeMacBook-Pro-2 ~ % cd /         
admin@admindeMacBook-Pro-2 / % sudo mkdir data
Password:
mkdir: data: Read-only file system
```



遂搜寻解决方案，搜寻关键词：`mac 根目录无法创建` `mac / 权限 log` `mac 启动项目 文件夹` `mac 运行项目 文件夹` `mac 启动项目 权限` `mac 运行项目 权限`

**放在前面**


- [MAC升级到最新Catalina系统后无法创建目录](https://blog.csdn.net/hanerer1314/article/details/107560309?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_title~default-0.control&spm=1001.2101.3001.4242)
- [Mac升级Catalina，根目录下无法创建个人文件夹](https://blog.csdn.net/ZERO_TO_NIL/article/details/102499503?utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-1.control&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-1.control)

## 根目录无法创建文件夹

> 此故障出现在10.15 Catalina版本及以后，系mac新的安全性策略所带来的`欢快`。

> 我的开发环境：macOS 11.3 Big Sur

**0.手动修改配置文件 日志写入路径**

> 以公司项目为例记解决办法

- 进入公司pass平台，配置中心，切入项目当前环境（sit/uat），拷贝配置文件

- 配置文件内容拷贝到本地分支的启动类配置文件中，并修改其中logdir
- 配置文件如需转换，记得转换。
- [ToYaml.com -- Convert between Properties and YAML Online ](https://www.toyaml.com/index.html)



```
## 原配置
logdir: /data/apps/java-${spring.application.name}-controller/logs

## 本地修改路径
logdir: /Users/admin/Downloads/java-${spring.application.name}-controller
```

> `都是团队开发，此法不建议使用，git 极不方便`





**1.关闭SIP，/ 下新建软连接到用户 /data**

SIP 全称为「System Integrity Protection」即「[系统完整性保护](https://support.apple.com/zh-cn/HT204899)」,是系统级的权限操作，我们无法直接关闭它，需要前往「[macOS 恢复功能](https://support.apple.com/zh-cn/HT201314)」下进行。

  > 先查看SIP状态：`csrutil status` 默认enabled。
  >
  > `此法不是苹果官方推荐，但貌似Catalina可用，BigSur不可用。`
  >
  > - [升级到macOS Big Sur后，以前在根目录下创建的文件夹找不到了](https://discussionschinese.apple.com/thread/252142331)



  - 1.重启电脑 长按`command`+`R` 进入安全模式

  - 2.关闭SIP：打开命令控制台输入 `csrutil disable`

  - 3.重启电脑（正常启动）

  - 4.打开 terminal 输入 `csrutil status` 此时状态应该是 `disabled`

  - 5.在 terminal 中继续输入 `sudo mount -uw /`

  - 6.将需要的目录软链接到根目录

    > `sudo ln -s /Users/admin/data /`

  - 7.重复1，2，3步骤，第2步命令修改为 `csrutil enable` 

  



**2.man synthetic.conf，官方软连接方案**

> 最终解决办法，Big Sur可用，Monterey未知。

```powershell
## 0.查看synthetic.conf说明
man synthetic.conf
```

```powershell
## 1.vim 修改synthetic.conf（没有会创建）
sudo vim /etc/synthetic.conf

## 2.添加一行记录(使用 tab 进行分割，空格 换行，使用空格分割会发现重启无效)

## 以此处举例

data	/Users/admin/data

## 重启后，在根目录下已经创建好 data 软连接到 /Users/admin/data
ls -al
......
lrwxr-xr-x   1 root  wheel    17  7 21 14:45 data -> /Users/admin/data
......
```





