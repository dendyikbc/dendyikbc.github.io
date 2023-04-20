---
layout: post
title: 'ubuntu 磁盘挂载与lvm'
subtitle: 'ubuntu server mount disk with lvm'
date: 2023-04-13
author: Dave
cover: ''
tags: Linux
---

> 背景：原ubuntu_server在一个256GB的SSD之上，最近开了几个docker后占用瞬间上来了，寻思就增加一块HDD。

## 硬盘挂载(标准分区形式)

> GPT 分区

1. 检测硬盘信息

   ```bash
   # 查看硬盘信息与分区情况
   sudo fdisk -l
   ## 新增硬盘: /dev/sda
   ```

2. fdisk分区

   ```bash
   sudo fdisk /dev/sda
   # 我只创建了一个分区，GPT,可自主决定是否多个分区以及分区格式、类型
   ```

3. 格式化分区

   ```bash
    sudo mkfs -t ext4 /dev/sda1
   ```

4. 挂载

   ```bash
   # 挂载点创建
   sudo mkdir /hdd
   # 挂载
   sudo mount /dev/sda1 /hdd
   # 查看挂载信息
   mount
   df -lh
   # e.g.
   ☁  ~  sudo df -lh        
   Filesystem                         Size  Used Avail Use% Mounted on
   /dev/mapper/ubuntu--vg-ubuntu--lv  207G   26G  172G  13% /
   /dev/sdb2                          2.0G  252M  1.6G  14% /boot
   /dev/sdb1                          1.1G  5.3M  1.1G   1% /boot/efi
   /dev/sda1                          3.6T   58G  3.4T   2% /hdd
   
   ```
   
5. 自动挂载

   ```bash
   ☁  ~  sudo vim /etc/fstab 
   # <file system> <mount point>   <type>  <options>       <dump>  <pass>
   # mount hdd on /data
   /dev/sda1 /hdd ext4 defaults 0 1
   
   # 也可以通过uuid的形式挂载
   /dev/disk/by-uuid/edd9-xxxxx-xxxxx-xxxx-xxxx-886 /hdd ext4 defaults 0 1
   
   # 添加完信息保存后，执行 sudo mount -a 命令，如果没有异常报错就ok
   sudo mount -a 
   
   # list uuid
   ☁  ~  sudo blkid -c /dev/null
   /dev/sda1: UUID="edd9-xxxxx-xxxxx-xxxx-xxxx-886" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="16226f92-edd9-xxxxx-xxxxx-xxxx-xxxx-886"
   
   ## 挂载参数解读
   # <file system>  分区设备文件名或UUID
   # <mount point>  挂载点
   # <type>         文件系统名称
   # <options>      挂载参数，挂载权限   
   # <dump>         指定分区是否被dump备份，0代表不备份，1代表每天备份，2代表不定期备份。
   # <pass>         指定分区是否被fsck检测，0代表不检测，其他数字代表检测的优先级，比如1的优先级比2高。根目录所在的分区的优先级为1，其他分区的优先级为大于或等于2
   ```

 

## 硬盘挂载(lvm)

> 挂载好了后发现，直接建立gpt分区做的挂载，没用上lvm，虽然后期也不太会再加硬盘，但是，我可以不用，我不能没有。
>
> lvm ext4

1. 检测硬盘信息

   ```bash
   # 查看硬盘信息与分区情况
   sudo fdisk -l
   ## /dev/sda
   ```

2. fdisk清楚分区并格式修改为lvm

   ```bash
   # 2.1 清楚分区与分区格式转换
   sudo fdisk /dev/sda
   # t   change a partition type
   # e.g. 
   t   change a partition type
   Aliases:
      linux          - 0FC63DAF-8483-4772-8E79-3D69D8477DE4
      swap           - 0657FD6D-A4AB-43C4-84E5-0933C84B4F4F
      home           - 933AC7E1-2EB4-4F13-B844-0E14E2AEF915
      uefi           - C12A7328-F81F-11D2-BA4B-00A0C93EC93B
      raid           - A19D880F-05FC-4D3B-A006-743F0F84911E
      lvm            - E6D6D379-F507-44C2-A23C-238F2A3DF928
   Partition type or alias (type L to list all): lvm
   Changed type of partition 'Linux filesystem' to 'Linux LVM'.
   
   
   # 2.2 依次创建pv vg lv
   ## pv
   ☁  ~  sudo pvcreate /dev/sda1 
     Physical volume "/dev/sda1" successfully created.
   ## vg
   ☁  ~  sudo vgcreate lvm_hdd /dev/sda1
     Volume group "lvm_hdd" successfully created 
   ## lv 
   #  pattern: lvcreate -L <size> -n <lvName> <VGName>
   ☁  ~  sudo lvcreate -L 3.6T -n hdd lvm_hdd          
     Rounding up size to full physical extent 3.60 TiB
     Logical volume "hdd" created.
   # e.g. lsblk
   ☁  ~  lsblk
   NAME                      MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
   sda                         8:0    0   3.6T  0 disk 
   └─sda1                      8:1    0   3.6T  0 part 
     └─lvm_hdd-hdd           253:1    0   3.6T  0 lvm  
   sdb                         8:16   0 223.6G  0 disk 
   ├─sdb1                      8:17   0     1G  0 part /boot/efi
   ├─sdb2                      8:18   0     2G  0 part /boot
   └─sdb3                      8:19   0 220.5G  0 part 
     └─ubuntu--vg-ubuntu--lv 253:0    0   210G  0 lvm  /
     
   ```

