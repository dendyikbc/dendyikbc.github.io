---
layout: post
title: '记录一次更换系统盘重载lvm数据盘'
subtitle: ''
date: 2024-03-11
author: Dave
cover: ''
tags: Linux
---

> 背景：系统盘出现io err，手动修复后过了一段时间又出现io err。还好提前对数据盘 系统盘做了物理分割，系统盘随意换数据没影响。
>
> 没办法，先更换一个系统盘。但数据盘 仍然在原机器上。数据盘组了lvm+ssd lvm cache且数据存储了部分内容，不想重新组。

### lvm activate & mount

#### lvm display

先看一下，lvm信息正常，只是新换的系统盘没挂载。

```bash
☁  ~  sudo lvdisplay          
  --- Logical volume ---
  LV Path                /dev/lvm_hdd/hdd
  LV Name                hdd
  VG Name                lvm_hdd
  LV UUID                48R90L-xxxxxxxxxxxxxxxx
  LV Write Access        read/write
  LV Creation host, time ubuntuserver, 2023-04-18 01:14:58 +0800
  LV Status              NOT available
  LV Size                3.60 TiB
  Current LE             943719
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
   
  --- Logical volume ---
  LV Path                /dev/lvm_ssd/ssd
  LV Name                ssd
  VG Name                lvm_ssd
  LV UUID                nVts1Y-xxxxxxxxxxxx
  LV Write Access        read/write
  LV Creation host, time ubuntuserver, 2023-04-23 22:28:12 +0800
  LV Status              NOT available
  LV Size                465.00 GiB
  Current LE             119040
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
```

#### lvm activate

```
☁  ~  sudo vgchange -ay
  1 logical volume(s) in volume group "lvm_hdd" now active
  1 logical volume(s) in volume group "lvm_ssd" now active
```

```
☁  ~  lsblk | grep -v loop
NAME                          MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda                             8:0    0 223.6G  0 disk 
└─sda1                          8:1    0 223.6G  0 part 
  ├─lvm_hdd-cache_cpool_cdata 253:0    0   220G  0 lvm  
  │ └─lvm_hdd-hdd             253:3    0   3.6T  0 lvm  
  └─lvm_hdd-cache_cpool_cmeta 253:1    0     3G  0 lvm  
    └─lvm_hdd-hdd             253:3    0   3.6T  0 lvm  
sdb                             8:16   0 119.2G  0 disk 
├─sdb1                          8:17   0   512M  0 part /boot/efi
└─sdb2                          8:18   0 118.8G  0 part /
sdc                             8:32   0   3.7T  0 disk 
└─sdc1                          8:33   0   3.7T  0 part 
  └─lvm_hdd-hdd_corig         253:2    0   3.6T  0 lvm  
    └─lvm_hdd-hdd             253:3    0   3.6T  0 lvm  
nvme0n1                       259:0    0 465.8G  0 disk 
└─nvme0n1p1                   259:1    0 465.8G  0 part 
  └─lvm_ssd-ssd               253:4    0   465G  0 lvm  
```

#### lvm mount

```
☁  ~  sudo mount /dev/lvm_hdd/hdd /data       
☁  ~  sudo mount /dev/lvm_ssd/ssd /data_nvme
```



剩下的，再设置一下，开机自动挂载；以及恢复之前的SMB配置即可。
