---
layout: post
title: 'ubuntu sudo权限配置与sudo免密'
subtitle: 'ubuntu sudo privileged'
date: 2023-03-02
author: Dave
cover: ''
tags: Linux
---

> 背景：sudo权限与sudo免密，sudo不想输密码。

## sudo权限配置（修改/etc/sudoers）

> /etc/sudoers

1. 切换root，修改 /etc/sudoers权限

   ```bash
   # 切换到 root 用户
   su root
   
   # 使用 ls -l /etc/sudoers查看权限：
   # 可以看到/etc/sudoers是只读的，可以修改文件权限，也可以修改以后wq!强制保存，这里使用前者
   -r--r----- 1 root root 3928 3月 10 16:35 /etc/sudoers
   
   # 修改 /etc/sudoers 权限
   chmod u+w /etc/sudoers. ##增加写的权限，修改文件后，记得将权限改回来
   ```

2. 修改/etc/sudoers

   ```bash
   sudo vim /etc/sudoers
   # User privilege specification
   root    ALL=(ALL:ALL) ALL
   piccolo ALL=(ALL:ALL) ALL ## sudo权限
   # Members of the admin group may gain root privileges
   %admin ALL=(ALL) ALL
   
   # Allow members of group sudo to execute any command
   %sudo   ALL=(ALL:ALL) ALL
   piccolo ALL=(ALL) NOPASSWD:ALL ## sudo免密
   ```

3. /etc/sudoers 还原权限

   ```bash
   # 将 /etc/sudoers 权限改回来
   chmod u-w /etc/sudoers
   ## 验证 切换回普通用户，使用 sudo 执行之前不能执行的操作
   su piccolo
   sudo cat /etc/shadow
   ```

## sudo visudo[推荐]

```bash
sudo visudo

# User privilege specification
root    ALL=(ALL:ALL) ALL
piccolo ALL=(ALL:ALL) ALL ## sudo权限
# Members of the admin group may gain root privileges
%admin ALL=(ALL) ALL

# Allow members of group sudo to execute any command
%sudo   ALL=(ALL:ALL) ALL
piccolo ALL=(ALL) NOPASSWD:ALL ## sudo免密

control + o 写入
control + x 退出
```

