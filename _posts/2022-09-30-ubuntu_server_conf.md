---
layout: post
title: 'Ubuntu server config'
subtitle: '主要为闲置电脑搭建ubuntu server的记录'
date: 2022-09-30
author: Dave
cover: ''
tags: Linux
---

> 因为个人对ubuntu比较熟悉，所以这里选择安装ubuntu_server_20.04 LTS 版本

- 安装过程记得安装ssh

### 1.1 账户配置

```powershell
vim /etc/hostname # 修改主机名

## sudo passwd {用户名} 修改密码范式
sudo passwd root #启用root账号并设置密码，根据提示输入2次密码

## su - {用户名}  切换用户范式
su - root #切换到root用户，需输入root密码
su - dave #切换到dave用户

## sudo passwd -l {用户名} 禁用用户范式
sudo passwd -l root # 禁用root账号，如果要启用，输入sudo passwd root再次设置root密码

## adduser {用户名} 普通用户添加范式
adduser dave  #创建一个普通用户dave

# 添加用户并生成home目录
sudo useradd username -m
# 添加用户到sudo组
sudo usermod -aG sudo username
# 设置用户名密码
sudo passwd username


## home目录迁移
# 在 data 下创建 username 文件夹
mkdir /data/username
# 迁移数据到新的home目录
mv /home/username/* /data/username/*
# 修改 home 目录到 /data 下
sudo usermod -d /data/username username
```

### 1.2 网络配置

```bash
## ubuntu_server_20.04 LTS 使用 netplan ⽅式，配置⽂件路径为：/etc/netplan/00-installer-config.yaml 

vim /etc/netplan/00-installer-config.yaml

## 具体内容 
network:
  ethernets:
    enp3s0:
      addresses: [192.168.1.106/24]
      dhcp4: false
      optional: true
      nameservers:
        addresses: [223.5.5.5,8.8.8.8]
    enp4s0:
      addresses: [192.168.1.107/24]
      dhcp4: false
      optional: true
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses: [223.5.5.5,8.8.8.8]
  version: 2
  
## 配置生效
netplan apply

## 查阅ip是否固化
ifconfig | grep inet
```

### 1.3 软件源配置

- [清华镜像：Ubuntu 镜像使用帮助](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/)

```powershell
## 查看系统信息
☁  ~  cat /proc/version
Linux version 5.15.0-50-generic (buildd@lcy02-amd64-086) (gcc (Ubuntu 11.2.0-19ubuntu1) 11.2.0, GNU ld (GNU Binutils for Ubuntu) 2.38) #56-Ubuntu SMP Tue Sep 20 13:23:26 UTC 2022
☁  ~  lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 22.04.1 LTS
Release:        22.04
Codename:       jammy

## 换源
cd /etc/apt/
sudo cp sources.list sources.list.bak
sudo vim sources.list  

## 替换内容

# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-proposed main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-proposed main restricted universe multiverse


## 记得update
sudo apt-get update
sudo apt-get upgrade
```

### 1.5 shell



#### zsh

```bash
## 查看shell
cat /etc/shells  
## install zsh
apt install zsh

## 将zsh设置成默认shell（不设置的话启动zsh需要zsh命令即可）
chsh -s /bin/zsh 
```

#### oh-my-zsh

```bash
## 官方
sh -c "$(wget -O- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
## gitee镜像
sh -c "$(wget -O- https://gitee.com/mirrors/oh-my-zsh/raw/master/tools/install.sh)"

sh -c "$(curl -fsSL https://gitee.com/shmhlsy/oh-my-zsh-install.sh/raw/master/install.sh)"

sh -c "$(curl -fsSL #!/bin/sh # # This script should be run via curl: # sh -c "$(curl -fsSL https://raw.githubuserconten \
    | sed 's|^REPO=.*|REPO=${REPO:-mirrors/oh-my-zsh}|g' \
    | sed 's|^REMOTE=.*|REMOTE=${REMOTE:-https://gitee.com/${REPO}.git}|g')"
```

插件

```bash
## 插件推荐

git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

## 同步修改
plugins=(其他的插件 web-search zsh-syntax-highlighting zsh-autosuggestions)
## 主题设置
cloud
```



#### shell环境问题

