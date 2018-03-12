# 使用Docker运行MongoDB和Redis

## 1. 课程说明

课程为纯动手实验教程，为了能说清楚实验中的一些操作会加入理论内容。理论内容我们不会写太多，已经有太多好文章了，会精选最值得读的文章推荐给你，在动手实践的同时扎实理论基础。

学习过程中遇到的所有问题，都可随时在[实验楼问答](https://www.shiyanlou.com/questions)中提出，与老师和同学一起交流。

实验环境中可以联网，不受实验楼网络限制。
## 2. 学习方法

实验楼的Docker课程包含15个实验，每个实验都提供详细的步骤和截图，适用于有一定Linux系统基础，想快速上手Docker的同学。

学习方法是多实践，多提问。启动实验后按照实验步骤逐步操作，同时理解每一步的详细内容，如果有任何疑问，随时在[实验楼问答](https://www.shiyanlou.com/questions/)中提问，实验楼团队和我都会及时回复大家的所有问题。

如果实验开始部分有推荐阅读的材料，请务必先阅读后再继续实验，理论知识是实践必要的基础。

## 3. 本节内容简介

在本实验里我们将通过完成 MongoDB 和 Redis 两个容器来学习Dockerfile及Docker的运行机制。

本节中，我们需要依次完成下面几项任务：

1. MongoDB 的安装及配置
2. Redis 的安装及配置
3. Dockerfile 的编写
4. 从 Dockerfile 构建镜像

本次实验的需求是完成 Dockerfile，通过 Dockerfile 创建 MongoDB 或 Redis 应用。Dockerhub上已经提供了官方的 MongoDB 和 Redis 镜像，本实验仅仅用于学习Dockerfile及Docker机制。

> MongoDB 是一个基于分布式文件存储的数据库。由C++语言编写。旨在为WEB应用提供可扩展的高性能数据存储解决方案。MongoDB 是一个介于关系数据库和非关系数据库之间的产品，是非关系数据库当中功能最丰富，最像关系数据库的。特点是高性能、易部署、易使用，存储数据非常方便。
> -来自百度百科


> Redis是一个开源的使用ANSI C语言编写、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API。
> -来自百度百科

## 4. 实验准备

### 4.1 实验分析

在本实验中，我们除了安装所需的核心服务外，还安装一个ssh服务提供便捷的管理。

为了提高`docker build`速度，我们直接使用阿里云的Ubuntu源。因此要在Dockerfile开始位置增加下面一句命令：

```
RUN echo "deb http://mirrors.aliyuncs.com/ubuntu/ trusty main universe" > /etc/apt/sources.list
```

### 4.2 创建 Dockerfile 文件

首先，需要创建一个目录来存放 Dockerfile 文件，目录名称可以任意，在目录里创建Dockerfile文件：

```
cd /home/shiyanlou
mkdir shiyanloumongodb shiyanlouredis
touch shiyanloumongodb/Dockerfile shiyanlouredis/Dockerfile
```

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1709timestamp1457511228797.png/wm)

使用vim/gedit编辑Dockerfile文件，根据我们的需求输入内容。

## 5. 实验一：Dockerfile 基本框架

### 5.1 基本框架

按照上一节学习的内容，我们先完成Dockerfile基本框架。

依次输入下面的基本框架内容：

```
# Version 0.1

# 基础镜像
FROM ubuntu:latest

# 维护者信息
MAINTAINER shiyanlou@shiyanlou.com

# 镜像操作命令
RUN echo "deb http://mirrors.aliyuncs.com/ubuntu/ trusty main universe" > /etc/apt/sources.list
RUN apt-get update && apt-get install -yqq supervisor && apt-get clean

# 容器启动命令
CMD ["supervisord"]
```

上面的Dockerfile创建了一个简单的镜像，并使用`Supervisord`启动服务。

### 5.2 安装SSH服务

首先安装所需要的软件包：

```
RUN apt-get install -yqq openssh-server openssh-client
```

创建运行目录：

```
RUN mkdir /var/run/sshd
```

设置root密码及允许root通过ssh登陆：

```
RUN echo 'root:shiyanlou' | chpasswd
RUN sed -i 's/PermitRootLogin without-password/PermitRootLogin yes/' /etc/ssh/sshd_config
```


## 6. 实验二：完成 MongoDB Dockerfile

在上述基本的架构下，我们根据需求可以增加新的内容到Dockerfile中，完成 MongoDB Dockerfile。

进入到 shiyanloumongodb的目录编辑 Dockerfile：

```
cd /home/shiyanlou/shiyanloumongodb/
vim Dockerfile
```

### 6.1 安装最新的MongoDB

在Ubuntu最新版本下安装MongoDB非常简单，参考 [MongoDB安装文档](https://docs.mongodb.org/manual/tutorial/install-mongodb-on-ubuntu/) 。有两种方法：

方法一是添加mongodb的源，执行 `apt-get install mongodb-org` 就可以安装下面的所有软件包：

1. mongodb-org-server：mongod 服务和配置文件
2. mongodb-org-mongos：mongos 服务
3. mongodb-org-shell：mongo shell工具
4. mongodb-org-tools：mongodump，mongoexport等工具

方法二是下载二进制包，然后解压出来就可以。

由于 MongoDB 的官网连接网速问题，我们使用第二种方案，并把最新的 MongoDB 的包放到阿里云上。

MongoDB 的下载链接如下：

```
http://labfile.oss-cn-hangzhou-internal.aliyuncs.com/courses/498/mongodb-linux-x86_64-ubuntu1404-3.2.3.tgz
```

我们完善 Dockerfile，使用 ADD 命令添加压缩包到镜像：

```
RUN mkdir -p /opt
ADD http://labfile.oss-cn-hangzhou-internal.aliyuncs.com/courses/498/mongodb-linux-x86_64-ubuntu1404-3.2.3.tgz /opt/mongodb.tar.gz
RUN cd /opt && tar zxvf mongodb.tar.gz && rm -rf mongodb.tar.gz
RUN mv /opt/mongodb-linux-x86_64-ubuntu1404-3.2.3 /opt/mongodb
```

创建 MongoDB 的数据存储目录：

```
RUN mkdir -p /data/db
```

将 MongoDB 的执行路径添加到环境变量里：

```
ENV PATH=/opt/mongodb/bin:$PATH
```

MongoDB 和 SSH 对外的端口：

```
EXPOSE 27017 22
```

### 6.2 编写`Supervisord`配置文件

添加`Supervisord`配置文件来启动mongodb和ssh，创建文件`/home/shiyanlou/shiyanloumongodb/supervisord.conf`，添加以下内容：

```
[supervisord]
nodaemon=true

[program:mongodb]
command=/opt/mongodb/bin/mongod

[program:ssh]
command=/usr/sbin/sshd -D
```

Dockerfile中增加向镜像内拷贝该文件的命令：

```
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
```

### 6.3 完整的 Dockerfile

```
# Version 0.1

# 基础镜像
FROM ubuntu:latest

# 维护者信息
MAINTAINER shiyanlou@shiyanlou.com

# 镜像操作命令
RUN echo "deb http://mirrors.aliyuncs.com/ubuntu/ trusty main universe" > /etc/apt/sources.list
RUN apt-get -yqq update && apt-get install -yqq supervisor
RUN apt-get install -yqq openssh-server openssh-client

RUN mkdir /var/run/sshd
RUN echo 'root:shiyanlou' | chpasswd
RUN sed -i 's/PermitRootLogin without-password/PermitRootLogin yes/' /etc/ssh/sshd_config

RUN mkdir -p /opt
ADD http://labfile.oss-cn-hangzhou-internal.aliyuncs.com/courses/498/mongodb-linux-x86_64-ubuntu1404-3.2.3.tgz /opt/mongodb.tar.gz
RUN cd /opt && tar zxvf mongodb.tar.gz && rm -rf mongodb.tar.gz
RUN mv /opt/mongodb-linux-x86_64-ubuntu1404-3.2.3 /opt/mongodb

RUN mkdir -p /data/db

ENV PATH=/opt/mongodb/bin:$PATH

COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf

EXPOSE 27017 22

# 容器启动命令
CMD ["supervisord"]
```


操作演示视频
`@
https://labfile.oss-cn-hangzhou.aliyuncs.com/courses/498/video/7-6.flv
@`

## 7. 实验三：完成 Redis Dockerfile

在上述基本的架构下，我们根据需求可以增加新的内容到Dockerfile中，完成 Redis Dockerfile。

进入到 shiyanlouredis 的目录编辑 Dockerfile：

```
cd /home/shiyanlou/shiyanlouredis/
vim Dockerfile
```

### 7.1 安装 Redis

由于 MongoDB 中我们已经学习了如何通过二进制压缩包安装最新版本MongoDB的过程，在此安装 Redis 我们直接使用 Ubuntu 源中默认的 Redis 版本。

安装方法非常简单：

```
RUN apt-get install redis-server
```

添加对外的端口号：

```
EXPOSE 27017 22
```

### 7.2 编写`Supervisord`配置文件

添加`Supervisord`配置文件来启动 redis-server 和 ssh，创建文件`/home/shiyanlou/shiyanlouredis/supervisord.conf`，添加以下内容：

```
[supervisord]
nodaemon=true

[program:redis]
command=/usr/bin/redis-server

[program:ssh]
command=/usr/sbin/sshd -D
```

Dockerfile中增加向镜像内拷贝该文件的命令：

```
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
```

### 7.3 完整的 Dockerfile

```
# Version 0.1

# 基础镜像
FROM ubuntu:latest

# 维护者信息
MAINTAINER shiyanlou@shiyanlou.com

# 镜像操作命令
RUN echo "deb http://mirrors.aliyuncs.com/ubuntu/ trusty main universe" > /etc/apt/sources.list
RUN apt-get -yqq update && apt-get install -yqq supervisor redis-server
RUN apt-get install -yqq openssh-server openssh-client

RUN mkdir /var/run/sshd
RUN echo 'root:shiyanlou' | chpasswd
RUN sed -i 's/PermitRootLogin without-password/PermitRootLogin yes/' /etc/ssh/sshd_config

COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf

EXPOSE 6379 22

# 容器启动命令
CMD ["supervisord"]
```

操作演示视频
`@
https://labfile.oss-cn-hangzhou.aliyuncs.com/courses/498/video/7-7.flv
@`

## 8. 实验五：从 Dockerfile 创建镜像

### 8.1 创建 MongoDB 镜像

进入到`/home/shiyanlou/shiyanloumongodb/`目录，执行创建命令。

`docker build` 执行创建，`-t`参数指定镜像名称：

```
docker build -t shiyanloumongodb:0.1 /home/shiyanlou/shiyanloumongodb/
```

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1709timestamp1457511284792.png/wm)

`docker images` 查看创建的新镜像已经出现在了镜像列表中：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1709timestamp1457511294074.png/wm)

