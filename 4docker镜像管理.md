# 镜像管理

## 1. 课程说明

课程为纯动手实验教程，为了能说清楚实验中的一些操作会加入理论内容。理论内容我们不会写太多，已经有太多好文章了，会精选最值得读的文章推荐给你，在动手实践的同时扎实理论基础。

学习过程中遇到的所有问题，都可随时在[实验楼问答](https://www.shiyanlou.com/questions)中提出，与老师和同学一起交流。

实验环境中可以联网，不受实验楼网络限制。

## 2. 学习方法

实验楼的Docker课程包含15个实验，每个实验都提供详细的步骤和截图，适用于有一定Linux系统基础，想快速上手Docker的同学。

学习方法是多实践，多提问。启动实验后按照实验步骤逐步操作，同时理解每一步的详细内容，如果有任何疑问，随时在[实验楼问答](https://www.shiyanlou.com/questions/)中提问，实验楼团队和我都会及时回复大家的所有问题。

如果实验开始部分有推荐阅读的材料，请务必先阅读后再继续实验，理论知识是实践必要的基础。

## 3. 本节内容简介

第一节实验中我们已经接触了一些镜像的概念，简单的说镜像就是一个容器的只读模板，用来创建容器。当运行容器时需要指定镜像，如果本地没有该镜像，则会从Docker Registry下载。默认查找的是Docker Hub。Docker的镜像是增量的修改，每次创建新的镜像都会在老的镜像上面构建一个增量的`层`，使用到的技术是`Another Union File System(AUFS)`，感兴趣的同学可以学习文档 [InfoQ:剖析Docker文件系统：Aufs与Devicemapper](http://www.infoq.com/cn/articles/analysis-of-docker-file-system-aufs-and-devicemapper/)。

本节中，我们需要依次完成下面几项任务：

1. 使用Docker Hub查找和下载镜像
2. 创建镜像
3. 查看镜像信息
4. 导入和导出镜像
5. 修改镜像
6. 删除镜像

## 4. 实验一：使用Docker Hub

镜像存储中的核心概念仓库（Repository）是镜像存储的位置。Docker 注册服务器（Registry）是仓库存储的位置。每个仓库包含不同的镜像。

比如一个镜像名称 `ubuntu:14.04`，冒号前面的`ubuntu`是仓库名，后面的`14.04`是TAG，不同的TAG可以对应相同的镜像，TAG通常设置为镜像的版本号。

Docker Hub 是Docker官方提供公共仓库，提供大量的常用镜像，由于国内网络原因经常连接Docker Hub会比较慢，所以我们也可以选择一些国内提供类似Docker Hub镜像服务站点。连接Docker Hub的常用命令包括：

1. 搜索镜像 `docker search`
2. 下载镜像 `docker pull`

本节实验中，我们需要一个busybox镜像，首先进行搜索，然后使用`docker pull`下载到本地：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1706timestamp1457320774849.png/wm)

查找到的数据中包含仓库名称，描述，以及有多少人关注。我们只需要下载最基本的`Busybox base image`就可以。

查找命令返回的结果中通常可以看到不同版本的busybox，不指定版本号默认下载`busybox:latest`。

使用 `docker pull` 命令将镜像下载到本地：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1706timestamp1457320901711.png/wm)

我们也可以在Docker Hub上创建一个账户，用来保存所需的镜像，但是在国内使用实在是太慢了。这里简单介绍下Docker中使用命令登陆Docker Hub保存镜像的方式：

1. 首先在Docker Hub注册一个账号：[注册链接](https://hub.docker.com/)
2. 然后可以基于Docker Hub上现有的镜像创建一个镜像
3. 在本地完成修改后使用`docker push`命令推送到Docker Hub上

此外，Docker Hub提供一个强大的自动创建镜像的功能，可以设定跟踪某个镜像中安装的软件，如果有更新则自动重新构建新的镜像。更多有趣的功能可以登录到Docker Hub 官网进行体验。在此不做更多介绍。

如果想创建一个本地的Docker Hub，在后续实验中我们会学习如何在本地搭建一个`Registry`服务器。

操作演示视频
`@
https://labfile.oss-cn-hangzhou.aliyuncs.com/courses/498/video/4-4.flv
@`

## 5. 实验二：创建镜像

### 5.1 下载镜像 `docker pull`
在本地创建镜像的方法有几种，最简单的是直接从`Registry`服务器上下载。上一节实验中已经有介绍：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1706timestamp1457321005751.png/wm)

镜像下载中可以看到是分层下载，每一层都有一个唯一的ID值表示，每层下载的大小实际为该层进行的修改增量。

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1706timestamp1457321038851.png/wm)

