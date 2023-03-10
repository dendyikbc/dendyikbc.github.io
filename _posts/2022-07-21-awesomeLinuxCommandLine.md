---
layout: post
title: 'Linux Command Line'
subtitle: 'linux命令行使用笔记'
date: 2022-07-21
author: Dave
cover: ''
tags: Linux


---

**最前面**

>整理Linux命令行

## 关机/重启/注销

```bash
## 关机
shutdown -h now		## 即刻关机
shutdown -h 10		## 10分钟后关机
shutdown -h 11:00	## 11：00关机
shutdown -h +10		## 预定时间关机（10分钟后）
shutdown -c		## 取消指定时间关机
init 0			## ⽴刻关机
telinit 0		## 关机
poweroff		## ⽴刻关机
halt			## 关机

## 重启
shutdown -r now		## 重启
shutdown -r 10		## 10分钟之后重启
shutdown -r 11:00	## 定时重启
reboot			## 重启
init 6			## 重启

## 注销
logout			## 注销
sync			## buff数据同步到磁盘
```



## 平台/设备/系统信息

```bash
## 平台信息：系统内核、版本、CPU、GPU
uname -a			## 查看内核/OS/CPU信息
uname -r			## 查看内核版本
uname -m			## 查看处理器架构
arch				## 查看处理器架构
cat /proc/version		## 查看linux版本信息
cat /proc/cpuinfo		## 查看CPU信息
lscpu				## 查看CPU信息
cat /etc/*release		## 查看系统信息

## 设备信息：内存、磁盘、USB设备、PCI设备
lsusb -tv			## 查看系统USB设备信息
lspci -tv			## 查看系统PCI设备信息
grep MemTotal /proc/meminfo	## 查看内存总量
grep MemFree /proc/meminfo	## 查看空闲内存量
free -m				## 查看内存⽤量和交换区⽤量

## 系统信息：用户、组、环境变量、时区
hostname			## 查看计算机名
users				## 查看系统存在哪些用户
w				## 查看当前登录系统的⽤户
who				## 查看当前登录系统的⽤户
who am i			## 查看登录时的⽤户名
whoami				## 查看当前⽤户名
groups				## 查看当前用户所在组
cat /etc/group			## 查看组文件
env				## 查看系统的环境变量
lsmod				## 查看已加载的系统模块
cat /proc/interrupts		## 查看中断
cat /proc/loadavg		## 查看系统负载
uptime				## 查看系统运⾏时间、⽤户数、负载
date				## 查看系统⽇期时间
cal 2022			## 显示2022⽇历表
```



## 磁盘分区与挂载

```bash
## 磁盘分区
fdisk -l			## 查看所有磁盘分区
swapon -s			## 查看所有交换分区
df -h				## 查看磁盘使⽤情况及挂载点
df -hl				## 同上
du -sh /dir			## 查看指定某个⽬录的⼤⼩
du -sk * | sort -rn		## 从⾼到低依次显示⽂件和⽬录⼤⼩

## 磁盘挂载/卸载
mount /dev/hda2 /mnt/hda2			## 挂载hda2盘
mount -t ntfs /dev/sdc1 /mnt/usbhd1		## 指定⽂件系统类型挂载（如ntfs）
mount -o loop xxx.iso /mnt/cdrom		## 挂载iso⽂件
umount -v /dev/sda1				## 通过设备名卸载
umount -v /mnt/mymnt				## 通过挂载点卸载
fuser -km /mnt/hda1				## 强制卸载(慎⽤)

```

## 性能监测

```bash
## CPU
sar -u 1 10	## 查询cpu使⽤情况（1秒⼀次，共10次）
## %user：用于表示用户模式下消耗的 CPU 时间的比例；
## %nice：通过 nice 改变了进程调度优先级的进程，在用户模式下消耗的 CPU 时间的比例；
## %system：系统模式下消耗的 CPU 时间的比例；
## %iowait：CPU 等待磁盘 I/O 导致空闲状态消耗的时间比例；
## %steal：利用 Xen 等操作系统虚拟化技术，等待其它虚拟 CPU 计算占用的时间比例；
## %idle：CPU 空闲时间比例。

iostat		## 查看io读写/cpu使⽤情况
vmstat 1 20	## 每1秒采⼀次系统状态，采20次
top		## 动态显示cpu/内存/进程等情况
## GPU

## 磁盘IO
sar -d 1 10	## 查询磁盘性能

## tps：每秒从物理磁盘 I/O 的次数。注意，多个逻辑请求会被合并为一个 I/O 磁盘请求，一次传输的大小是不确定的；
## rd_sec/s：每秒读扇区的次数；
## wr_sec/s：每秒写扇区的次数；
## avgrq-sz：平均每次设备 I/O 操作的数据大小（扇区）；
## avgqu-sz：磁盘请求队列的平均长度；
## await：从请求磁盘操作到系统完成处理，每次请求的平均消耗时间，包括请求队列等待时间，单位是毫秒（1 秒=1000 毫秒）；
## svctm：系统处理每次请求的平均时间，不包括在请求队列中消耗的时间；
## util：I/O 请求占 CPU 的百分比，比率越大，说明越饱和


## 网络
```



