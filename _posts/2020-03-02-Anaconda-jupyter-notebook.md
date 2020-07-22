---
layout: post
title: 'Anaconda安装 和 jupyter notebook默认工作目录修改'
subtitle: '这是windows10下的 '
date: 2020-03-02
author: Dave
tags: Anaconda 
---

![](https://raw.githubusercontent.com/dendyikbc/PicGoBed/master/imganaconda-jupyter-notebook.jpg)
> 安装Anaconda后经常需要更改jupyter notebook默认工作目录，趁这次疫情期间被关在家里，整理所得
>
> 电脑用户名:even

### part.01.anaconda安装
+ 下载安装
+ 环境变量配置
    ```python
    #在安装完成anaconda后就应该首先配置环境变量
    C:\ProgramData\Anaconda3\Scripts
    C:\ProgramData\Anaconda3\Library\bin
    ```
+ 验证安装成功与否
    ```python
    C:\Users\even>conda --version
    conda 4.5.11
    ```
+ 镜像设置
    ```python
    设置镜像网站可以加快下载速度。

    在命令行输入如下命令：

    conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/

    conda config --set show_channel_urls yes

    在C盘下可找到一个名为.condarc的文件，打开发现已经添加了指定的镜像网站，当然也可以直接修改.condarc文件：

    ```

### part.02.jupyter notebook更改默认路径
+ 新建新的工作目录`E:\pywd`

    在E盘新建文件夹"pywd"，路径`E:\pywd`，作为默认目录
+ 生成配置文件`jupyter_notebook_config.py`

    cmd命令行中输入`jupyter notebook --generate-config`，可在`C:\Users\even\.jupyter`目录下生成`jupyter_notebook_config.py`。
    
+ 修改工作目录

    找到配置文件并打开
    ```
    C:\Users\even\.jupyter\jupyter_notebook_config.py
    ```
    找到`c.NotebookApp.notebook_dir`填充新的工作目录
    ```python
    ## The directory to use for notebooks and kernels.
    c.NotebookApp.notebook_dir = 'E:\pywd'
    ```
+ 环境变量添加
    ```python
    #在安装完成anaconda后就应该首先配置环境变量
    C:\ProgramData\Anaconda3\Scripts
    C:\ProgramData\Anaconda3\Library\bin
    ```
+ 修改开始菜单中Jupyter Notebook的快捷方式(`最重要一步`)
    >操作流程（以win10为例）：程序→`Anaconda`→ `Jupyter Notebook`→ 右键→ 属性→ 快捷方式→ 去掉“目标”一项中后面的`"%USERPROFILE%"`。
