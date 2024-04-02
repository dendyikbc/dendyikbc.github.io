---
layout: post
title: 'docker 使用指南'
subtitle: 'docker usage info'
date: 2022-12-26
author: Dave
cover: ''
tags: 通用文档
---

# docker 使用指南

> 简单记一下

![img](https://logos-marcas.com/wp-content/uploads/2021/03/Docker-Logo.png)

## 安装

> [docker-io docker-engine docker-ce的关系](https://www.jianshu.com/p/38ed1ef20b1c)

```
## ubuntu
sudo apt-get update
sudo apt-get install docker.io
## ubuntu docker path
/var/lib/docker

## mac
brew install docker

## check version
docker version        
## run hello-world test
docker container run hello-world  
```

## 使用

> 大部分基础看这里 -> [RUNOOB: Docker 容器使用](https://www.runoob.com/docker/docker-container-usage.html)

### 镜像

> docker login

```
☁  ~  docker login registry.gitlab.xxxx.com      
Authenticating with existing credentials...
Login Succeeded
```

#### CLI

```
# image
docker search <image_name>  ## search image
docker pull ubuntu          ## pull ubuntu latest version
docker pull ubuntu:22.04    ## pull ubuntu 22.04
docker rmi hello-world      ## delete hello-world image
docker images               ## list images
docker images --format '{{.Size}}\t{{.Repository}}:{{.Tag}}' | sort -hr ## 镜像大小排序
docker tag <image_id> <username>/<repository name>:<tag>  ## add tag
docker image prune -a       ## 删除无用镜像
docker image prune -a -f    ##强制删除镜像
docker rmi -f  `docker images | grep '<none>' | awk '{print $3}'`   ## 删除名称或标签为none的镜像
## e.g.
☁  ~  docker tag feb5d9fea6a5 piccolo/hello-world:test
☁  ~  docker images                                   
REPOSITORY            TAG       IMAGE ID       CREATED         SIZE
hello-world           latest    feb5d9fea6a5   14 months ago   13.3kB
piccolo/hello-world   test      feb5d9fea6a5   14 months ago   13.3kB
```

#### 镜像导出 导入

##### export

```
docker export <container_id> > <xxx.tar>       ## export xxx.tar to local path
```

##### import

```
docker import <xxx.tar> test/ubuntu:v1         ## import
docker import <path or url>                    ## import from path or url
```

#### 镜像创建

##### 1）docker commit

> 从容器创建

```
docker commit -m="update image" -a="piccolo" <container_id> <target_image_name>  ## update image & create
## -m commit
## -a author
## -p 生成过程中停止容器的运行

## e.g.
docker commit -m="update image" -a="piccolo" 35a5217c1faa image_dev/ubuntu_focal_with_tools:v2
```



##### 2）docker build

> 基于Dockerfile构建新镜像 
>
> 1. create DockerFile
> 2. Build

create DockerFile

```
mkdir static_image & cd static_image  ## 创建 build environment(build context)
vim Dockerfile                        ## 创建 Dockerfile
## Dockerfile content e.g.

## start----------------------------------------------------------------------------------------
# build image 
# Version: 0.0.1
FROM ubuntu:20.04 as buildstage
MAINTAINER piccolo "your@gmail.com"
## change source
RUN sed -i 's#http://archive.ubuntu.com/#http://mirrors.tuna.tsinghua.edu.cn/#' /etc/apt/sources.list;
RUN apt-get update
## install tools
RUN apt-get install -y sudo git cmake software-properties-common g++ curl zsh vim --fix-missing
RUN apt-get install -y nginx
RUN echo 'Hi, I am in your container' > /usr/share/nginx/html/index.html
## port expose
EXPOSE 80
EXPOSE 8000
EXPOSE 9000
## ----------------------------------------------------------------------------------------end
```

> 默认情况下，RUN指令会在shell里使用命令包装器/bin/sh -c 来执行。若在一个不支持shell的平台上运行或者不希望在shell中运行（比如避免shell字符串篡改），也可以使用exec格式的RUN指令，通过一个数组的方式指定要运行的命令和传递给该命令的每个参数：
>
> ```
> RUN ["apt-get", "install", "-y", "nginx"]
> ```
>
> EXPOSE指令是告诉Docker该容器内的应用程序将会使用容器的指定端口。这并不意味着可以自动访问任意容器运行中服务的端口。出于安全的原因，Docker并不会自动打开该端口，而是需要你在使用docker run运行容器时来指定需要打开哪些端口。
>
> 可以指定多个EXPOSE指令来向外部公开多个端口，Docker也使用EXPOSE指令来帮助将多个容器链接。
>
> **构建缓存：**
>
> 在上面执行构建镜像的过程中，我们发现当执行apt-get update时，返回Using cache。Docker会将之前的镜像层看做缓存，因为在安装nginx前并没有做其他的修改，因此Docker会将之前构建时创建的镜像当做缓存并作为新的开始点。然后，有些时候需要确保构建过程不会使用缓存。可以使用docker build 的 --no-cache标志。



build

```
## in build context
docker build -t="test/static_web" .
## -t选项为新镜像设置了仓库和名称，这里仓库为test，镜像名为static_web。建议为自己的镜像设置合适的名字方便以后追踪和管理
```



#### 减小镜像的体积

> [云原生实验室_Docker 镜像制作教程：减小镜像体积](https://icloudnative.io/posts/docker-images-part1-reducing-image-size/)
>
> - 多阶段构建
>
> - 使用经典的基础镜像
>
> - `COPY --from` 使用绝对路径
>
>   实践
>
> ```
>  ## start----------------------------------------------------------------------------------------
>   # build image 
>   # Version: 0.0.1
>   FROM ubuntu:20.04 as buildstage
>   MAINTAINER piccolo "your@gmail.com"
>   ## change source
>   RUN sed -i 's#http://archive.ubuntu.com/#http://mirrors.tuna.tsinghua.edu.cn/#' /etc/apt/sources.list;
>   RUN apt-get update
>   ## install tools
>   RUN apt-get install -y sudo git cmake software-properties-common g++ curl zsh vim --fix-missing
>   ## set data zone
>   RUN export DEBIAN_FRONTEND=noninteractive \
>       && apt-get update \
>       && apt-get install -y tzdata \
>       && ln -fs /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
>       && dpkg-reconfigure --frontend noninteractive tzdata
>   ## port expose
>   EXPOSE 80
>   EXPOSE 8000
>   EXPOSE 9000
>   ## ----------------------------------------------------------------------------------------end
> 
>   FROM gcc AS mybuildstage
>     COPY hello.c .
>   RUN gcc -o hello hello.c
>   FROM ubuntu
>   COPY --from=mybuildstage hello .
>   CMD ["./hello"]
> ```



### 容器

#### CLI

```
# container
docker ps																		   ## ls
docker ps -a																   ## ls all
docker ps --format "table {{.ID}}\t{{.Names}}\t{{.Image}}"
docker start <container_id>/<container_name>	 ## start
docker stop <container_id>/<container_name>		 ## stop,参数 -t：关闭容器的限时，如果超时未能关闭则用kill强制关闭，默认值10s，这个时间用于容器的自己保存状态
docker kill <container_id>/<container_name>		 ## kill
docker restart <container_id>/<container_name> ## restart,-t：关闭容器的限时，如果超时未能关闭则用kill强制关闭，默认值10s，这个时间用于容器的自己保存状态
docker rm -f <container_id>/<container_name>	 ## delete
docker rm `docker ps -a | grep Exited | awk '{print $1}'`## 删除异常停止的docker容器
# run
docker run <options> <images> <shell>
# e.g.
## basic docker run
docker run -itd --name=container_name ubuntu:latest /bin/bash
## 端口映射 -p <host_port>:<container_port>
# -d 参数默认不进入容器，想要进入容器使用docker exec
#-i 交互式操作
#-t 终端
#-d 后台运行
docker run -itd -p 8888:8888 -p 8890:8890 ubuntu /bin/bash
docker run -itd -p 8888-8890:8888-8890 ubuntu /bin/bash

## 文件映射 -v <host_path>:<container_path>
## host网络
## 特权容器
## -w
docker run -itd --net=host --privileged=true --name=ubuntu_dev --rm -v /data/piccolo/docker_swap:/home/swap -w /home/swap ubuntu:20.04 /bin/bash

## --rm 前台容器退出时清理数据，不能与 -d 同时使用（或者说同时使用没有意义）
docker run -it --name=ubuntu_dev --rm -v /data/piccolo/docker_swap:/home/swap -w /home/swap ubuntu:20.04 /bin/bash

# exec 
docker exec <options> <container_id>/<container_name> <shell>		##从这个容器退出，容器不会停止
## e.g.
docker exec -it 243c32535da7 /bin/bash	## 若，推荐使用 docker exec 


# 语言设置（docker容器中输出中文会报出ascill编码错误，需要修改系统编码）
export LC_CTYPE=C.UTF-8

# install common tools
apt-get install sudo git cmake software-properties-common g++ curl zsh vim wget curl
apt-get install -y git g++ make libssl-dev libgflags-dev libprotobuf-dev libprotoc-dev protobuf-compiler libleveldb-dev
```

#### 与宿主机 文件交互

```
# 宿主机->容器
docker cp <from_path> <container>:<to_path>
# 容器->宿主机
docker cp <container>:<from_path> <to_path>
```

#### 运行信息查看

```
# 动态查看日志
docker logs -f <container>
# 详细信息
docker inspect <container>
## 查看docker 容器运行路径 
☁  ~  docker inspect dev | grep "HostnamePath"
"HostnamePath":"/var/lib/docker/containers/83192ceb422dee967b249cf7a4cf062995c12506e8b9652480c8dbaf91acd034/hostname",
# 端口
docker port <container>
# 统计
docker stats <container>
```

### docker 路径迁移

> /var/lib/docker是docker默认路径，往往机器系统盘并不大，多人使用时docker路径需要迁移到更大的数据盘去。
>
> /var/lib/docker 目录迁移/data/docker

查看路径

```
sudo docker info | grep "Docker Root Dir"
☁  ~  sudo docker info | grep "Docker Root Dir"
 Docker Root Dir: /var/lib/docker
```

1.停止docker

```
systemctl stop docker
```

2.路径创建[并迁移]

```
sudo mkdir -p /data/docker
# 如果不管之前的镜像与容器 可以不需迁移
sudo rsync -avz /var/lib/docker /data/docker/
```

3.配置修改

```
sudo vim /etc/docker/daemon.json
{
    "data-root": "/data/docker"
}
```

4.重启

```
 ☁  ~  systemctl start docker
```

5.测试

```
☁  ~  sudo docker info | grep "Docker Root Dir"
 Docker Root Dir: /data/docker
☁  ~  docker run -itd --privileged=true --name=db_mysql  ubuntu/mysql /bin/zsh
21165c24c315a4ff6cbbdbee625410034230fdabfd13ca325a25bf3566f94ab9
☁  ~  docker inspect db_mysql | grep "HostnamePath"                         
        "HostnamePath": "/data/docker/containers/21165c24c315a4ff6cbbdbee625410034230fdabfd13ca325a25bf3566f94ab9/hostname",
   
```



### docker 资源清理

```
# 删除异常停止的容器
docker rm `docker ps -a | grep Exited | awk '{print $1}'`
# 删除无用容器
docker container prune -f
# 删除无用镜像
docker image prune -a -f
## 删除24h前的镜像
docker image prune -a --filter "until=24h"
## 删除所有 匹配 *1.0* 的镜像
docker images | grep '1.0' | awk '{print $3}' | xargs docker rmi
## 删除名称或标签为none的镜像
docker rmi -f  `docker images | grep '<none>' | awk '{print $3}'`
```



```
# 谨慎使用以下
# 清理未使用的 Docker 资源
docker system prune --all --force --volumes
# 先停止所有容器再清理资源
docker stop $(docker container ls -a -q) && docker system prune --all --force --volumes
```