## ⽤户和⽤户组

```bash
w				## 查看活动⽤户
who				## 查看活动⽤户
users				## 存在哪些用户
useradd	<user_name>		## 创建⽤户
userdel -r <user_name>		## 删除⽤户
usermod -g <group_name> <user_name>			## 修改⽤户的组
usermod -aG <group_name> <user_name>			## 将⽤户添加到组
usermod -s /bin/zsh -d /home/dave –g <group_name> dave	## 修改⽤户dave的登录Shell、主⽬录以及⽤户组
groups							## 当前用户所在组
groups Dave						## 查看⽤户dave所在的组
groupadd <group_name>					## 创建⽤户组
groupdel <group_name>					## 删除⽤户组
groupmod -n <new_name> <old_name>			## 重命名⽤户组
su - <user_name>					## 切换用户
passwd							## 修改当前用户密码
passwd <user_name>					## 修改指定⽤户的⼝令
id <user_name>						## 查看指定⽤户信息
last							## 查看⽤户登录⽇志
crontab -l						## 查看当前⽤户的计划任务
cut -d: -f1 /etc/passwd					## 查看系统所有⽤户
cut -d: -f1 /etc/group					## 查看系统所有组
```



## ⽹络

```bash
## 网络
ifconfig						## 查看⽹络接⼝属性
ifconfig | grep inet					## 查看ipv4地址
ifconfig eth0						## 查看某⽹卡的配置
ifconfig eth0 192.168.1.1 netmask 255.255.255.0		## 配置ip地址 注：ubuntu已启netplan配置静态地址
route -n						## 查看路由表
netstat -lntp						## 查看所有监听端⼝
netstat -antp						## 查看已经建⽴的TCP连接
netstat -lutp						## 查看TCP/UDP的状态信息
ifup eth0						## 启⽤eth0⽹络设备
ifdown eth0						## 禁⽤eth0⽹络设备
iptables -L						## 查看iptables规则
dhclient eth0							## 以dhcp模式启⽤eth0
route add -net 0/0 gw Gateway_IP				## 配置默认⽹关
route add -net 192.168.0.0 netmask 255.255.0.0 gw 192.168.1.1	## 配置静态路由到达⽹络’192.168.0.0/16’
route del 0/0 gw Gateway_IP					## 删除静态路由
hostname			## 查看主机名
host www.baidu.com		## 解析主机名
nslookup www.baidu.com		## 查询DNS记录，查看域名解析是否正常
```



## 系统服务与进程

```bash
## 系统服务
chkconfig –list				## 列出系统服务(chkconfig 是Redhat下的程序) 
					## 注：Ubuntu 中 sysv-rc-conf 替代 chkconfig
service <service_name> status		## 查看某个服务
service <service_name> start		## 启动某个服务
service <service_name> stop		## 终⽌某个服务
service <service_name> restart		## 重启某个服务
systemctl status <service_name>		## 查看某个服务
systemctl start <service_name>		## 启动某个服务
systemctl stop <service_name>		## 终⽌某个服务
systemctl restart <service_name>	## 重启某个服务
systemctl enable <service_name>		## 关闭⾃启动
systemctl disable <service_name>	## 关闭⾃启动

## 进程
ps -ef					## 查看所有进程
ps -ef | grep <process_name>		## 过滤出你需要的进程
kill -s <process_name>			## kill指定名称的进程
kill -s pid				## kill指定pid的进程
top					## 实时显示进程状态
vmstat 1 20				## 每1秒采⼀次系统状态，采20次
iostat					## iostat
netstat -anp|grep 11000 		##查看端口占用
lsof -i tcp:11000 			##查看端口占用
ps -ef | grep pid 			## 查看线程对应进程
```



## ⽂件和⽬录操作

