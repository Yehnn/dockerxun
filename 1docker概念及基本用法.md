# Docker 概念及基本用法 

## 1. 课程说明

课程为纯动手实验教程，为了能说清楚实验中的一些操作会加入理论内容。理论内容我们不会写太多，已经有太多好文章了，会精选最值得读的文章推荐给你，在动手实践的同时扎实理论基础。

学习过程中遇到的所有问题，都可随时在[实验楼问答](https://www.shiyanlou.com/questions)中提出，与老师和同学一起交流。

实验环境中可以联网，不受实验楼网络限制。

## 2. 学习方法

实验楼的Docker课程包含15个实验，每个实验都提供详细的步骤和截图以及演示视频。适用于有一定Linux系统基础，想快速上手Docker的同学。

学习方法是多实践，多提问。启动实验后按照实验步骤逐步操作，同时理解每一步的详细内容，如果有任何疑问，随时在[实验楼问答](https://www.shiyanlou.com/questions/)中提问，实验楼团队和我都会及时回复大家的所有问题。

如果实验开始部分有推荐阅读的材料，请务必先阅读后再继续实验，理论知识是实践必要的基础。

## 3. 本节内容简介

本实验中我们初步接触Docker的概念和基本用法。需要依次完成下面几项任务：

1. 理解Docker是什么
2. 学习如何在Linux上安装Docker
3. 学习如何使用Docker Hub
4. 创建第一个Hello Shiyanlou的Docker应用
5. Docker基本的容器和镜像管理

## 4. 推荐阅读

本节实验推荐先阅读下述内容：

+ 4.1 [深入浅出Docker（一）：Docker核心技术预览](http://www.infoq.com/cn/articles/docker-core-technology-preview)

这篇文章介绍了Docker产生的技术发展历程，Docker中的核心技术以及相关的子项目，非常好的入门资料。

+ 4.2 [Understand the architecture](https://docs.docker.com/engine/understanding-docker/)

这篇Docker 官方的文章详细介绍了Docker的运行机制和必要的组件。不涉及到很底层的技术，可以做为对Docker的一个初步了解。


## 5. Docker概念

### 5.1 容器技术

Linux容器技术很早就有了，比较有名的是被集成到主流Linux内核中的LXC项目。容器通过对操作系统的资源访问进行限制，构建成独立的资源池，让应用运行在一个相对隔离的空间里，同时容器间也可以进行通信。

容器技术对比虚拟化技术，容器比虚拟化更轻量级，对资源的消耗小很多。容器操作也更快捷，启动和停止都要比虚拟机快。但Docker容器需要与主机共享操作系统内核，不能像虚拟机那样运行独立的内核。

Docker是一个基于LXC技术构建的容器引擎，基于GO语言开发，遵循Apache2.0协议开源。Docker的发展得益于为使用者提供了更好的容器操作接口。包括一系列的容器，镜像，网络等管理工具，可以让用户简单的创建和使用容器。

Docker支持将应用打包进一个可以移植的容器中，重新定义了应用开发，测试，部署上线的过程，核心理念就是 `Build once, Run anywhere`。

Docker容器技术的典型应用场景是开发运维上提供持续集成和持续部署的服务。

下面我们开始介绍Docker中的几个基本概念。

### 5.2 镜像

Docker的镜像概念类似于虚拟机里的镜像，是一个只读的模板，一个独立的文件系统，包括运行容器所需的数据，可以用来创建新的容器。

镜像可以基于Dockerfile构建，Dockerfile是一个描述文件，里面包含若干条命令，每条命令都会对基础文件系统创建新的层次结构。

用户可以通过编写Dockerfile创建新的镜像，也可以直接从类似github的Docker Hub上下载镜像使用。

### 5.3 容器

Docker容器是由Docker镜像创建的运行实例。Docker容器类似虚拟机，可以支持的操作包括启动，停止，删除等。每个容器间是相互隔离的，但隔离的效果比不上虚拟机。容器中会运行特定的应用，包含特定应用的代码及所需的依赖文件。

在Docker容器中，每个容器之间的隔离使用Linux的 `CGroups` 和 `Namespaces` 技术实现的。其中 `CGroups` 对CPU，内存，磁盘等资源的访问限制，`Namespaces` 提供了环境的隔离。

### 5.4 仓库

如果你使用过 `git` 和 `github` 就很容易理解Docker的仓库概念。Docker仓库相当于一个 `github` 上的代码库。

Docker 仓库是用来包含镜像的位置，Docker提供一个注册服务器（Registry）来保存多个仓库，每个仓库又可以包含多个具备不同tag的镜像。Docker运行中使用的默认仓库是 Docker Hub 公共仓库。

仓库支持的操作类似 `git`，创建了新的镜像后，我们可以 `push` 提交到仓库，也可以从指定仓库 `pull` 拉取镜像到本地。

## 6. 安装

### 6.1 Ubuntu 上安装最新Docker

实验楼提供的环境中已经安装好了Docker，如果我们想自己安装也并不麻烦，针对不同系统都提供了详细的文档介绍。

针对实验楼提供的实验环境，是Ubuntu 14.04 64位操作系统，只需要执行下面的命令就可以安装：

```
$ sudo apt-get update
$ sudo apt-get install docker
```

但是Ubuntu默认的源中版本是`1.5`，太老了，目前实验楼环境中安装的是最新的 `Docker 1.10` 版本。

**下面的步骤仅供参考，无需在实验楼的环境里重复安装。**

安装的时候添加的是Docker官方的`apt-get`源，安装过程如下：

```
$ sudo apt-get update
$ sudo apt-get install apt-transport-https ca-certificates
$ sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
```

添加文件`/etc/apt/sources.list.d/docker.list`，然后向文件中写入下面的内容：

```
deb https://apt.dockerproject.org/repo ubuntu-trusty main
```

最后再次执行安装命令：

```
$ sudo apt-get update
$ sudo apt-get install docker-engine
```

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1702timestamp1456717978376.png/wm)

更多安装选项可以参考[Docker 安装文档](https://docs.docker.com/engine/installation/linux/ubuntulinux/)

安装完成后，可以使用 `docker version` 查看Docker的版本信息。

```
$ docker version
```

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1702timestamp1456717987219.png/wm)

### 6.2 基本命令

#### 查看Docker命令

查看所有 Docker 命令：

```
$ docker 
```

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1702timestamp1456717996404.png/wm)

查看指定命令的帮助`docker command --help`，例如启动命令`docker run --help`：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1702timestamp1456718001732.png/wm)

### 6.3 Docker 服务

配置Docker服务启动选项，可以通过修改文件 `/etc/default/docker` 来实现，查看实验楼中已有的配置文件内容：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1702timestamp1456718012622.png/wm)

