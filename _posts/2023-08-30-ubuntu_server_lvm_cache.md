---
layout: post
title: 'ubuntu lvm cache'
subtitle: 'hdd turbo with cache'
date: 2023-08-30
author: Dave
cover: ''
tags: Linux
---

> 背景：ubuntu_server 上 hdd 有点慢了，找了块闲置的256GB sata ssd 加进去做缓存。
>
> hdd 希捷银河 4TB CMR
>
> vg: lvm_hdd
>
> lv:hdd

## lvm cache

> 注意：cache分区必须和data分区位于同一个vg。
>
> 这里我想要加速的是 lvm_hdd (vg_name)下的 hdd(lv_name)

1. 检测硬盘信息

   ```bash
   # 查看硬盘信息与分区情况
   sudo fdisk -l
   ## /dev/sda
   ```

2. 分区与cache配置

   ```bash
   # 2.1 清楚分区与分区格式转换
   sudo fdisk /dev/sda
   
   # n   add a new partition
   # e.g. 
   n   add a new partition
   Command (m for help): n
   Partition number (1-128, default 1): 
   First sector (34-468862094, default 2048): 
   Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-468862094, default 468862094): 
   
   Created a new partition 1 of type 'Linux filesystem' and of size 223.6 GiB.
   Partition #1 contains a vfat signature.
   
   Do you want to remove the signature? [Y]es/[N]o: Y
   The signature will be removed by a write command.
   
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
   
   
   # 2.2 pv vg lv
   ## pv
   ☁  ~  sudo pvcreate /dev/sda1
     Physical volume "/dev/sda1" successfully created.
   ## vg extend
   ☁  ~  sudo vgextend lvm_hdd /dev/sda1  
     Volume group "lvm_hdd" successfully extended
   ## lv 
   #  pattern: lvcreate -L <size> -n <LV_Name> <VG_Name> [option]<path>
   # 这里最好指定挂载点，并注意： meta 不低于 1/1000 cache 大小
   ☁  ~  sudo lvcreate -L 220G -n cache lvm_hdd /dev/sda1
     Logical volume "cache" created.
   ☁  ~  sudo lvcreate -L 3G -n meta lvm_hdd /dev/sda1
     Logical volume "meta" created.
   ## create cache-pool
   # pattern: lvconvert --type cache-pool --poolmetadata <VG_Name>/meta <VG_Name>/cache 
   ☁  ~  sudo lvconvert --type cache-pool --poolmetadata lvm_hdd/meta lvm_hdd/cache                                        
     Using 256.00 KiB chunk size instead of default 64.00 KiB, so cache pool has less than 1000000 chunks.
     WARNING: Converting lvm_hdd/cache and lvm_hdd/meta to cache pool's data and metadata volumes with metadata wiping.
     THIS WILL DESTROY CONTENT OF LOGICAL VOLUME (filesystem etc.)
   Do you really want to convert lvm_hdd/cache and lvm_hdd/meta? [y/n]: y
     Converted lvm_hdd/cache and lvm_hdd/meta to cache pool.
   ## cache-pool enable
   # pattern: lvconvert --type cache --cachepool <VG_Name>/cache --cachemode <cachemode_name> <VG_Name>/<data_lv_name>
   # 注意：cachemode有 writeback 和 writethrough 两种模式，默认为 writethrough。
   # writeback: 数据先写入缓存再写入磁盘（ASYNC）
   # writethrough: 写入数据会同步写入缓存和磁盘（SYNC）
     ☁  ~  sudo lvconvert --type cache --cachepool lvm_hdd/cache --cachemode writeback lvm_hdd/hdd 
   Do you want wipe existing metadata of cache pool lvm_hdd/cache? [y/n]: y
     Logical volume lvm_hdd/hdd is now cached.
     
   # e.g. lsblk
   # 挂载点都是/data
     ☁  ~  lsblk
   NAME                          MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
   sda                             8:0    0 223.6G  0 disk 
   └─sda1                          8:1    0 223.6G  0 part 
     ├─lvm_hdd-cache_cpool_cdata 253:3    0   220G  0 lvm  
     │ └─lvm_hdd-hdd             253:2    0   3.6T  0 lvm  /data
     └─lvm_hdd-cache_cpool_cmeta 253:4    0     3G  0 lvm  
       └─lvm_hdd-hdd             253:2    0   3.6T  0 lvm  /data
   sdc                             8:32   0   3.6T  0 disk 
   └─sdc1                          8:33   0   3.6T  0 part 
     └─lvm_hdd-hdd_corig         253:5    0   3.6T  0 lvm  
       └─lvm_hdd-hdd             253:2    0   3.6T  0 lvm  /data
     
   ```