### 5.2 创建镜像 Dockerfile

后续章节中我们会详细介绍Dockerfile的编写，这里仅仅介绍最简单的使用。

Dockerfile 可以很方便的基于已有镜像创建新的镜像。Dockerfile文件里包含若干条命令，每个命令都会创建一个新的层，Dockerfile创建的层数不可以超过127层。

回顾下先前实验中我们的一个最简单的Dockerfile实例：

Dockerfile的详细编写方法我们后续有专门的实验介绍，此处只写一个最简单的Dockerfile来介绍。

使用 vim 或 gedit 打开一个文件 `Dockerfile`：

```
$ cd /home/shiyanlou/
$ mkdir shiyanlouimage
$ cd shiyanlouimage/
$ vim Dockerfile
```

在文件中输入以下内容：

```
from ubuntu:latest
ENV HOSTNAME=shiyanlou
```

保存退出编辑器。

这个 `Dockerfile` 中只有两行，第一行表示基于哪个镜像创建新的镜像，类似于程序开发中的 `import` 或 `include`，我们这里以 `ubuntu:latest` 镜像为基础创建新的镜像。第二行是在新的镜像中我们要对基础镜像 `ubuntu:latest` 做的改变。这句是设置一个环境变量`HOSTNAME`等于`shiyanlou`。

完成 Dockerfile 后，使用 `docker build` 命令进行构建：

```
$ cd /home/shiyanlou/shiyanlouimage/
$ docker build -t shiyanlou .
```

这个命令中第一个参数 `-t shiyanlou` 指定创建的新镜像的名字，第二个参数是一个点 `.` 指定从当前目录查找 Dockerfile 文件。

命令执行过程截图：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1702timestamp1456718176359.png/wm)

执行完成后我们 `docker images` 命令中就可以看到新的 `shiyanlou` 镜像了。

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1702timestamp1456718181759.png/wm)

我们现在运行这个 shiyanlou 镜像并进入到bash环境：

```
$ docker run -t -i shiyanlou /bin/bash
```

进入到bash后，我们查看镜像是否已经设置了`HOSTNAME`环境变量：

```
$ echo $HOSTNAME
```

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1702timestamp1456718187777.png/wm)


### 5.3 其他方法

创建镜像的方法很多，除了上述两种之外还可以使用下述方法进行创建，每种方法都会在本节或其他章节实验中学习：

1. 在容器管理中我们学过的 `docker import`
2. 本章节后续要学习的提交修改 `docker commit`
3. 本章节后续要学习的导入镜像 `docker load`

操作演示视频
`@
https://labfile.oss-cn-hangzhou.aliyuncs.com/courses/498/video/4-5.flv
@`

## 6. 实验三：查看镜像信息

### 6.1 基本命令 `docker images`

`docker images` 命令查看本地的镜像列表，信息包括：

1. REPOSITORY：仓库名称
2. TAG：标签名，一个仓库可以有若干个标签对应不同的镜像，默认都是`latest`
3. IMAGE ID：镜像ID
4. CREATED：创建时间，注意不是本地的pull时间
5. SIZE：镜像大小

