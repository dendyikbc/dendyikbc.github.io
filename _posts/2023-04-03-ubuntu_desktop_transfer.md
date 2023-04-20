---
layout: post
title: '记录一次ubuntu 全盘迁移&引导修复'
subtitle: 'ubuntu desktop 恢复'
date: 2023-04-03
author: Dave
cover: ''
tags: Linux
---

## 记录一次ubuntu 全盘迁移&引导修复

> 背景：原ubuntu放在一个128GB的sata SSD之上，近期在上面玩docker，磁盘占用拉满100%，重启后无法直接进入系统，遂ubuntu迁移新磁盘512GB NVME。
>
> 原磁盘在启动选项选择recovery模式下进行了docker相关内容清理已恢复正常。

### 前奏

UBUNTU 引导盘启动，Try Ubuntu，主要在 live CD模式中的终端下完成操作。

## 步骤

### 0.磁盘分区

对目标磁盘建立系统分区，并提前预留一个boot分区。

```bash
## boot分区
分区名：BIOS-Boot
分区卷标：bios_grub
分区大小：几百MB量级
```

### 1.全盘复制

```bash
sudo dd if=/dev/sdb2 of=/dev/nvme1n1p1
```

```bash
sudo watch -n 5 killall -USR1 dd
```

### 2.重建引导

> windows + ubuntu 双系统用户，此步骤需注意：确保win10的分区正确，具备完整的MSR 与 ESP 分区（主要是ESP分区），能够正确引导，否则重建grub后，无win10选项。

```bash
## config repo & install
sudo add-apt-repository ppa:yannubuntu/boot-repair
sudo apt-get update
sudo apt-get install -y boot-repair
## excute repair 
sudo boot-repair
```

### 3.重启，正常使用

双系统的小伙伴可以试着刷新一下grub信息，看是否有windows信息

```bash
sudo update-grub
```

> 此处没有windows efi信息，则grub 引导页面无对应 win bootmanager选项，因此：双系统用户需先确保windows ready后再进行grub修复。




### 我的踩坑

> 全盘拷贝 + 重建引导后， ubuntu ok，但是grub引导页无windows选项，反复多次尝试在ubuntu下多次操作发现，怀疑检测不到windows的efi文件信息所致，`fdisk -l`查看磁盘信息后确定windows efi信息丢失。主要原因：之前双系统做了，共用EFI分区于ubuntu所在磁盘，全盘拷贝只拷贝了数据就物理移除原ubuntu磁盘，因此原win10 缺少EFI引导。
> 确定根因后，执行修复步骤：
>
> - 使用windows PE，重建ESP分区，修复windows引导
> - 重启进入ubuntu，再次 boot repair，并check grub信息

#### windows PE下的 windows 引导修复

- 创建ESP分区，约300MB
- 使用引导修复，依次修复主引导与分区引导，并对ESP分区进行UEFI引导修复

#### ubutnu 下的 grub 引导修复

- `sudo fdisk -l` 查看windows对应磁盘是否具备ESP分区

```bash
Disk /dev/nvme1n1: 465.78 GiB, 500107862016 bytes, 976773168 sectors
Disk model: WDC WDS500G2B0C-00PXH0                  
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 8FDBC359-0D53-4119-8513-A523FF427D25

Device             Start       End   Sectors   Size Type
/dev/nvme1n1p1      2048 974612479 974610432 464.7G Microsoft basic data
/dev/nvme1n1p2 974612480 976773119   2160640     1G BIOS boot


Disk /dev/nvme0n1: 953.89 GiB, 1024209543168 bytes, 2000409264 sectors
Disk model: SAMSUNG MZVLB1T0HBLR-000L7              
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 48B6C7EA-4F07-4684-9686-9CAA8A7FA1E2

Device              Start        End    Sectors   Size Type
/dev/nvme0n1p1       2048      34815      32768    16M Microsoft reserved
/dev/nvme0n1p2      34816  377694207  377659392 180.1G Microsoft basic data
/dev/nvme0n1p3  377694208  709044223  331350016   158G Microsoft basic data
/dev/nvme0n1p4  709044224 1998310357 1289266134 614.8G Microsoft basic data
/dev/nvme0n1p5 1998311424 1998925823     614400   300M EFI System
```

- boot repair 

- check grub 信息

  > Found Windows Boot Manager

```bash
☁  Desktop  sudo update-grub
[sudo] password for dave: 
Sourcing file `/etc/default/grub'
Sourcing file `/etc/default/grub.d/init-select.cfg'
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-5.15.0-43-generic
Found initrd image: /boot/initrd.img-5.15.0-43-generic
Found linux image: /boot/vmlinuz-5.15.0-41-generic
Found initrd image: /boot/initrd.img-5.15.0-41-generic
Found Windows Boot Manager on /dev/nvme0n1p5@/EFI/Microsoft/Boot/bootmgfw.efi
Adding boot menu entry for UEFI Firmware Settings
done
```
