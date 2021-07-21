---
layout: post
title: 'Mac开发环境（2）：maven安装配置+idea配置'
subtitle: 'maven 3.3.9 + idea 2021'
date: 2021-07-09
author: Dave
cover: ''
tags: MAC开发环境 
---


**最前面**
- 上一节：[Mac开发环境（1）：Homebrew安装配置JDK MySql](https://picc0lo.top/2021/07/09/macForJavaDev-brew-jdk-mysql.html)
- 下一节：
- [Springboot 版本+ jdk 版本 + Maven 版本的匹配](https://www.cnblogs.com/jpfss/p/9037005.html)
- [Spring Boot Documention](https://spring.io/projects/spring-boot#learn)
>springboot 2.x +java 8 ，maven需要3.2以上，

>下面为自行下载maven并安装过程，也可`brew search maven` 查阅对应maven版本并进行安装，科大镜像源速度不错。

## maven基础安装

**1.下载**

>建议非必要情况并不要下载最新版本，此处我下载的3.3.9。（下载了最新3.8.1 结果不稳定 遂回退）
- [Downloading Apache Maven](http://maven.apache.org/download.cgi)
- [Index of /dist/maven/maven-3](https://archive.apache.org/dist/maven/maven-3/)

**2.解压**

>解压到/usr/local目录下，可自定义


**3.环境变量配置**

>**macOS 10.15 Catalina后zsh为macOS的默认shell**，之前为bash。以下按照shell做区分

- 3.1 默认bash的mac

  ~/ 目录下面有没有 .bash_profile文件
  >这是个默认隐藏的文件，Mac下`command` + `shift` + `.`可以切换显示/不显示隐藏文件。

  要是没有 咱们就去创建一个
  ```powershell
  touch ~/.bash_profile
  ```
  如果有就直接进入（或直接打开编辑）
  ```powershell
  vi ~/.bash_profile
  ```

  **添加maven路径**
  ```powershell
  PATH=$PATH:/usr/local/apache-maven-3.3.9/bin
  export PATH
  ```

  **让配置生效**
  ```powershell
  source ~/.bash_profile
  ```

  **验证安装成功与否**
  ```powershell
  mvn -v 
  ```

- 3.2 默认zsh的mac

  ~/ 目录下面有没有 .zshrc文件
  >这是个默认隐藏的文件，Mac下`command` + `shift` + `.`可以切换显示/不显示隐藏文件。

  要是没有 咱们就去创建一个
  ```powershell
  touch ~/.zshrc
  ```
  如果有就直接进入（或直接打开编辑）
  ```powershell
  vi ~/.zshrc
  ```

  **添加maven路径**
  ```powershell
  PATH=$PATH:/usr/local/apache-maven-3.3.9/bin
  export PATH
  ```

  **让配置生效**
  ```powershell
  source ~/.zshrc
  ```

  **验证安装成功与否**
  ```powershell
  mvn -v 
  ```

  ```zsh
  admin@admindeMacBook-Pro-2 ~ % mvn -v         
  Apache Maven 3.3.9 (bb52d8502b132ec0a5a3f4c09453c07478323dc5; 2015-11-11T00:41:47+08:00)
  Maven home: /usr/local/apache-maven-3.3.9
  Java version: 1.8.0_282, vendor: Oracle Corporation
  Java home: /usr/local/Cellar/openjdk@8/1.8.0+282/libexec/openjdk.jdk/Contents/Home/jre
  Default locale: zh_CN, platform encoding: UTF-8
  OS name: "mac os x", version: "11.3.1", arch: "x86_64", family: "mac"
  ```

**4.更换阿里源和修改本地仓库地址**

  maven的xml配置文件路径

  `apache-maven-3.3.9/conf/settings.xml`

- **更换阿里源**
  
  mirors中添加
  ```xml
      <!-- mirror
       | maven 国内阿里云镜像
       |
       |-->
      <mirror>
        <id>nexus-aliyun</id>
        <name>Nexus aliyun</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
        <mirrorOf>central</mirrorOf>
      </mirror>
  ```

- **添加私服**

  mirors中添加
  ```xml
      <!-- mirror
       | maven 私服镜像
       |
       |-->
      <mirror>
        <id>公司id</id>
        <mirrorOf>*</mirrorOf>
        <name>公司字段</name>
        <url>私服地址</url>
      </mirror>
  ```


- **私服添加后还需做部分配置**
    
  >以我司举例

  servers中添加
  ```xml
      <!-- servers
       | maven 私服认证接入
       |
       |-->
      <server>
          <id>公司id</id>
          <username>认证用户名</username>
          <password>密码</password>
      </server>
  ```

  profiles中添加对应设置

- **修改本地仓库**

  >**为了便利，我直接再maven解压目录下创建repository文件夹**

  本地仓库路径：`/usr/local/apache-maven-3.3.9/repository`

  ```xml
  <localRepository>/usr/local/apache-maven-3.3.9/repository </localRepository>
  ```

- **中央仓库下载文件到本地库**

  ```powershell
  mvn help:system
  ```

## maven基础结构

- **maven 项目目录结构**

  ```powershell
  $ MavenProject
  |-- pom.xml
  |-- src
  |   |-- main
  |   |   `-- java
  |   |   `-- resources
  |   `-- test
  |   |   `-- java
  |   |   `-- resources
  ```

  **说明：一般将java的功能代码，放在main/java下面，而测试代码放在test/java下，这样在运行时，maven才可以识别目录并进行编译。**

  | 路径     | 说明 |
  | ----------- | ----------- |
  |`src/main/java` |存放项目.java文件；|
  |`src/main/resources` |存放项目资源文件；|
  |`src/test/java` |存放测试类.java文件；|
  |`src/test/resources` |存放测试资源文件；|
  |`target` |项目输出目录；|
  |`pom.xml`|Maven核心文件（Project Object Model）|


## idea 对应配置


**Preferences --> Builder,Execution... --> Build Tools --> Maven**
去指定

- Maven home path
  >maven安装路径
- User setting file
  >maven setting路径
- Local repository
  >本地仓库路径


## maven卸载

>若出现版本不可控/不稳定/不适配，往往需要降低版本

- 找到maven安装目录，打开终端输入 `mvn -version`找到安装目录**maven home**
- 终端输入`sudo rm -rf [maven的路径]`

## maven 常用命令

- [Maven常用命令详解](https://www.linuxidc.com/Linux/2020-04/162861.htm)
  ```powershell
  1、mvn compile 编译,将Java 源程序编译成 class 字节码文件。
  2、mvn test 测试，并生成测试报告
  3、mvn clean 将以前编译得到的旧的 class 字节码文件删除
  4、mvn pakage 打包,动态 web工程打 war包，Java工程打 jar 包。
  5、mvn install 将项目生成 jar 包放在仓库中，以便别的模块调用
  ```
