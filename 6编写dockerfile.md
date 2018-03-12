# 编写Dockerfile

## 1. 课程说明

课程为纯动手实验教程，为了能说清楚实验中的一些操作会加入理论内容。理论内容我们不会写太多，已经有太多好文章了，会精选最值得读的文章推荐给你，在动手实践的同时扎实理论基础。

学习过程中遇到的所有问题，都可随时在[实验楼问答](https://www.shiyanlou.com/questions)中提出，与老师和同学一起交流。

实验环境中可以联网，不受实验楼网络限制。

## 2. 学习方法

实验楼的Docker课程包含15个实验，每个实验都提供详细的步骤和截图，适用于有一定Linux系统基础，想快速上手Docker的同学。

学习方法是多实践，多提问。启动实验后按照实验步骤逐步操作，同时理解每一步的详细内容，如果有任何疑问，随时在[实验楼问答](https://www.shiyanlou.com/questions/)中提问，实验楼团队和我都会及时回复大家的所有问题。

如果实验开始部分有推荐阅读的材料，请务必先阅读后再继续实验，理论知识是实践必要的基础。

## 3. 本节内容简介

在前面的实验中我们多次用到的 Dockerfile，在本实验里我们将通过完成一个实例来学习Dockerfile的编写。

本节中，我们需要依次完成下面几项任务：

1. Dockerfile 基本框架
2. Dockerfile 编写常用命令
3. 从 Dockerfile 构建镜像

本次实验的需求是完成一个Dockerfile，通过该Dockerfile创建一个Web应用，该web应用为apache托管的一个静态页面网站，换句话说，我们写一个Dockerfile，用来创建一个实验楼公司的网站应用，就是[http://www.simplecloud.cn](http://www.simplecloud.cn)这个站点。这个站点是纯静态的页面，我们也可以直接下载得到。

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1708timestamp1457407965754.png/wm)

## 4. 实验准备

### 4.1 创建 Dockerfile 文件

首先，需要创建一个目录来存放 Dockerfile 文件，目录名称可以任意，在目录里创建Dockerfile文件：

```
cd /home/shiyanlou
mkdir shiyanloutest
cd shiyanloutest
touch Dockerfile
```

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1708timestamp1457353212911.png/wm)

使用vim/gedit编辑Dockerfile文件，根据我们的需求输入内容。

## 5. 实验一：Dockerfile 基本框架

Dockerfile一般包含下面几个部分：

1. 基础镜像：以哪个镜像作为基础进行制作，用法是`FROM 基础镜像名称`
2. 维护者信息：需要写下该Dockerfile编写人的姓名或邮箱，用法是`MANITAINER 名字/邮箱`
3. 镜像操作命令：对基础镜像要进行的改造命令，比如安装新的软件，进行哪些特殊配置等，常见的是`RUN 命令`
4. 容器启动命令：当基于该镜像的容器启动时需要执行哪些命令，常见的是`CMD 命令`或`ENTRYPOINT`

在本节实验中，我们依次先把这四项信息填入文档。Dockerfile中的`#`标志后面为注释，可以不用写，另外实验楼的环境不支持中文输入，比较可惜。

依次输入下面的基本框架内容：

```
# Version 0.1

# 基础镜像
FROM ubuntu:latest

# 维护者信息
MAINTAINER shiyanlou@shiyanlou.com

# 镜像操作命令
RUN apt-get -yqq update && apt-get install -yqq apache2 && apt-get clean

# 容器启动命令
CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```

上面的Dockerfile非常简单，创建了一个apache的镜像。包含了最基本的四项信息。

其中`FROM`指定基础镜像，如果镜像名称中没有制定TAG，默认为`latest`。`RUN`命令默认使用`/bin/sh` Shell执行，默认为root权限。如果命令过长需要换行，需要在行末尾加`\`。`CMD`命令也是默认在`/bin/sh`中执行，并且默认只能有一条，如果是多条`CMD`命令则只有最后一条执行。用户也可以在`docker run`命令创建容器时指定新的`CMD`命令来覆盖Dockerfile里的`CMD`。

这个Dockerfile已经可以使用`docker build`创建新镜像了，先构建一个版本shiyanloutest:0.1：

```
cd /home/shiyanlou/shiyanloutest
docker build -t shiyanloutest:0.1 .
```

构建需要安装apache2，会花几分钟，最后查看新创建的镜像：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1708timestamp1457403299356.png/wm)

使用该镜像创建容器web1，将容器中的端口80映射到本地80端口：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1708timestamp1457403440808.png/wm)

使用实验环境桌面上的firefox浏览器打开`localhost`进行测试，查看是否apache已运行：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1708timestamp1457403470883.png/wm)

操作演示视频
`@
https://labfile.oss-cn-hangzhou.aliyuncs.com/courses/498/video/6-5.flv
@`

## 6. 实验二：Dockerfile 编写常用命令

在上述基本的架构下，我们根据需求可以增加新的内容到Dockerfile中。后续的扩展操作都需要放置在Dockerfile的镜像操作部分。其中部分命令在本实验中并不会用到，但需要有所了解。

### 6.1 指定容器运行的用户

该用户将作为后续的RUN命令执行的用户。这个命令本实验不需要，但在一些需要指定用户来运行的应用部署时非常关键，比如提供hadoop服务的容器通常会使用`hadoop`用户来启动服务。

命令使用方式，例如使用shiyanlou用户来执行后续命令：

```
USER shiyanlou
```

### 6.2 指定后续命令的执行目录

由于我们需要运行的是一个静态网站，将启动后的工作目录切换到`/var/www/html`目录：

```
WORKDIR /var/www/html
```

### 6.3 对外连接端口号

由于内部服务会启动Web服务，我们需要把对应的80端口暴露出来，可以提供给容器间互联使用，可以使用`EXPOSE`命令。

在镜像操作部分增加下面一句：

```
EXPOSE 80
```

### 6.4 设置容器主机名

`ENV`命令能够对容器内的环境变量进行设置，我们使用该命令设置由该镜像创建的容器的主机名为`shiyanloutest`，向Dockerfile中增加下面一句：

```
ENV HOSTNAME shiyanloutest
```

### 6.5 向镜像中增加文件

向镜像中添加文件有两种命令：`COPY` 和 `ADD`。

`COPY`命令可以复制本地文件夹到镜像中：

```
COPY simplecloudsite /var/www/html
```

`ADD` 命令支持添加本地的tar压缩包到容器中指定目录，压缩包会被自动解压为目录，也可以自动下载URL并拷贝到镜像，例如：

```
ADD html.tar /var/www
ADD http://www.shiyanlou.com/html.tar /var/www
```

根据实验需求，我们把需要的一个网站放到镜像里，需要把一个`tar`包添加到apache的`/var/www`目录下，因此选择使用 `ADD`命令：

```
ADD html.tar /var/www
```

### 6.6 CMD 与 ENTRYPOINT

`ENTRYPOINT` 容器启动后执行的命令，让容器执行表现的像一个可执行程序一样，与`CMD`的区别是不可以被`docker run`覆盖，会把`docker run`后面的参数当作传递给`ENTRYPOINT`指令的参数。Dockerfile中只能指定一个`ENTRYPOINT`，如果指定了很多，只有最后一个有效。`docker run`命令的`-entrypoint`参数可以把指定的参数继续传递给`ENTRYPOINT`。

在本实验中两种方式都可以选择。

### 6.7 挂载数据卷

将apache访问的日志数据存储到宿主机可以访问的数据卷中：

```
VOLUME ["/var/log/apche2"]
```

### 6.8 设置容器内的环境变量

使用`ENV`设置一些apache启动的环境变量：

```
ENV APACHE_RUN_USER www-data
ENV APACHE_RUN_GROUP www-data
ENV APACHE_LOG_DIR /var/log/apche2
ENV APACHE_PID_FILE /var/run/apache2.pid
ENV APACHE_RUN_DIR /var/run/apache2
ENV APACHE_LOCK_DIR /var/lock/apche2
```

### 6.9 使用 `Supervisord`

CMD如果只有一个命令，那如果我们需要运行多个服务怎么办呢？最好的办法是分别在不同的容器中运行，通过link进行连接，比如先前实验中用到的web，app，db容器。如果一定要在一个容器中运行多个服务可以考虑用`Supervisord`来进行进程管理，方式就是将多个启动命令放入到一个启动脚本中。

首先安装`Supervisord`，添加下面内容到Dockerfile:

```
RUN apt-get install -yqq supervisor
RUN mkdir -p /var/log/supervisor
```

拷贝配置文件到指定的目录：

```
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
```

其中`supervisord.conf`文件需要放在`/home/shiyanlou/shiyanloutest`下，文件内容如下：

```
[supervisord]
nodaemon=true

[program:apache2]
command=/bin/bash -c "source /etc/apache2/envvars && exec /usr/sbin/apache2ctl -D FOREGROUND"
```

如果有多个服务需要启动可以在文件后继续添加`[program:xxx]`，比如如果有ssh服务，可以增加`[program:ssh]`。

修改`CMD`命令，启动`Supervisord`：

```
CMD ["/usr/bin/supervisord"]
```


操作演示视频
`@
https://labfile.oss-cn-hangzhou.aliyuncs.com/courses/498/video/6-6.flv
@`

## 7. 实验三：从 Dockerfile 创建镜像

将上述内容完成后放入到`/home/shiyanlou/shiyanloutest/Dockerfile`文件中，最终得到的Dockerfile文件如下：

```
# Version 0.2

# 基础镜像
FROM ubuntu:latest

# 维护者信息
MAINTAINER shiyanlou@shiyanlou.com

# 镜像操作命令
RUN apt-get -yqq update && apt-get install -yqq apache2 && apt-get clean
RUN apt-get install -yqq supervisor
RUN mkdir -p /var/log/supervisor

VOLUME ["/var/log/apche2"]

ADD html.tar /var/www
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf

WORKDIR /var/www/html

ENV HOSTNAME shiyanloutest
ENV APACHE_RUN_USER www-data
ENV APACHE_RUN_GROUP www-data
ENV APACHE_LOG_DIR /var/log/apche2
ENV APACHE_PID_FILE /var/run/apache2.pid
ENV APACHE_RUN_DIR /var/run/apache2
ENV APACHE_LOCK_DIR /var/lock/apche2

EXPOSE 80

# 容器启动命令
CMD ["/usr/bin/supervisord"]
```

同时在`/home/shiyanlou/shiyanloutest`目录下，添加`supervisord.conf`文件：

```
[supervisord]
nodaemon=true

[program:apache2]
command=/bin/bash -c "source /etc/apache2/envvars && exec /usr/sbin/apache2ctl -D FOREGROUND"
```

并下载静态页面文件压缩包：

```
cd /home/shiyanlou/shiyanloutest
wget http://labfile.oss.aliyuncs.com/courses/498/html.tar
```

将`http://simplecloud.cn`网站的页面tar包下载到`/home/shiyanlou/shiyanloutest`目录：



`docker build` 执行创建，`-t`参数指定镜像名称：

```
docker build -t shiyanloutest:0.2 /home/shiyanlou/shiyanloutest/
```

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1708timestamp1457408223361.png/wm)

`docker images` 查看创建的新镜像已经出现在了镜像列表中：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1708timestamp1457408382685.png/wm)

`docker inspect shiyanloutest:0.2` 查看该镜像的详细信息：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1708timestamp1457409298274.png/wm)

由该镜像创建新的容器web2，并映射本地的80端口到容器的80端口：

```
docker run -d -p 80:80 --name web2 shiyanloutest:0.2
```

最后打开桌面上的firefox浏览器，输入本地地址访问`127.0.0.1`，看到我们克隆的琛石科技的网站：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1708timestamp1457409249090.png/wm)


操作演示视频
`@
https://labfile.oss-cn-hangzhou.aliyuncs.com/courses/498/video/6-7.flv
@`

## 8. 总结

本节实验中我们学习了以下内容，任何不清楚的地方欢迎到[实验楼问答](https://www.shiyanlou.com/questions)与我们交流：

1. Dockerfile 基本框架
2. Dockerfile 编写常用命令
3. 从 Dockerfile 构建镜像


请务必保证自己能够动手完成整个实验，只看文字很简单，真正操作的时候会遇到各种各样的问题，解决问题的过程才是收获的过程。