```bash
## 目录/文件 创建、切换、查看

mkdir <dir>				## 创建⽬录
mkdir <dir1> <dir2> 			## 同时创建两个⽬录
mkdir -p /tmp/dir1/dir2			## 创建⽬录树
cd <path>				## 进⼊某个⽬录
cd ..					## 回上级⽬录
cd ../..				## 回上两级⽬录
cd					## 进个⼈主⽬录
cd -					## 回上⼀步所在⽬录
pwd					## 显示当前路径

## ls 展示的是文件内容的大小。
## du 展示的是磁盘空间占用量。
tree					## 查看⽂件和⽬录的树形结构
ls					## 查看⽂件⽬录列表
ls -F					## 查看⽬录中内容（显示是⽂件还是⽬录）
ls -l					## 查看⽂件和⽬录的详情列表
ls -lh					## 查看⽂件和⽬录的详情列表（增强⽂件⼤⼩易读性）
ls -a					## 查看隐藏⽂件
ls -lSr					## 查看⽂件和⽬录列表（以⽂件⼤⼩升序查看）
du -sh    				## 当前文件夹总大小，-s选项表示计算总和，-h选项表示以恰当的K/M/G单位展示
du -sh *  				## 当前文件夹各子文件夹/子文件大小
du -csh *  				## 当前文件夹各子文件夹/子文件大小，-c 增加汇总语义
du -ch *.gz 				## 在当前文件夹下的所有子文件内容中，匹配 *.gz 并total汇总size
du -sk *|sort -nr 			## 降序查看当前文件夹下第一级的大小 
					## sort -n选项，只按数值进行比较，只比较数字，所以此时du不能用配置-h														
du -ah .|sort -hr			## 降序查看当前文件夹和其子文件夹下的大小 
					## sort -h选项，先优先比较单位（G>M>K），然后再对数值进行比较；
du --max-depth=0 -h .			## 效果同du -sh，只显示当前文件夹总大小
du --max-depth=1 -h 			## 显示当前文件夹大小基础之上，增加第一级的文件夹大小
du --max-depth=1 -ah  			## 在上述基础上，-a选项让 du 输出包括文件夹和文件在内的完整统计信息


## 目录/文件 删除
rm -f <file>				## 删除’file’⽂件
rmdir <dir>				## 删除’dir’⽬录
rm -rf <dir>				## 删除’dir’⽬录和其内容
rm -rf <dir1> <dir2>			## 同时删除两个⽬录及其内容

## 目录/文件 移动、复制
mv <old_dir> <new_dir>			## 重命名/移动⽬录
cp <file> <file2>			## 复制⽂件
cp dir/* .				## 复制dir⽬录下的所有⽂件⾄当前⽬录
cp -a <src_dir> <dest_dir>		## 复制⽬录
cp -a /tmp/dir .			## 复制⼀个⽬录⾄当前⽬录

##-r 参数可以将 packageA 下的所有文件拷贝到 packageB 中

cp -r /home/packageA/* /home/cp/packageB/
将一个文件夹复制到另一个文件夹下，以下实例 packageA 文件会拷贝到 packageB 中：

cp -r /home/packageA /home/packageB
运行命令之后 packageB 文件夹下就有 packageA 文件夹了。

下面四个命令结果相同，都是递归拷贝 packageA 文件及其任意层的结构到 packageB 中:

cp -r /home/packageA /home/packageB
cp -r /home/packageA /home/packageB/
cp -r /home/packageA/ /home/packageB
cp -r /home/packageA/ /home/packageB/
下面两个命令结果相同，都是不拷贝 packageA 文件，只递归拷贝其任意层的子结构到 packageB 中:

cp -r packageA/* packageB
cp -r packageA/* packageB/

## 目录/文件 搜索定位
find / -name file1		## 从跟⽬录开始搜索⽂件/⽬录
find / -user user1		## 搜索⽤户user1的⽂件/⽬录
find /dir -name *.bin		## 在⽬录/dir中搜带有.bin后缀的⽂件
## 文件查找 支持通配符
find . -name 'compassdb.ini*'

locate <keyword>			## 快速定位⽂件
locate *.mp4				## 寻找.mp4结尾的⽂件
whereis <keyword>			## 显示某⼆进制⽂件/可执⾏⽂件的路径
which <keyword>				## 查找系统⽬录下某的⼆进制⽂件




## 目录/文件 权限控制
chmod ugo+rwx <dir>					## 设置⽬录所有者(u)、群组(g)及其他⼈(o)的读（r）写(w)执⾏(x)权限
chmod go-rwx <dir>					## 移除群组(g)与其他⼈(o)对⽬录的读写执⾏权限
chown <user> <file>					## 改变⽂件的所有者属性
chown :<group> <file>				        ## 改变⽂件群组属性
chgrp <group> <file>				        ## 改变⽂件群组属性
chown <user>:<group> <file>	                        ## 改变⽂件的所有者和群组
chown -R <user> <dir>				        ## 改变⽬录的所有者属性

## e.g.
chown :root *       					## 当前层级所有文件和目录群组修改为root
chown -R dave:dave bk_ram/ 			        ## 修改文件夹及其子文件夹/内容所有权限
chown -R <user>:<group> </PATH/TO/FILE>                 ## 修改文件夹及其子文件夹/内容所有权限

## 文件 link
ln -s <file> <link>	                                ## 创建指向⽂件/⽬录的软链接
ln <file> <lnk>	                                        ## 创建指向⽂件/⽬录的物理链接
```