看到其中的启动参数为 `DOCKER_OPTS`，表示 Docker 服务启动时会监听在 `tcp://127.0.0.1:4243`。

如果你修改这个文件则需要重新启动 Docker 服务。

停止Docker服务：

```
$ sudo service docker stop
```

启动Docker服务：

```
$ sudo service docker start
```

查看 Docker 服务状态：

```
$ sudo service docker status
```

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1702timestamp1456718019626.png/wm)

操作演示视频
`@
https://labfile.oss-cn-hangzhou.aliyuncs.com/courses/498/video/1-6.flv
@`

## 7. Hello Shiyanlou 应用

### 7.1 需求描述

Docker 安装后，我们开始尝试启动第一个 Docker 应用。这个应用很简单，作用就是输出一句 `Hello, Shiyanlou!`。

### 7.2 需求分析

应用执行的命令是 `echo "Hello, Shiyanlou"`，我们需要为这个影响构建一个运行的容器，让这条命令在容器中运行，运行后容器自动退出。

### 7.3 解决方案

首先，我们需要有一个镜像来运行这个应用，这里我们选择用 `busybox` 镜像，直接使用 `docker run` 命令来运行容器：

```
$ docker run busybox echo "Hello, Shiyanlou"
```

其中后面跟的 busybox 是镜像的名字， busybox 镜像不需要提前从仓库中下载到本地，因为 Docker会首先在本地查找，如果找不到就会自动从 Docker Hub 或系统配置的默认 Registry 中下载 busybox 镜像。

最后的 `echo "Hello, Shiyanlou"` 是在容器中运行的命令。`docker run` 命令会基于指定的镜像运行一个容器实例，然后把后续的命令传递给该容器内部运行。

该操作运行的结果会在屏幕上输出 `Hello, Shiyanlou`。

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1702timestamp1456718036485.png/wm)