其中需要注意的是运行容器时候如果不指定镜像的TAG，则默认为latest。镜像的唯一标识符是镜像ID，不是TAG，有的时候同一个镜像可以有不同的TAG，但实际指向的是同一个镜像ID。TAG可以理解为镜像的别名。

查看当前系统中存储的所有镜像信息：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1706timestamp1457321281512.png/wm)

### 6.2 查看镜像详细信息 `docker inspect`

`docker inspect` 可以查看指定镜像的详细信息。这条命令可以查看容器或镜像的详细信息，输出是一个JSON格式的内容，比较重要的信息是创建时间，启动命令等：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1706timestamp1457321350932.png/wm)

可以看到输出的信息非常多，如果想查看其中的一项，只需要使用`-f {{}}` 指定即可：


![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1706timestamp1457321454072.png/wm)

操作演示视频
`@
https://labfile.oss-cn-hangzhou.aliyuncs.com/courses/498/video/4-6.flv
@`

## 7. 实验四：导出及导入镜像

与容器的导出和导入类似（请回忆相关命令），镜像可以被导出到本地文件，也可以从本地文件中加载。导出命令是 `docker save` 命令，导出后的镜像如果需要导入到新的Docker 服务器，则使用`docker load`命令。

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1706timestamp1457321548101.png/wm)

导出的镜像文件是`/home/shiyanlou/busybox.tar`，可以拷贝到其他Docker服务器上进行导入：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1706timestamp1457321706683.png/wm)

操作演示视频
`@
https://labfile.oss-cn-hangzhou.aliyuncs.com/courses/498/video/4-7.flv
@`

## 8. 实验五：更新及删除镜像

### 8.1 更新镜像 `docker commit`

如果需要对镜像进行更新的话，一种方法是创建容器，在容器中进行修改，然后将修改后容器提交到镜像中。提交使用 `docker commit`命令。

**注意：**本方法不推荐用在生产系统中，未来会很难维护镜像。最好的创建镜像的方法是Dockerfile，修改镜像的方法是修改Dockerfile，然后重新从Dockerfile中构建新的镜像。

本实验中，我们首先基于shiyanlou镜像创建一个容器：

```
docker run -t -i --name updateimage shiyanlou /bin/bash
```

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1706timestamp1457322023230.png/wm)

进入到容器中进行修改，创建三个新的文件夹，然后退出容器：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1706timestamp1457322028645.png/wm)


`docker diff` 查看修改的内容：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1706timestamp1457322541393.png/wm)

`docker commit`命令将修改后的内容提交到本地，另存为镜像`newshiyanlou`：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1706timestamp1457322454043.png/wm)

几个参数的说明：

+ -m 本次提交的描述
+ -a 指定镜像作者信息
+ -p 提交时暂停容器运行
+ 容器的ID或名称
+ 目标镜像

如果指定了目标镜像，Docker会创建新的镜像。类似我们修改一个word文档后进行的另存为。

操作演示视频
`@
https://labfile.oss-cn-hangzhou.aliyuncs.com/courses/498/video/4-8.flv
@`

### 8.2 删除镜像 `docker rmi`

`docker rmi`命令可以删除本地的镜像，删除前需要先使用`docker rm` 删除所有依赖该镜像的容器。

`docker rmi -f` 可以强制删除存在容器依赖的镜像，但这不是一个好习惯，请先删除容器再清理镜像。

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1706timestamp1457322624055.png/wm)

操作演示视频
`@
https://labfile.oss-cn-hangzhou.aliyuncs.com/courses/498/video/4-8.flv
@`

## 9. 总结

本节实验中我们学习了以下内容，任何不清楚的地方欢迎到[实验楼问答](https://www.shiyanlou.com/questions)与我们交流：

1. 使用Docker Hub查找和下载镜像
2. 创建镜像
3. 查看镜像信息
4. 导入和导出镜像
5. 修改镜像
6. 删除镜像

请务必保证自己能够动手完成整个实验，只看文字很简单，真正操作的时候会遇到各种各样的问题，解决问题的过程才是收获的过程。