1.若创建用户登录服务器，终端中，按上下左右键，回显^[[A、^[[B、^[[C、^[[D情况，则终端环境出现问题

出现上述问题的时候，命令行往往只显示一个提示符：

```bash
$
```

可尝试先用 bash 解决

```bash
## 1、切换 bash 模式
bash
## 2、shell 变更
chsh
## 在Login Shell [*]后输入
/bin/bash
## 以后每次连接都会使用 bash
## zsh类似
```

### 1.4 mysql

- [Ubuntu20.04 离线安装 MySQL 5.7](https://blog.csdn.net/emergencysun/article/details/124229238)

#### 正常安装mysql

```bash
## libtinfo.so.5缺这个 如何解决的
sudo apt-get install libncurses5
## 依赖修复
sudo apt --fix-broken install
```



```bash
wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-server_5.7.31-1ubuntu18.04_amd64.deb-bundle.tar
tar -xvf mysql-server_5.7.31-1ubuntu18.04_amd64.deb-bundle.tar
rm -f mysql-testsuite_5.7.31-1ubuntu18.04_amd64.deb
rm -f mysql-community-test_5.7.30-1ubuntu18.04_amd64.deb
sudo dpkg -i mysql-*.deb
```



```powershell
mysql -V										# mysql版本信息
netstat -tap | grep mysql   # mysql服务查阅

## e.g.
☁  ~  mysql -V
mysql  Ver 14.14 Distrib 5.7.31, for Linux (x86_64) using  EditLine wrapper
☁  ~  netstat -tap | grep mysql
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp        0      0 localhost:mysql         0.0.0.0:*               LISTEN      -            
```



#### mysql端口开放远程登录

```powershell
## 配置mysql
sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf
## 注释掉bind ip 并配置 port = 3306

port            = 3306
# By default we only accept connections from localhost
# bind-address  = 127.0.0.1



## 允许root访问
mysql -u root -p				## 输入密码
use mysql   		    		## 选择访问mysql库
update user set host = '%' where user = 'root'; 		## 使root能再任何host访问
FLUSH PRIVILEGES;       ## 刷新

## 开放root账号所有权限
grant all privileges on *.* to 'root'@'%' identified by 'yourpasswd';
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'yourpasswd' WITH GRANT OPTION;



## 查看端口开放
netstat -an|grep 3306
## 探测目标主机1-10000范围内所开放的所有端口
nmap 192.168.1.1
## 探测指定地址端口开放
nmap 192.168.1.1 -p3306


## e.g.
☁  ~  netstat -an|grep 3306
tcp6       0      0 :::3306                 :::*                    LISTEN  
☁  ~  nmap 192.168.1.111 -p3306
Starting Nmap 7.80 ( https://nmap.org ) at 2022-10-20 18:15 UTC
Nmap scan report for ubuntuserver (192.168.1.17)
Host is up (0.00027s latency).

PORT     STATE SERVICE
3306/tcp open  mysql

Nmap done: 1 IP address (1 host up) scanned in 0.13 seconds
```



### 1.5 安装jdk

```powershell
sudo apt install openjdk-11-jdk
sudo apt install openjdk-17-jdk
## 切换jdk
sudo update-alternatives --config java

## e.g.
☁  ~  java -version
openjdk version "1.8.0_342"
OpenJDK Runtime Environment (build 1.8.0_342-8u342-b07-0ubuntu1~22.04-b07)
OpenJDK 64-Bit Server VM (build 25.342-b07, mixed mode)
☁  ~  sudo update-alternatives --config java
There are 3 choices for the alternative java (providing /usr/bin/java).

  Selection    Path                                            Priority   Status
------------------------------------------------------------
  0            /usr/lib/jvm/java-17-openjdk-amd64/bin/java      1711      auto mode
* 1            /usr/lib/jvm/java-11-openjdk-amd64/bin/java      1111      manual mode
  2            /usr/lib/jvm/java-17-openjdk-amd64/bin/java      1711      manual mode
  3            /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java   1081      manual mode
```







### jekyll环境

[在阿里云服务器上部署Jekyll博客](https://blog.csdn.net/yfren1123/article/details/104136715)

```
sudo apt-get update
sudo apt-get install ruby-full make gcc nodejs build-essential patch


```



### 工具安装

#### iperf3

```bash
sudo apt install iperf3
```

- [iperf-doc](https://iperf.fr/iperf-doc.php)
- [iperf3使用方法](https://zhuanlan.zhihu.com/p/314727150)

```bash
## iperf3 server start
iperf3 -s 
## 测试UDP吞吐量
iperf3 -u -c 192.168.1.107 -b 100m -t 60
## 进行上下行带宽测试（TCP双向传输）
iperf3 -c 192.168.1.107 -d -t 60
```

#### speedtest-cli

```bash
sudo apt install speedtest-cli

## 测速测试
☁  ~  speedtest-cli
Retrieving speedtest.net configuration...
Testing from China Telecom (222.210.154.238)...
Retrieving speedtest.net server list...
Selecting best server based on ping...
Hosted by China Mobile Henan 5G (Zhengzhou) [1002.18 km]: 38.196 ms
Testing download speed................................................................................
Download: 131.17 Mbit/s
Testing upload speed......................................................................................................
Upload: 28.33 Mbit/s
```