运行过程见上图所示，第一个箭头指向的是运行的命令，第二个箭头表示在本地没有查找到 `busybox:latest` 的镜像，所以需要从远端的Docker Registry 中 pull，镜像下载下来后进入到执行应用阶段，最后一个箭头表示应用执行完毕输出了`Hello, shiyanlou!`字符串到屏幕上。

执行 `docker ps` 命令查看运行的容器列表：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1702timestamp1456718042076.png/wm)

发现没有任何容器在运行，原因是容器运行了 echo 命令后已经终止，进入到停止状态，需要用 `docker ps -a` 命令查看。

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1702timestamp1456718049006.png/wm)

在这个命令的输出中我们看到列表里有一个处于 `Exited (0 3 minutes ago)` 的容器，这个容器表示大约3分钟之前终止。列表中还包括该容器的ID，是用的镜像，执行的命令，创建的时间等信息。最后一个是为容器随机设置的名称，也可以通过`run`的参数进行设置为指定名称，注意同一个服务器上的容器不可以同名。

操作演示视频
`@
https://labfile.oss-cn-hangzhou.aliyuncs.com/courses/498/video/1-7.flv
@`

## 8. 容器管理

本节实验中我们创建几个容器，并介绍 docker 容器管理中最常用的几个命令。

### 8.1 docker run

最主要的是创建运行容器的命令 `docker run`，这个命令的参数非常多，可以通过 `docker run --help` 查看。

继续上一节实验，echo 命令运行后容器就退出了，如果我们需要一个保持运行的容器呢，最简单的方法就是给这个容器一个可以保持的应用，比如`bash`，运行 ubuntu  容器并进入容器的 bash：

```
$ docker run -t -i ubuntu /bin/bash
```

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1702timestamp1456718056669.png/wm)

上面命令的说明：

1. `-t`：分配一个 `pseudo-TTY`
2. `-i`：`--interactive`参数缩写，表示交互模式，如果没有 `attach` 保持 STDIN 打开状态
3. `ubuntu`：运行的镜像名称，默认为`latest` 标签
4. `/bin/bash`：容器中运行的应用

通过这个简单的命令，我们现在进入了新创建容器的bash中，在bash里执行的任何命令都不会影响到我们的宿主机，可以随意操作。你可以看到主机名和环境变量 `HOSTNAME` 都已经显示为容器的ID了。

在这个bash下，我们可以进行各种Ubuntu系统上的操作，当然因为Docker本身的限制，有些涉及到磁盘、网络、设备等Linux特权命令是无法执行的，可以试试`reboot`命令，会提示你`shutdown: Unable to shutdown system`。

如何退出这个bash呢？有两种方法，两种方法的效果完全不同：

1. 直接 `exit`，这时候 bash 程序终止，容器进入到停止状态
2. 使用组合键退出，仍然保持容器运行，我们可以随时回来到这个bash中来，组合键是 `Ctrl-p Ctrl-q`，你没有看错，是两组组合键，先同时按下`Ctrl`和p，再按`Ctrl`和q。就可以退出到我们的宿主机了。

上述第二种方法比较常用，此时使用 `docker ps` 查看，能看到容器仍然在运行中：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1702timestamp1456718063612.png/wm)

**注意：**有些浏览器把`Ctrl-Q`作为退出标签的快捷键，可能在执行上述命令时实验楼的页面会关掉，不过没有关系，重新打开实验楼的网站登陆后点击头像旁边的`继续实验`就能够再次回到实验桌面。

如果想再次回到刚才的bash中，只需要使用 `docker attach`命令就可以再次连接到运行的bash里：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1702timestamp1456718083623.png/wm)

**注意：** 命令后面的参数是容器的ID，并不需要输入完整的数字，只要能唯一定位这个容器即可，通常输入4位就足够了。


本节只是针对Docker有一个基本的概念，`docker run` 的其他参数我们将在后续的实验课程中详细介绍。

### 8.2 docker ps

`docker ps`命令用来查看正在运行的容器：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1702timestamp1456718092203.png/wm)

几个最常用的参数：

+ `-a`：查看所有容器，含停止运行的
+ `-l`：查看刚启动的容器
+ `-q`：只显示容器ID

我们查看所有容器的ID列表：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1702timestamp1456718100340.png/wm)

### docker start