3. fio test

   ```bash
   # fio
   alias fio_test_hdd="fio -filename=/data/piccolo/fio_test/test.file -direct=1 -iodepth 1 -thread -rw=randread -ioengine=psync -bs=16k -size=1G -numjobs=10 -runtime=60 -group_reporting -name=mytest" 
   alias fio_test_ssd="fio -filename=/home/piccolo/fio_test/test.file -direct=1 -iodepth 1 -thread -rw=randread -ioengine=psync -bs=16k -size=1G -numjobs=10 -runtime=60 -group_reporting -name=mytest"
   alias fio_test_nvme="fio -filename=/data_nvme/piccolo/fio_test/test.file -direct=1 -iodepth 1 -thread -rw=randread -ioengine=psync -bs=16k -size=1G -numjobs=10 -runtime=60 -group_reporting -name=mytest"
   ```

   

   ```bash
   # 256G sata ssd 系统盘
   ☁  /data  fio_test_ssd 
   mytest: (g=0): rw=randread, bs=(R) 16.0KiB-16.0KiB, (W) 16.0KiB-16.0KiB, (T) 16.0KiB-16.0KiB, ioengine=psync, iodepth=1
   ...
   fio-3.28
   Starting 10 threads
   Jobs: 10 (f=10): [r(10)][100.0%][r=283MiB/s][r=18.1k IOPS][eta 00m:00s]
   mytest: (groupid=0, jobs=10): err= 0: pid=4999: Wed Aug 30 23:16:37 2023
     read: IOPS=17.5k, BW=274MiB/s (287MB/s)(10.0GiB/37368msec)
       clat (usec): min=76, max=11057, avg=568.38, stdev=218.23
        lat (usec): min=76, max=11057, avg=568.49, stdev=218.23
       clat percentiles (usec):
        |  1.00th=[  247],  5.00th=[  322], 10.00th=[  367], 20.00th=[  424],
        | 30.00th=[  469], 40.00th=[  510], 50.00th=[  545], 60.00th=[  586],
        | 70.00th=[  627], 80.00th=[  685], 90.00th=[  766], 95.00th=[  848],
        | 99.00th=[ 1106], 99.50th=[ 1729], 99.90th=[ 2900], 99.95th=[ 3130],
        | 99.99th=[ 3490]
      bw (  KiB/s): min=268384, max=292992, per=100.00%, avg=280660.76, stdev=385.40, samples=740
      iops        : min=16774, max=18312, avg=17541.30, stdev=24.09, samples=740
     lat (usec)   : 100=0.01%, 250=1.06%, 500=36.88%, 750=50.36%, 1000=10.07%
     lat (msec)   : 2=1.28%, 4=0.34%, 10=0.01%, 20=0.01%
     cpu          : usr=0.50%, sys=2.16%, ctx=655408, majf=0, minf=40
     IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
        submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
        complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
        issued rwts: total=655360,0,0,0 short=0,0,0,0 dropped=0,0,0,0
        latency   : target=0, window=0, percentile=100.00%, depth=1
   
   Run status group 0 (all jobs):
      READ: bw=274MiB/s (287MB/s), 274MiB/s-274MiB/s (287MB/s-287MB/s), io=10.0GiB (10.7GB), run=37368-37368msec
   
   Disk stats (read/write):
       dm-1: ios=651467/22, merge=0/0, ticks=364752/16, in_queue=364768, util=99.79%, aggrios=655361/12, aggrmerge=0/10, aggrticks=367253/21, aggrin_queue=367292, aggrutil=99.76%
     sdb: ios=655361/12, merge=0/10, ticks=367253/21, in_queue=367292, util=99.76%
   
   # 256G sata ssd cache 加持下的 HDD
   # 可以看到，缓内速度就是ssd的速度。
   ☁  /data  fio_test_hdd 
   mytest: (g=0): rw=randread, bs=(R) 16.0KiB-16.0KiB, (W) 16.0KiB-16.0KiB, (T) 16.0KiB-16.0KiB, ioengine=psync, iodepth=1
   ...
   fio-3.28
   Starting 10 threads
   Jobs: 10 (f=10): [r(10)][97.5%][r=459MiB/s][r=29.4k IOPS][eta 00m:01s]
   mytest: (groupid=0, jobs=10): err= 0: pid=4821: Wed Aug 30 23:15:09 2023
     read: IOPS=16.4k, BW=257MiB/s (269MB/s)(10.0GiB/39866msec)
       clat (usec): min=55, max=554251, avg=605.81, stdev=5338.89
        lat (usec): min=55, max=554251, avg=605.89, stdev=5338.90
       clat percentiles (usec):
        |  1.00th=[   184],  5.00th=[   219], 10.00th=[   237], 20.00th=[   262],
        | 30.00th=[   281], 40.00th=[   306], 50.00th=[   326], 60.00th=[   351],
        | 70.00th=[   375], 80.00th=[   408], 90.00th=[   453], 95.00th=[   490],
        | 99.00th=[   627], 99.50th=[  8979], 99.90th=[ 80217], 99.95th=[119014],
        | 99.99th=[229639]
      bw (  KiB/s): min= 2048, max=476448, per=99.66%, avg=262139.21, stdev=22574.73, samples=789
      iops        : min=  128, max=29778, avg=16383.70, stdev=1410.92, samples=789
     lat (usec)   : 100=0.02%, 250=14.25%, 500=81.44%, 750=3.57%, 1000=0.04%
     lat (msec)   : 2=0.03%, 4=0.05%, 10=0.12%, 20=0.13%, 50=0.18%
     lat (msec)   : 100=0.10%, 250=0.06%, 500=0.01%, 750=0.01%
     cpu          : usr=0.35%, sys=2.07%, ctx=656090, majf=4, minf=44
     IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
        submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
        complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
        issued rwts: total=655360,0,0,0 short=0,0,0,0 dropped=0,0,0,0
        latency   : target=0, window=0, percentile=100.00%, depth=1
   
   Run status group 0 (all jobs):
      READ: bw=257MiB/s (269MB/s), 257MiB/s-257MiB/s (269MB/s-269MB/s), io=10.0GiB (10.7GB), run=39866-39866msec
   
   Disk stats (read/write):
       dm-2: ios=650444/5, merge=0/0, ticks=390364/120, in_queue=390484, util=99.81%, aggrios=219820/1374, aggrmerge=0/0, aggrticks=187702/840, aggrin_queue=188542, aggrutil=96.00%
       dm-4: ios=0/18, merge=0/0, ticks=0/8, in_queue=8, util=0.04%, aggrios=651087/4110, aggrmerge=0/8, aggrticks=213583/2318, aggrin_queue=215907, aggrutil=96.01%
     sda: ios=651087/4110, merge=0/8, ticks=213583/2318, in_queue=215907, util=96.01%
       dm-5: ios=8373/5, merge=0/0, ticks=350780/112, in_queue=350892, util=46.28%, aggrios=8373/4, aggrmerge=0/1, aggrticks=350916/117, aggrin_queue=351146, aggrutil=46.28%
     sdc: ios=8373/4, merge=0/1, ticks=350916/117, in_queue=351146, util=46.28%
     dm-3: ios=651087/4100, merge=0/0, ticks=212328/2400, in_queue=214728, util=96.00%
   
   # 512g nvme ssd
   ☁  /data  fio_test_nvme
   mytest: (g=0): rw=randread, bs=(R) 16.0KiB-16.0KiB, (W) 16.0KiB-16.0KiB, (T) 16.0KiB-16.0KiB, ioengine=psync, iodepth=1
   ...
   fio-3.28
   Starting 10 threads
   Jobs: 10 (f=10): [r(10)][100.0%][r=993MiB/s][r=63.6k IOPS][eta 00m:00s]
   mytest: (groupid=0, jobs=10): err= 0: pid=5080: Wed Aug 30 23:16:59 2023
     read: IOPS=53.5k, BW=837MiB/s (877MB/s)(10.0GiB/12240msec)
       clat (usec): min=111, max=31311, avg=185.48, stdev=536.53
        lat (usec): min=111, max=31311, avg=185.52, stdev=536.54
       clat percentiles (usec):
        |  1.00th=[  114],  5.00th=[  115], 10.00th=[  116], 20.00th=[  124],
        | 30.00th=[  126], 40.00th=[  130], 50.00th=[  141], 60.00th=[  149],
        | 70.00th=[  165], 80.00th=[  190], 90.00th=[  227], 95.00th=[  269],
        | 99.00th=[  437], 99.50th=[ 1663], 99.90th=[ 4228], 99.95th=[16712],
        | 99.99th=[20317]
      bw (  KiB/s): min=28128, max=1027584, per=100.00%, avg=857410.67, stdev=33138.21, samples=240
      iops        : min= 1758, max=64224, avg=53588.17, stdev=2071.14, samples=240
     lat (usec)   : 250=93.37%, 500=5.73%, 750=0.13%, 1000=0.07%
     lat (msec)   : 2=0.29%, 4=0.30%, 10=0.02%, 20=0.07%, 50=0.01%
     cpu          : usr=0.90%, sys=2.95%, ctx=655393, majf=0, minf=40
     IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
        submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
        complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
        issued rwts: total=655360,0,0,0 short=0,0,0,0 dropped=0,0,0,0
        latency   : target=0, window=0, percentile=100.00%, depth=1
   
   Run status group 0 (all jobs):
      READ: bw=837MiB/s (877MB/s), 837MiB/s-837MiB/s (877MB/s-877MB/s), io=10.0GiB (10.7GB), run=12240-12240msec
   
   Disk stats (read/write):
       dm-0: ios=651263/8, merge=0/0, ticks=117420/80, in_queue=117500, util=99.28%, aggrios=655361/6, aggrmerge=0/2, aggrticks=118576/82, aggrin_queue=118708, aggrutil=99.15%
     nvme0n1: ios=655361/6, merge=0/2, ticks=118576/82, in_queue=118708, util=99.15%
   ```

   