## ⽂件查看和处理

```bash
## 文件查看
cat file1						## 查看⽂件内容
cat -n file1						## 查看内容并标示⾏数
tac file1						## 从最后⼀⾏开始反看⽂件内容
more file1
less file1						## 类似more命令，但允许反向操作
head -2 file1						## 查看⽂件前两⾏
tail -2 file1						## 查看⽂件后两⾏
tail -f /log/msg					## 实时查看添加到⽂件中的内容

grep codesheep hello.txt				## 在⽂件hello.txt中查找关键词codesheep
grep ^sheep hello.txt					## 在⽂件hello.txt中查找以sheep开头的内容
grep [0-9] hello.txt					## 选择hello.txt⽂件中所有包含数字的⾏
sed ‘s/s1/s2/g’ hello.txt				## 将hello.txt⽂件中的s1替换成s2
sed ‘/^$/d’ hello.txt					## 从hello.txt⽂件中删除所有空⽩⾏
sed ‘/ *#/d; /^$/d’ hello.txt				## 从hello.txt⽂件中删除所有注释和空⽩⾏
sed -e ‘1d’ hello.txt					## 从⽂件hello.txt 中排除第⼀⾏
sed -n ‘/s1/p’ hello.txt				## 查看只包含关键词”s1”的⾏
sed -e ‘s/ *$//’ hello.txt				## 删除每⼀⾏最后的空⽩字符
sed -e ‘s/s1//g’ hello.txt				## 从⽂档中只删除词汇s1并保留剩余全部
sed -n ‘1,5p;5q’ hello.txt				## 查看从第⼀⾏到第5⾏内容
sed -n ‘5p;5q’ hello.txt				## 查看第5⾏
paste file1 file2					## 合并两个⽂件或两栏的内容
paste -d ‘+’ file1 file2				## 合并两个⽂件或两栏的内容，中间⽤”+”区分
sort file1 file2					## 排序两个⽂件的内容
comm -1 file1 file2					## ⽐较两个⽂件的内容(去除’file1’所含内容)
comm -2 file1 file2					## ⽐较两个⽂件的内容(去除’file2’所含内容
comm -3 file1 file2					## ⽐较两个⽂件的内容(去除两⽂件共有部分)
```



## 打包、解压、压缩

```bash
## 打包与解压
## 1）.zip
zip <xxx.zip> <file>				## 压缩⾄zip包
zip -r <xxx.zip> <file1> <file2> <dir1>		## 将多个⽂件+⽬录压成zip包
unzip <xxx.zip>					## 解压zip包

## 2）.tar
tar -cvf <xxx.tar> <file>			## 创建⾮压缩tar包
tar -cvf <xxx.tar> <file1> <file2> <dir1>	## 将多个⽂件+⽬录打tar包
tar -tf <xxx.tar>				## 查看tar包的内容
tar -xvf <xxx.tar>				## 解压tar包
tar -xvf <xxx.tar> -C /dir			## 将tar包解压⾄指定⽬录

## 3）.bz2
tar -cvfj <xxx.tar.bz2> <dir>	## 创建bz2压缩包
tar -jxvf <xxx.tar.bz2>		## 解压bz2压缩包
bunzip2 <xxx.bz2>		## 解压bz2压缩包

## 4）.gz
tar -cvfz <xxx.tar.gz> <dir>	## 创建gzip压缩包
tar -zxvf <xxx.tar.gz>		## 解压gzip压缩包
gunzip <xxx.gz>			## 解压gzip压缩包
gzip -d <xxx.gz> 		## 解压gzip压缩包

## 压缩文件
bzip2 <file_name>		## 压缩⽂件
gzip <file_name>		## 基础压缩（不保留源文件）压缩文件为 源文件.gz
gzip -9 <file_name>		## 最⼤程度压缩
## e.g. 保留源文件压缩
gzip -c file_name.txt > file_name.txt.gz ## -c压缩重定向（保留源文件）压缩文件为 源文件.gz
```