3. 格式化lv

   ```bash
   ## pattern:mkfs -t ext4 /dev/<vg_name>/<lv_name>
   ☁  ~  sudo mkfs -t ext4 /dev/lvm_hdd/hdd                               
   mke2fs 1.46.5 (30-Dec-2021)
   Creating filesystem with 966368256 4k blocks and 241598464 inodes
   Filesystem UUID: edd9e5d2-98e3-4dca-8bbd-1f0a4a43c886
   Superblock backups stored on blocks: 
           32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
           4096000, 7962624, 11239424, 20480000, 23887872, 71663616, 78675968, 
           102400000, 214990848, 512000000, 550731776, 644972544
   
   Allocating group tables: done                            
   Writing inode tables: done                            
   Creating journal (262144 blocks): done
   Writing superblocks and filesystem accounting information: done  
   ```

4. 挂载

   ```bash
   ## 正常挂载
   sudo mkdir /data                  
   sudo mount /dev/lvm_hdd/hdd /data
   
   # 查看挂载信息
   mount
   df -lh
   ## e.g.
   ☁  ~  sudo df -lh   
   Filesystem                         Size  Used Avail Use% Mounted on
   /dev/mapper/ubuntu--vg-ubuntu--lv  207G   26G  172G  13% /
   /dev/sdb2                          2.0G  254M  1.6G  14% /boot
   /dev/sdb1                          1.1G  5.3M  1.1G   1% /boot/efi
   /dev/mapper/lvm_hdd-hdd            3.6T   58G  3.4T   2% /data
   ```
   
5. 自动挂载

    ```bash
    ☁  ~  sudo vim /etc/fstab 
    # <file system> <mount point>   <type>  <options>       <dump>  <pass>
    # mount hdd on /data
    /dev/mapper/lvm_hdd-hdd  /data ext4 defaults 0 1
    
    # 也可以通过uuid的形式挂载
    /dev/disk/by-uuid/edd9-xxxxx-xxxxx-xxxx-xxxx-886 /data ext4 defaults 0 1
    
    ## 添加完信息保存后，执行 sudo mount -a 命令，如果没有异常报错就ok
    sudo mount -a 
    ```

    ```bash
    # list uuid 
    ☁  ~  sudo blkid -c /dev/null
    # 选mapper这个uuid
    /dev/mapper/lvm_hdd-hdd: UUID="edd9-xxxxx-xxxxx-xxxx-xxxx-886" BLOCK_SIZE="4096" TYPE="ext4"
    /dev/sda1: UUID="abcd-xxxxx-xxxxx-xxxx-xxxx-995" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="16226f92-edd9-xxxxx-xxxxx-xxxx-xxxx-995"
    ```

## lvm扩容(vg存在空闲)

> 256GB的ssd之前只开了一半空间

1. 查看存在的卷组

   ```bash
   ## list block
   ☁  ~  sudo lsblk   
   NAME                      MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
   sdb                         8:16   0 223.6G  0 disk 
   ├─sdb1                      8:17   0     1G  0 part /boot/efi
   ├─sdb2                      8:18   0     2G  0 part /boot
   └─sdb3                      8:19   0 220.5G  0 part 
     └─ubuntu--vg-ubuntu--lv 253:0    0   100G  0 lvm  /
   ## list vg
   ☁  ~  sudo vgdisplay
     --- Volume group ---
     VG Name               ubuntu-vg
     System ID             
     Format                lvm2
     Metadata Areas        1
     Metadata Sequence No  2
     VG Access             read/write
     VG Status             resizable
     MAX LV                0
     Cur LV                1
     Open LV               1
     Max PV                0
     Cur PV                1
     Act PV                1
     VG Size               <220.52 GiB
     PE Size               4.00 MiB
     Total PE              56452
     Alloc PE / Size       25600 / 100.00 GiB
     Free  PE / Size       30852 / <120.52 GiB
     VG UUID               keDVak-IAwn-ztCD-a5Gp-elxl-5iwG-bZWDeZ
   ```

2. lvextend 进行扩容

   ```bash
   ## 使用 lvextend 进行扩容 <target size>
   sudo lvextend -L 210G /dev/mapper/ubuntu--vg-ubuntu--lv
   ## 使用 resize2fs 更新
   sudo resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv
   ## list block
   ☁  ~  sudo lsblk                                             
   NAME                      MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
   sdb                         8:16   0 223.6G  0 disk 
   ├─sdb1                      8:17   0     1G  0 part /boot/efi
   ├─sdb2                      8:18   0     2G  0 part /boot
   └─sdb3                      8:19   0 220.5G  0 part 
     └─ubuntu--vg-ubuntu--lv 253:0    0   210G  0 lvm  /
   ```

   

## lvm扩容(新增磁盘扩容vg)

> 没实践过，目测流程ok

1. 查看存在的卷组

   ```bash
   ## list pv
   pvdisplay
   ## list vg
   vgdisplay
   ```

2. 新增磁盘创建pv

   ```bash
   pvcreate /dev/sdc1
   ```

3. 扩容vg

   ```bash
   # pattern: vgextend <vg_name> <pv_name>
   ## e.g. 模拟加3.6T 的sdc
   vgextend lvm_hdd /dev/sdc1
   ```

3. 扩容lv

   ```bash
   ## 使用 lvextend 进行扩容 <target size>
   sudo lvextend -L 7.2T /dev/mapper/lvm_hdd-hdd
   ## 或 +3.6T
   sudo lvextend -L +3.6T /dev/mapper/lvm_hdd-hdd
   ## 或 100% 
   sudo lvextend -L +100%FREE /dev/mapper/lvm_hdd-hdd
   
   ## (ext4)使用 resize2fs 更新
   sudo resize2fs /dev/mapper/lvm_hdd-hdd
   ## (xfs)使用 xfs_groupfs 更新
   sudo xfs_groupfs /dev/mapper/lvm_hdd-hdd
   ```

