# Docker Compose项目

## 1. 课程说明

课程为纯动手实验教程，为了能说清楚实验中的一些操作会加入理论内容。理论内容我们不会写太多，已经有太多好文章了，会精选最值得读的文章推荐给你，在动手实践的同时扎实理论基础。

学习过程中遇到的所有问题，都可随时在[实验楼问答](https://www.shiyanlou.com/questions)中提出，与老师和同学一起交流。

实验环境中可以联网，不受实验楼网络限制。

## 2. 学习方法

实验楼的Docker课程包含15个实验，每个实验都提供详细的步骤和截图，适用于有一定Linux系统基础，想快速上手Docker的同学。

学习方法是多实践，多提问。启动实验后按照实验步骤逐步操作，同时理解每一步的详细内容，如果有任何疑问，随时在[实验楼问答](https://www.shiyanlou.com/questions/)中提问，实验楼团队和我都会及时回复大家的所有问题。

如果实验开始部分有推荐阅读的材料，请务必先阅读后再继续实验，理论知识是实践必要的基础。

## 3. 本节内容简介

在本实验里我们将通过实验学习Docker Compose项目。Docker Compose 是一个用来创建和运行多容器应用的工具。使用Compose首先需要编写Compose文件来描述多个容器服务以及之间的关联，然后通过命令根据配置启动所有的容器。

Dockerfile 可以定义一个容器，而一个 Compose 的模板文件（YAML格式）可以定义一个包含多个相互关联容器的应用。Compose 项目使用 python 编写，基于后面的实验中我们将学习的 Docker API实现。

Docker Compose 项目地址：[https://github.com/docker/compose](https://github.com/docker/compose) 感兴趣的同学可以关注。

本节中，我们需要依次完成下面几项任务：

1. 安装 Docker Compose
2. 创建 Docker Compose 服务 - Web+redis 网站

在实验之前，为了能够顺利连接 docker.io 我们使用阿里云的 Docker Hub 加速服务，在服务器上配置`/etc/default/docker`文件中的`DOCKER_OPTS`，然后再重启 Docker：


```
# 配置文件中添加 "--registry-mirror=https://n6syp70m.mirror.aliyuncs.com"
# 重启 Docker
$ sudo service docker restart
```

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1713timestamp1458794123429.png/wm)

## 4. Docker Compose

### 4.1 概述

#### Compose

Compose 是定义和运行多容器 `Docker` 应用程序的工具。使用 Compose，可以通过编辑 `YAML` 文件来配置应用程序的服务。它可以用来管理应用程序的生命周期，例如启动，停止以及重构服务。

#### service

在分布式应用程序中，应用程序的不同部分被称为服务（service），例如常见的提供数据库存储的服务。服务实际上只是生产中的镜像。一个服务仅仅运行一个镜像，但它定义了服务运行的方式，例如使用哪个端口，该容器应该运行多少个副本等。

#### 使用过程

使用 Compose 的三个过程如下：

1. 定义应用程序的环境，即 Dockerfile

2. 定义组成应用程序的服务，一般为定义 `docker-compose.yml` 文件

3. 启动整个应用程序

> 关于 docker-compose.yml 文件的详细编写格式可以参考：https://docs.docker.com/compose/compose-file/#reference-and-guidelines

目前有三种版本的 Compose 文件格式：

1. version 1: 最早的版本使用传统格式，将在未来的 Compose 版本中被弃用

2. version 2: 现在使用最多的文件格式

3. version 3: 最新的版本，旨在 Compose 和被集成到 Docker Engine 中的 swarm mode 之间互相兼容（swarm 在下一节的内容会学习相关的知识）。

### 4.2 安装

在 linux 中，Compose 需要单独安装，我们需要从 github 上下载 Docker Compose 二进制文件。但是官网提供的从 github 上下载的链接速度十分缓慢，在实验环境中，我们已提供该文件，直接使用以下命令进行下载：

```
$ wget http://labfile.oss-cn-hangzhou.aliyuncs.com/courses/980/software/docker-compose-Linux-x86_64
```

下载成功后，为了能够直接使用该可执行文件执行命令，一般将其放入 `$PATH` 的环境变量支持的路径中，并添加可执行权限，使用的命令如下：

```
$ sudo mv docker-compose-Linux-x86_64 /usr/local/bin/docker-compose

$ sudo chmod +x /usr/local/bin/docker-compose
```

执行完成后，就能够在终端下直接使用 `docker-compose` 命令了：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517365151690.png/wm)

### 4.3 实例

在这里，我们将创建一个 web 应用程序，该实例参考 Docker Compose 官方文档，有两个服务，并做了一些改变。

在本实验中，我们需要两个容器：

1. web 容器：提供 web 服务，并连接后端的 redis 服务

2. redis 容器：提供 redis 服务，接收 web 容器的连接

其文件目录结构如下所示：

```
app
|----web
|     |----web.py
|     |----requirements.txt
|     |----Dockerfile
|
|----docker-compose.yml
```

1. 首先编辑 `app/web/web.py` 文件，写入下面的内容：

```py
from flask import Flask
from redis import Redis

app = Flask(__name__)
redis = Redis(host='redis', port=6379)

@app.route('/')
def hello():
    redis.incr('number')
    return 'Hello Shiyanlou! # %s' % redis.get('number')

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=80, debug=True)
```

上述代码创建一个十分简单的 web 应用程序。该程序会连接 `redis` 服务，在访问 `/` 页面时，会自动将变量 `number` 加一，该 `INCR` 命令也在 `redis 数据类型` 一节中学习过。

2. 编辑 `app/web/requirements.txt` 文件，输入如下内容：

```txt
flask==0.10
redis==2.10.3
```

创建 `requirements.txt` 文件，输入需要使用的 python 依赖包，方便安装。

3. 编辑 `app/web/Dockerfile` 文件，添加如下内容：

```
FROM python:2.7
COPY ./ /web/
WORKDIR /web
RUN pip install -r requirements.txt
CMD python web.py
```

上述 `Dockerfile` 定义了一个镜像，该镜像基于 `python:2.7` 镜像制作，在其基础上安装相应的 python 包，并执行 `CMD` 命令来启动该应用程序。

4. 编辑 `app/docker-compose.yml` 文件：

```txt
services:
  redis:
    image: redis:3.2
  web:
    build:
      context: /home/shiyanlou/app/web
    depends_on:
    - redis
    ports:
    - 8001:80/tcp
    volumes:
    - /home/shiyanlou/app/web:/web:rw
version: '3.0'
```

该 `docker-compose.yml` 文件定义了两个服务，分别为 `web` 和 `redis` 服务。并且我们配置 `web` 服务的端口映射，以及挂载相应的目录。 `depends_on` 定义了依赖关系，被依赖服务的容器需要先创建。

5. 进入 `app` 目录下，执行 `docker-compose up` 命令来启动服务：

```
$ cd app
$ docker-compose up
```

由于此时 `web` 服务的镜像还未构建，所以此时会自动根据 `build`指示，使用 `/home/shiyanlou/app/web/Dockerfile` 文件构建镜像。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517557297094.png/wm)

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517557398547.png/wm)

运行成功后，此时我们可以打开浏览器，输入 `127.0.0.1:8001`，获取到正确的结果：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517368221961.png/wm)

6. 除此之外，也可以使用 `-d` 参数，即 `docker-compose up -d` 在其在后台运行：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517557688428.png/wm)

7. 如果需要暂停以及删除容器，可以直接运行 `docker-compose down` 命令即可

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517558444487.png/wm)

## 7. 总结

本节实验中我们学习了以下内容：

1. 安装 Docker Compose
2. 创建 Docker Compose 服务 - Web+redis网站

请务必保证自己能够动手完成整个实验，只看文字很简单，真正操作的时候会遇到各种各样的问题，解决问题的过程才是收获的过程。