`docker start` 启动容器，此处我们尝试启动先前的运行`/bin/bash`的容器：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1702timestamp1456718111049.png/wm)

步骤说明：

1. `docker ps -a` 查看当前所有的容器，得到该容器的ID
2. `docker start` 启动该容器
3. `docker ps` 再次查看运行的容器

最后我们可以通过 `docker attach` 连接到该bash上继续操作。

### 8.3 docker stop

`docker stop` 停止容器，此处我们停止刚才启动的`/bin/bash`容器：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1702timestamp1456718116876.png/wm)

此处还有一些其他的类似命令，比如 `docker restart` 重启运行中的容器，相当于先`stop`，再`start`。

### 8.4 docker inspect

查看 Docker 容器或镜像的一些内部信息：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1702timestamp1456718127800.png/wm)

启动并查看 `/bin/bash` 容器：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1702timestamp1456718133721.png/wm)

返回的信息非常多，是JSON格式，每一项内容具体含义本节不做详细介绍，可以参考官方文档。

### 8.5 docker rm

删除容器操作。该命令默认不可以删除运行的容器，但提供了强制删除的参数`-f`，下面的实验操作中我们尝试删除一个运行容器会报错，只可以把停止状态的`Hello, shiyanlou!`容器删除：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1702timestamp1456718140348.png/wm)

### 8.6 docker top

查看容器中运行的进程信息，显示容器中进程的PID，UID，PPID，时间，tty等信息。

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1702timestamp1456718146371.png/wm)

本节作为入门介绍，无法涵盖所有命令，后续单独的`容器管理`实验中学习更多的操作命令。

操作演示视频
`@
https://labfile.oss-cn-hangzhou.aliyuncs.com/courses/498/video/1-8.flv
@`
## 9. 镜像管理

这里介绍镜像获取和创建的最常见的方法。更多实验会在`镜像管理`实验中学习。

### 9.1 查看镜像列表

`docker images` 命令我们可以列出当前系统上所有的镜像信息：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1702timestamp1456718152616.png/wm)

其中：

1. REPOSITORY：仓库名称
2. TAG：标签名，一个仓库可以有若干个标签对应不同的镜像，默认都是`latest`
3. IMAGE ID：镜像ID
4. CREATED：创建时间，注意不是本地的pull时间
5. SIZE：镜像大小

### 9.2 获取镜像

最简单的获取镜像的方式是从 Docker Hub上 `pull` 最新的镜像，比如我们想要一个 `busybox` 的镜像，直接使用命令：

```
$ docker pull busybox
```

由于我们已经在第一步骤`Hello, shiyanlou!`中下载过这个镜像，则会显示如下信息：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1702timestamp1456718159844.png/wm)

查看镜像列表 `docker images`，可以看到新pull的镜像已经在列表里了：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1702timestamp1456718165850.png/wm)


可以查看 [Docker Hub](https://hub.docker.com/explore/) 查看别人做好的镜像，这个网站上有很多制作很好的官方镜像，可以避免我们自己在制作镜像过程中重复造轮子。


### 9.3 创建镜像

最常用的是写一个Dockerfile，从Dockerfile里创建新的镜像。

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

还记得如何退出bash并保持运行吗？那个组合键。

### 9.4 清理镜像

退出到宿主机后，请使用 `docker rm` 命令删除容器，并使用 `docker rmi` 删除镜像。

```
$ docker rmi shiyanlou
$ docker images
```

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1702timestamp1456718193708.png/wm)

上述命令说明：

1. `docker ps` 查看运行的容器
2. `docker rm -f 6c86` 强制删除运行的容器
3. `docker rmi shiyanlou` 删除shiyanlou镜像
4. `docker images` 查看镜像列表

操作演示视频
`@
http://labfile.oss-cn-hangzhou.aliyuncs.com/courses/498/video/1-9.flv
@`
## 10. 总结

本节实验中我们学习了以下内容，任何不清楚的地方欢迎到[实验楼问答](https://www.shiyanlou.com/questions)与我们交流：

1. 理解Docker是什么
2. 学习如何在Linux上安装Docker
3. 学习如何使用Docker Hub
4. 创建第一个Hello Shiyanlou的Docker应用
5. Docker基本的容器和镜像管理

请务必保证自己能够动手完成整个实验，只看文字很简单，真正操作的时候会遇到各种各样的问题，解决问题的过程才是收获的过程。

