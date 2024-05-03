---
layout: post
title: 'WireGuard组网'
subtitle: '异地组网'
date: 2024-04-01
author: Dave
cover: ''
tags: 网络
---


> 很多办公环境无ipv6环境，不方便连接家里的开发机。
>
> 电信的动态公网ip 成都区域 每月额外100块。阿里云轻量ECS 2C2G 3M带宽 一年99。

![](https://picc0lo-st.oss-cn-beijing.aliyuncs.com/img/wireguard.png)

## 前言

Wireguard组网前提: 有一台具备公网IP的主机。

本次组网以云服务器做server为例完成组网。

## 组件安装

click here [WireGuard Installation ](https://www.wireguard.com/install/)

```bash
# ubuntu
sudo apt-get install wireguard
# mac
brew install wireguard-tools
# openwrt
opkg install wireguard
```

## 服务端配置

> 云服务器: 阿里云轻量ECS 2C2G 3M带宽;
> 写的脚本这里识别cpu参数这里 被阿里云内部給屏蔽掉了 识别有问题 物理机上是ok的

```bash
➜  ~ sys_info
whoami  : root
hostname: xxxxx......
work_dir: /home/dave/
date: Tue Apr 01 23:13:28 AM CST 2024
================================================================
Host_info     :                                                                    
System        : Ubuntu 22.04.2 64 bit
Kernel        : 5.15.0-71-generic
CPU Info      : Intel(R) Xeon(R) Platinum 1 Pcs 1 Core/pcs
CPU Cores     : 2
CPU Arch      : x86_64 
CPU Load      : 1min:0.00  5min:  15min:
Memmory Total : 1.65GB
Memmory Used  : 1.56GB(94.56 %)
Memmory Free  : 91.70MB
Disk Info     : 0 disks detected 
*Disk isn't ready or permission denied, please check disk status or check permission! 
Partition Info: <partition,total_size,used,avaliable,uesd_ratio,mount_on_path>
/dev/vda3        40G  6.7G   31G  18% / 
```

### 1.环境初始化

```bash
# 开启ipv4流量转发
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
sysctl -p

# 创建并进入WireGuard文件夹：
mkdir -p /etc/wireguard && chmod 0777 /etc/wireguard
cd /etc/wireguard
umask 077
```

### 2.密钥生成

> 家庭网络设备有以下需要接入网络
>
> - 群晖DS916+
>
>   CPU性能孱弱，但承担主力文件存储及个人gitea托管、在线文档服务等
>
> - DIY Nas with ubuntu 
>
>   i5 8600k + 24G Mem 附文件存储、docker、开发
>
> - ubuntu desktop 
>
>   i7 9700k + 64G Mem + 2080Ti 主力开发机
>
> - OpenWrt
>
>   赛扬N2930 4C4T 4G Mem 承担路由及部分轻量docker
>
> - mbp 14 
>
>   m1pro + 32G 上班用，常在身边，必须配置
>
> - mbp 13.3 
>
>   i7 + 16G 老设备也配置上

```bash
# gen server key pairs
wg genkey | tee server_privatekey | wg pubkey > server_publickey
# gen client key pairs
wg genkey | tee client_privatekey_nas | wg pubkey > client_publickey_nas
wg genkey | tee client_privatekey_dev | wg pubkey > client_publickey_dev
wg genkey | tee client_privatekey_desktop | wg pubkey > client_publickey_desktop
wg genkey | tee client_privatekey_openwrt | wg pubkey > client_publickey_openwrt
wg genkey | tee client_privatekey_mac_iu | wg pubkey > client_publickey_mac_iu
wg genkey | tee client_privatekey_mac_au | wg pubkey > client_publickey_mac_au
```

必要情况，可以对这些密钥对备个份 。

### 3.服务端配置文件

```bash
vim /etc/wireguard/wg0.conf
```

```bash
[Interface]
PrivateKey = {...context...} # server privatekey
Address = 192.168.7.1 # server VIP

PostUp   = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -A FORWARD -o wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -D FORWARD -o wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
# 注意：eth0 为本机网卡名称

ListenPort = 888 # 自定义
DNS = 223.5.5.5  # 自定义
MTU = 1420       # 自定义
[Peer]
PublicKey =  {...context...}  # nas public key
AllowedIPs = 192.168.7.168/32

[Peer]
PublicKey =  {...context...}  # dev public key
AllowedIPs = 192.168.7.178/32

[Peer]
PublicKey =  {...context...}  # desktop public key
AllowedIPs = 192.168.7.188/32

[Peer]
PublicKey =  {...context...}  # openwrt public key
AllowedIPs = 192.168.7.198/32

[Peer]
PublicKey =  {...context...}  # mac_au public key
AllowedIPs = 192.168.7.201/32

[Peer]
PublicKey =  {...context...}  # mac_iu public key
AllowedIPs = 192.168.7.202/32
```

### 4.服务启动及验证

```bash
wg-quick up wg0   # wg启动
wg-quick down wg0 # wg关闭
wg # wg status

# 示例:可以在server上很清晰看到LAN的情况
☁  ~  sudo wg                       
interface: wg0
  public key: {...context...}
  private key: (hidden)
  listening port: 888
  
peer: {...context...}
  endpoint: {...context...}
  allowed ips: 192.168.7.168/32
  latest handshake: 48 minutes, 34 seconds ago
  transfer: 16.48 KiB received, 14.25 KiB sent

peer: {...context...}
  allowed ips: 192.168.7.188/32

```

### 5.端口开放

如果server端有防火墙，记得开放端口

```bash
sudo ufw allow 888
```

***注意：如果server是云服务器，务必记得进入安全组 新增规则开放端口。***

### 6.开机自启


## 客户端配置

### mac 

> 推荐直接wireguard客户端新增一个隧道，启用即可

```bash
[Interface]
PrivateKey = {...context...}  # client private key
Address = 192.168.7.168/32    # client vip
# ListenPort = 888            # client listen port
# MTU = 1420 

[Peer]
PublicKey = {...context...}   # server public key
AllowedIPs = 192.168.7.0/24   # VIP LAN Range/mask
Endpoint = {公网IP}:888        # server ip:port
```

### ubuntu

1.新增配置文件

```bash
 sudo vim /etc/wireguard/client.conf
```

```bash
[Interface]
PrivateKey = {...context...}  # client private key
Address = 192.168.7.168/32    # client vip
# ListenPort = 888            # client listen port
# MTU = 1420 

[Peer]
PublicKey = {...context...}   # server public key
AllowedIPs = 192.168.7.0/24   # VIP LAN Range/mask
Endpoint = {公网IP}:888        # server ip:port
```

2.服务启动

```bash
wg-quick up client   # wg启动
wg-quick down client # wg关闭
wg # wg status

# 示例 可以看到连接的server情况
☁  ~  sudo wg                                                                  
interface: client
  public key: {...context...}
  private key: (hidden)
  listening port: 39825

peer: {...context...}
  endpoint: {...context...}
  allowed ips: 192.168.7.0/24
  latest handshake: 1 hour, 30 seconds ago
  transfer: 12.52 KiB received, 16.48 KiB sent
```
3.开机自启
> 使用 systemd 来管理 WireGuard 服务

3.1 创建 systemd 服务单元文件
```bash
vim /etc/systemd/system/wireguard.service
```

填充内容
```toml
[Unit]
Description=WireGuard VPN service
After=network.target

[Service]
Type=oneshot
ExecStart=/usr/bin/wg-quick up client
ExecStop=/usr/bin/wg-quick down client
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```
3.2 启用 systemd 服务
```bash
sudo systemctl enable wireguard.service
## 示例
☁  ~  sudo systemctl enable wireguard.service
Created symlink /etc/systemd/system/multi-user.target.wants/wireguard.service → /etc/systemd/system/wireguard.service.
```

## 连通性测试

### 1.互ping

```bash
☁  ~  ping -c 4 192.168.7.1
PING 192.168.7.1 (192.168.7.1) 56(84) bytes of data.
64 bytes from 192.168.7.1: icmp_seq=1 ttl=64 time=34.1 ms
64 bytes from 192.168.7.1: icmp_seq=2 ttl=64 time=33.8 ms
64 bytes from 192.168.7.1: icmp_seq=3 ttl=64 time=33.5 ms
64 bytes from 192.168.7.1: icmp_seq=4 ttl=64 time=33.8 ms

--- 192.168.7.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 33.486/33.785/34.092/0.214 ms
```

更具体的可以同时使用tcpdump抓包看数据

```bash
☁  ~  tcpdump -i client
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on client, link-type RAW (Raw IP), capture size 262144 bytes
00:54:44.615420 IP ubuntu-desktop-xeon > 192.168.7.1: ICMP echo request, id 8, seq 1, length 64
00:54:44.649466 IP 192.168.7.1 > ubuntu-desktop-xeon: ICMP echo reply, id 8, seq 1, length 64
00:54:45.616724 IP ubuntu-desktop-xeon > 192.168.7.1: ICMP echo request, id 8, seq 2, length 64
00:54:45.650443 IP 192.168.7.1 > ubuntu-desktop-xeon: ICMP echo reply, id 8, seq 2, length 64
00:54:46.618664 IP ubuntu-desktop-xeon > 192.168.7.1: ICMP echo request, id 8, seq 3, length 64
00:54:46.652111 IP 192.168.7.1 > ubuntu-desktop-xeon: ICMP echo reply, id 8, seq 3, length 64
00:54:47.620331 IP ubuntu-desktop-xeon > 192.168.7.1: ICMP echo request, id 8, seq 4, length 64
00:54:47.654099 IP 192.168.7.1 > ubuntu-desktop-xeon: ICMP echo reply, id 8, seq 4, length 64
8 packets captured
8 packets received by filter
0 packets dropped by kernel
```



### 2.server端 wg 

```bash
☁  ~  sudo wg                       
interface: wg0
  public key: {...context...}
  private key: (hidden)
  listening port: 888
  
peer: {...context...}
  endpoint: {...context...}
  allowed ips: 192.168.7.168/32
  latest handshake: 48 minutes, 34 seconds ago
  transfer: 16.48 KiB received, 14.25 KiB sent

peer: {...context...}
  allowed ips: 192.168.7.188/32
```

> 至此，基本的组网结束。