## RPM包管理命令

```bash
rpm -qa				## 查看已安装的rpm包
rpm -q <pkg_name>		## 查询某个rpm包
rpm -q –whatprovides <xxx>	## 显示xxx功能是由哪个包提供的
rpm -q –whatrequires <xxx>	## 显示xxx功能被哪个程序包依赖的
rpm -q –changelog <xxx>		## 显示xxx包的更改记录
rpm -qi <pkg_name>		## 查看⼀个包的详细信息
rpm -qd <pkg_name>		## 查询⼀个包所提供的⽂档
rpm -qc <pkg_name>		## 查看已安装rpm包提供的配置⽂件
rpm -ql <pkg_name>		## 查看⼀个包安装了哪些⽂件
rpm -qf <filename>		## 查看某个⽂件属于哪个包
rpm -qR <pkg_name>		## 查询包的依赖关系
rpm -ivh <xxx.rpm>		## 安装rpm包
rpm -ivh –test <xxx.rpm>	## 测试安装rpm包
rpm -ivh –nodeps <xxx.rpm>	## 安装rpm包时忽略依赖关系
rpm -e <xxx>			## 卸载程序包
rpm -Fvh <pkg_name>		## 升级确定已安装的rpm包
rpm -Uvh <pkg_name>		## 升级rpm包(若未安装则会安装)
rpm -V <pkg_name>		## RPM包详细信息校验
```



## YUM包管理命令

```bash
yum repolist enabled			## 显示可⽤的源仓库
yum search <pkg_name>			## 搜索软件包
yum install <pkg_name>			## 下载并安装软件包
yum install –downloadonly <pkg_name>	## 只下载不安装
yum list				## 显示所有程序包
yum list installed			## 查看当前系统已安装包
yum list updates			## 查看可以更新的包列表
yum check-update			## 查看可升级的软件包
yum update				## 更新所有软件包
yum update <pkg_name>			## 升级指定软件包
yum deplist <pkg_name>			## 列出软件包依赖关系
yum remove <pkg_name>			## 删除软件包
yum clean all				## 清除缓存
yum clean packages			## 清除缓存的软件包
yum clean headers			## 清除缓存的header
```



## DPKG包管理命令

```bash
dpkg -c <xxx.deb>			## 列出deb包的内容
dpkg -i <xxx.deb>			## 安装/更新deb包
dpkg -r <pkg_name>			## 移除deb包
dpkg -P <pkg_name>			## 移除deb包(不保留配置)
dpkg -l					## 查看系统中已安装deb包
dpkg -l <pkg_name>			## 显示包的⼤致信息
dpkg -L <pkg_name>			## 查看deb包安装的⽂件
dpkg -s <pkg_name>			## 查看包的详细信息
dpkg –unpack <xxx.deb>			## 解开deb包的内容
```



## APT软件⼯具

```bash
apt-cache search <pkg_name>		## 搜索程序包
apt-cache show <pkg_name>		## 获取包的概览信息
apt-get install <pkg_name>		## 安装/升级软件包
apt-get remove <pkg_name>		## 卸载软件（不包括配置）
					## 描述：仅移除与该package相关联的所有二进制文件，不移除与之相关联的配置文件、数据文件、所依赖的包
apt-get -s remove <pkg_name>		## 模拟卸载软件（不包括配置）一般用于卸载package前，查看移除过程中会移除哪些内容
apt-get purge <pkg_name>		## 卸载软件（包括配置）
					## 描述：移除与该package相关联的文件，包括二进制文件、全局配置文件，不移除所依赖的包、位于用户目录中的相关联的配置文件或数据文件
aptitude remove <pkg_name>		## 移除该package及其所依赖但是不被系统中其他包依赖的包
aptitude purge <pkg_name>               ## 移除该package及其所依赖但是不被系统中其他包依赖的包，及其配置文件
apt-get autoremove			## 移除当前系统中的所有孤立包，具体指那些曾经被其他包所依赖，但是现在不被任何包依赖的包。
apt-get update				## 更新包索引信息
apt-get upgrade				## 更新已安装软件包
apt-get clean				## 清理缓存
```



