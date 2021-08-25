---
layout: post
title: 'Mac开发环境（1）：Homebrew安装配置JDK MySql'
subtitle: 'Homebrew openjdk1.8 MySql5.7.4'
date: 2021-07-09
author: Dave
cover: ''
tags: MAC开发环境 
---

>习惯了win下的环境，最近换开发环境了，记录以下

**最前面**
- 下一节：[Mac开发环境（2）：maven安装配置+idea配置](https://picc0lo.top/2021/07/09/macForJavaDev-maven.html)

>springboot 2.x +java 8 ，maven需要3.2以上，

## homebrew安装与配置

- [Homebrew](https://brew.sh/index_zh-cn)
- [mac下镜像飞速安装Homebrew教程](https://zhuanlan.zhihu.com/p/90508170)
- [中科大Homebrew源](http://mirrors.ustc.edu.cn/help/brew.git.html#)
- [清华Homebrew源](https://mirrors.tuna.tsinghua.edu.cn/help/homebrew/)
- [腾讯Homebrew源｜切换](https://mirrors.cloud.tencent.com/help/homebrew-bottles.html)

Homebrew为mac上优秀好使的第三方包管理器，推荐使用。
Homebrew主要有四个部分组成: brew、homebrew-core 、homebrew-bottles、homebrew-cask。

  | 名称	| 说明| 
  | :-----------| :----------- |
  | brew	| Homebrew 源代码仓库| 
  | homebrew-core	| Homebrew 核心软件仓库| 
  | homebrew-bottles	| Homebrew 预编译二进制软件包| 
  | homebrew-cask	| MacOS 客户端应用| 

**1.安装与卸载**

官网bash安装（不做直接推荐，可以**出国留学**的可以使用）：

```powershell
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

中科大镜像源安装（推荐，国内比较快，相对稳定）且有教程维护[brew.idayer.com](https://brew.idayer.com/)

```powershell
/bin/bash -c "$(curl -fsSL https://cdn.jsdelivr.net/gh/ineo6/homebrew-install/install.sh)"
```

>卸载推荐使用中科大镜像源卸载

  ```powershell
  /bin/bash -c "$(curl -fsSL https://cdn.jsdelivr.net/gh/ineo6/homebrew-install/uninstall.sh)"
  ```


**brew默认安装路径: /usr/local/Cellar**

>也可指定，参照[Homebrew学习（二）之安装、卸载、更新](https://www.cnblogs.com/kunmomo/p/11267429.html),安装Homebrew时对安装路径进行指定，直接安装在不需要系统root用户授权就可以自由读写的目录下.
  
```powershell
## 指定路径安装，我还没试过
<install path> -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```


**2.配置**

**brew、homebrew/core是必备项目，homebrew/cask、homebrew/bottles按需设置**

- 查看本地镜像源地址
  
  ```powershell 
  ## 查看 brew.git 当前源
  cd "$(brew --repo)" && git remote -v
  ## 查看 homebrew-core.git 当前源
  cd "$(brew --repo homebrew/core)" && git remote -v
  ```

- 切换国内镜像源
  
  >以中科大镜像源为例，另有阿里源/清华源/腾讯源可选，有人说腾讯源（见[推荐一个Mac brew软件源，迄今为止最快的源](https://zhuanlan.zhihu.com/p/72251385)）很快，我用着也不快；有人说阿里源东西较少，推荐的人较少，就不主做推荐了。

  - [Homebrew更换国内镜像源（中科大、阿里、清华）](https://blog.csdn.net/H_WeiC/article/details/107857302)
  - [Homebrew更换国内源提速](https://blog.csdn.net/toopoo/article/details/104709816/?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-0.control&spm=1001.2101.3001.4242)

  ```powershell
  ## 替换brew.git:
  cd "$(brew --repo)" && git remote set-url origin https://mirrors.ustc.edu.cn/brew.git

  ## 替换homebrew-core.git:
  cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core" && git remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git

  ## 替换homebrew-cask.git:
  cd "$(brew --repo)/Library/Taps/homebrew/homebrew-cask" && git remote set-url origin https://mirrors.ustc.edu.cn/homebrew-cask.git

  ## zsh 替换homebrew-bottles镜像(macOS 10.15 Catalina后zsh为macOS的默认shell，之前为bash。)
  ## 此为长期切换，非临时切换
  echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles' >> ~/.zshrc

  ## 修改使其立即生效
  source ~/.zshrc

  ## bash 替换homebrew-bottles镜像
  ## 此为长期切换，非临时切换
  echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles' >> ~/.bash_profile

  ## 修改使其立即生效
  source ~/.bash_profile

  ## 刷新源
  brew update
  ```





- 如遇需求需重置镜像源
  ```powershell
  ## 重置brew.git:
  cd "$(brew --repo)" && git remote set-url origin https://github.com/Homebrew/brew.git

  ## 重置homebrew-core.git:
  cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core" && git remote set-url origin https://github.com/Homebrew/homebrew-core.git

  ## 重置homebrew-cask.git:
  cd "$(brew --repo)/Library/Taps/homebrew/homebrew-cask" && git remote set-url origin https://github.com/Homebrew/homebrew-cask.git

  ## zsh 重置homebrew-bottles镜像(macOS 10.15 Catalina后zsh为macOS的默认shell，之前为bash。)
  ## 此为长期切换，非临时切换
  手动删除 ~/.zshrc 中的 HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles

  ## 修改使其立即生效，bash同理
  source ~/.zshrc

  ## 刷新源
  brew update
  ```

**3.常用命令**

```powershell
## 查询软件包
brew search 软件包名

## 查询软件包信息
brew info 软件包名

## 安装软件包（用@来区分版本）
brew install 软件包名

## 举例：brew install maven@3.3

brew install --cask 软件包名

## 举例：brew install --cask docker

## 卸载
brew uninstall 软件包名

## 更新 Homebrew
brew update 

## 查看 Homebrew 配置信息：
brew config 

## 查看已安装包列表
brew list

## 查看已安装软件包的路径
brew list 软件包名

## brew 诊断
brew doctor

## 查看版本
brew -v

## 查看帮助
brew -h
```
## jdk安装与配置

>在这里我选择brew安装openjdk 1.8，也可直接去红帽官网下载openjdk1.8
- [Download the Red Hat build of OpenJDK](https://developers.redhat.com/products/openjdk/download)
>在这里我参考了以下文档
- [新 Mac 如何优雅地配置 Java 开发环境](https://zhuanlan.zhihu.com/p/298535991)


**1.安装jdk**

  ```powershell
  brew search openjdk
  brew install openjdk@8
  ```
**2.配置jdk**

```powershell
## 软连接 jdk安装路径 /Library/Java
sudo ln -sfn /usr/local/opt/openjdk@8/libexec/openjdk.jdk /Library/Java/JavaVirtualMachines/openjdk-8.jdk

## 检查 /Library/Java 结构
admin@admindeMacBook-Pro-2 / % tree /Library/Java
/Library/Java
├── Extensions
└── JavaVirtualMachines
    └── openjdk-8.jdk -> /usr/local/opt/openjdk@8/libexec/openjdk.jdk

## 执行`/usr/libexec/java_home`
/usr/libexec/java_home

## 查看版本
admin@admindeMacBook-Pro-2 / % java -version 
openjdk version "1.8.0_282"
OpenJDK Runtime Environment (build 1.8.0_282-bre_2021_01_20_16_06-b00)
OpenJDK 64-Bit Server VM (build 25.282-b00, mixed mode)
```

## mysql安装

>在这里我选择brew安装mysql5.7，也可直接去红帽官网下载mysql5.7

```powershell
brew search mysql
brew install mysql@5.7
```

```powershell
## mysql启动/停止
mysql.server start/stop

## mysql状态查询
mysql.server status

## 示例
admin@admindeMacBook-Pro-2 / % mysql --version
mysql  Ver 14.14 Distrib 5.7.34, for osx10.16 (x86_64) using  EditLine wrapper

admin@admindeMacBook-Pro-2 / % mysql.server status
 ERROR! MySQL is not running, but PID file exists

admin@admindeMacBook-Pro-2 / % mysql.server start
Starting MySQL
 SUCCESS! 

admin@admindeMacBook-Pro-2 / % mysql.server status
 SUCCESS! MySQL running (5393)
 ```

 更多mysql操作见mysql专题
 - []()