由该镜像创建新的容器mongodb：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1709timestamp1457511328107.png/wm)

上述`docker ps`命令的输出可以看到 MongoDB 的端口号已经被自动映射到了本地的 32768 端口，后续步骤我们对 MongoDB 是否启动进行测试。

打开 Xfce 终端中输入下面的命令连接 mongodb 容器中的服务：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1709timestamp1457511456998.png/wm)

### 8.2 创建 Redis 镜像

进入到`/home/shiyanlou/shiyanlouredis/`目录，执行创建命令。

`docker build` 执行创建，`-t`参数指定镜像名称：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1709timestamp1457511491952.png/wm)

`docker images` 查看创建的新镜像已经出现在了镜像列表中：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1709timestamp1457511499237.png/wm)

由该镜像创建新的容器redis：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1709timestamp1457511508152.png/wm)

上述`docker ps`命令的输出可以看到 redis 的端口号已经被自动映射到了本地的 32769 端口，SSH服务的端口号也映射到了 32770 端口。

打开 Xfce 终端中输入下面的命令连接 redis 容器中的 ssh 和 redis 服务：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1709timestamp1457511585507.png/wm)

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1709timestamp1457511592966.png/wm)

操作演示视频
`@
https://labfile.oss-cn-hangzhou.aliyuncs.com/courses/498/video/7-8.flv
@`

## 8. 总结

本节实验中我们学习了以下内容，任何不清楚的地方欢迎到[实验楼问答](https://www.shiyanlou.com/questions)与我们交流：

1. MongoDB 的安装
2. Redis 的安装
3. Dockerfile 的编写
4. 从 Dockerfile 构建镜像

请务必保证自己能够动手完成整个实验，只看文字很简单，真正操作的时候会遇到各种各样的问题，解决问题的过程才是收获的过程。