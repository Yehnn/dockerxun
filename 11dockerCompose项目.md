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
2. 创建 Docker Compose 服务 - Web+redis网站
3. Docker Compose 服务管理

在实验之前，为了能够顺利连接 docker.io 我们使用阿里云的 Docker Hub 加速服务，在服务器上配置`/etc/default/docker`文件中的`DOCKER_OPTS`，然后再重启 Docker：


```
# 配置文件中添加 "--registry-mirror=https://n6syp70m.mirror.aliyuncs.com"
# 重启 Docker
$ sudo service docker restart
```

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1713timestamp1458794123429.png/wm)

## 4. 实验一：安装 Docker Compose

最简单的安装方法是 pip 命令直接安装 `pip install docker-compose`，但我们要装最新版，最好的方式是从 github 下载。

目前 Docker Compose 发布的最新版本是 1.6.2，可以从github上直接下载安装，因为网速原因，我们把github上的安装包上传到了阿里云存储上：

```
$ cd /home/shiyanlou
$ curl -L http://labfile.oss.aliyuncs.com/courses/498/docker-compose-`uname -s`-`uname -m` > /home/shiyanlou/docker-compose
```

将下载的二进制文件拷贝到`/usr/local/bin` 目录：

```
$ sudo cp /home/shiyanlou/docker-compose /usr/local/bin/docker-compose
$ sudo chmod a+x /usr/local/bin/docker-compose
```

安装完成后查看 `docker-compose` 命令的版本：

```
$ docker-compose --version
```

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1713timestamp1458792923737.png/wm)

如果需要删除 Compose，只需要将执行文件删除即可：`sudo rm -rf /usr/local/bin/docker-compose`

操作演示视频
`@
http://labfile.oss-cn-hangzhou.aliyuncs.com/courses/498/video/11-4.flv
@`

## 5. 实验二：创建 Docker Compose 服务 - Web+redis网站

在本实验中我们将创建含有两个容器的Web服务，该实例参考 Docker Compose 官方文档，并做了一些改变。

通常实现一个 Docker Compose 项目包含下面几个步骤：

1. 实现容器中运行的应用程序
2. 创建 Compose 中所有容器所需的镜像
3. 创建 Compose 配置文件定义服务
4. 测试并启动服务

### 5.1 实现应用

在本实验中，我们需要两个容器：

1. web 容器：提供 web 服务，并连接后端的 redis 服务
2. redis 容器：提供 redis 服务，接收 web 容器的连接

我们需要自己实现的 web 应用程序放在 web 容器中运行，这个应用可以是任意语言开发，我们选择使用 python 的 flask web 框架实现一个简单的 web 应用。

创建一个目录，在该目录下编辑 `app.py` 文件：

```
$ mkdir /home/shiyanlou/webapp
$ cd /home/shiyanlou/webapp
$ vim app.py
```

`app.py` 文件中写入下面的内容：

```
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

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1713timestamp1458792983776.png/wm)

这个程序很简单，哪怕不懂 Python 也没有关系，简单的说明下程序的作用：

1. 引入 Flask 和 Redis 包
2. 连接 Redis 服务
3. 创建一个访问web页面`/`时执行的逻辑，逻辑中自动增加redis里的一个变量`number`，并返回一句`Hello Shiyanlou! # %s` 给页面。
4. 运行这个应用，监听在80端口

在 `/home/shiyanlou/webapp` 目录下创建 `requirements.txt` 文件：

```
$ vim /home/shiyanlou/webapp/requirements.txt
```

输入所需要的python 依赖包：

```
flask
redis
```

至此，我们的简单的 web 应用程序已经开发完成。

### 5.2 定义镜像

web 服务所需的两个容器中，其中 redis 镜像可以直接从 Docker Hub 下载现成的，web 镜像我们需要写一个 Dockerfile 来自定义。

创建 Dockerfile：

```
$ mkdir /home/shiyanlou/webdocker
$ cd /home/shiyanlou/webdocker
$ cp -a /home/shiyanlou/webapp ./
$ vim Dockerfile
```

输入下面的内容：

```
FROM python:2.7
ADD ./webapp /webapp
WORKDIR /webapp
RUN pip install -r requirements.txt
CMD python app.py
```

很简单的一个 Dockerfile，基于 Python 2.7 的 Docker 镜像制作，将应用程序拷贝到镜像中并安装所需的Python软件包，最后的CMD用来启动 Flask web 应用。

这时候我们可以使用 `docker build` 创建 web 镜像，然后在 Compose 的配置文件中指定镜像名称为 web，也可以在 配置文件中直接指定 Dockerfile， Compose 会自动 build 镜像。


### 5.3 定义服务

我们开始编写 Compose 的配置文件，来定义这个包含两个容器的 Web 服务。

创建 Docker Compose 配置文件：

```
$ mkdir /home/shiyanlou/composetest
$ cd /home/shiyanlou/composetest
$ vim docker-compose.yml
```

在配置文件中输入下面的内容：

```
version: '2'
services:
  web:
    build: /home/shiyanlou/webdocker
    ports:
     - "80:80"
    volumes:
     - /home/shiyanlou/webdocker/webapp:/webapp
    depends_on:
     - redis
  redis:
    image: redis
```

对这个配置文件进行说明：

1. version 表示配置文件的语法版本，Compose 1.6.0 + 和 Docker 1.10.0+ 开始支持 version 2。
2. services 下面列举了服务所需的所有容器，每个名称对应一个容器
3. web 服务1的名称为web
4. build 指定 Dockerfile 所在的路径，从该路径build当前容器所需的镜像
5. ports 端口映射
6. volumes 挂载的数据卷，这里可以不用写，但每次修改 `app.py` 的代码后都需要重新 build web镜像，为了方便调试，我们直接挂载宿主机上的目录到容器中
7. depends_on 依赖关系，被依赖的容器需要先创建
8. redis 服务2的名称为redis
9. image redis 容器所用的镜像

这个配置文件中只使用了最常用的配置信息，详细的配置参数可以见[Compose File](https://docs.docker.com/compose/compose-file/)。

### 5.4 启动服务

完成了服务配置文件后，我们将使用 docker-compose 命令来创建和启动 Web 服务。

首先进入到 Compose 文件所在的目录，执行 `docker-compose up`：

```
$ cd /home/shiyanlou/composetest
$ docker-compose up
```

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1713timestamp1458794189509.png/wm)

这个命令会有很多的输出，依次执行：

1. 获取redis镜像
2. build web 镜像
3. 启动 redis 容器
4. 启动 web 容器并映射端口

服务启动后我们打开浏览器，访问`http://localhost`页面，可以看到 web 服务的输出，每次刷新一次页面计数都会加一：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1713timestamp1458795153482.png/wm)

在命令行界面使用 Ctrl-C 终止 `docker-compose up` 的运行，服务停止。


操作演示视频
`@
http://labfile.oss-cn-hangzhou.aliyuncs.com/courses/498/video/11-5.flv
@`

## 6. 实验三：Docker Compose 服务管理

上面我们已经学习了 `docker-compose up` 命令用来创建和启动服务，本节实验我们将继续学习其他 Compose 常用的命令。

可以通过 `docker-compose --help` 查看所有命令。其中很多命令都对应着 `docker` 的命令，不同之处一个是针对一组容器，一个是针对单个容器。


### 6.1 服务状态查看

#### ps

`docker-compose ps` 命令将列出所有服务的容器及状态信息：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1713timestamp1458795249846.png/wm)

#### run

`docker-compose run` 命令用来在指定的容器上执行一次指定的命令：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1713timestamp1458795296632.png/wm)

上述命令启动了web容器后进入到bash程序。使用 ps 命令查看：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1713timestamp1458795373978.png/wm)

一个新的容器被创建并执行了bash命令，同时该容器依赖的redis容器被自动启动。

#### logs

重新启动所有服务：`docker-compose start`，然后继续后续实验。

用来查看服务中的每个容器的 log 输出：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1713timestamp1458795429767.png/wm)


#### port

输出web容器80端口映射信息：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1713timestamp1458795591750.png/wm)

#### config

验证服务的配置信息：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1713timestamp1458795622708.png/wm)

### 6.2 服务生命周期管理

#### stop 与 rm

stop 停止服务并使用rm命令删除服务：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1713timestamp1458795796667.png/wm)

#### up -d

`-d` 参数可以创建并在后台运行服务中的所有容器：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1713timestamp1458795835308.png/wm)

#### restart 与 start

restart 重启服务，start 启动停止的服务：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1713timestamp1458795869647.png/wm)


#### kill

向服务发信号，kill 掉web服务：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1713timestamp1458795944114.png/wm)

#### pause 和 unpause

重新启动 web 服务，然后实验暂停和恢复服务：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1713timestamp1458796025268.png/wm)


### 6.3 服务扩展

`docker-compose scale` 命令可以用来创建指定数量的容器。

首先停止并删除先前创建的服务：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1713timestamp1458796215189.png/wm)

创建3个redis服务容器和1个web服务容器：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1713timestamp1458796255027.png/wm)


操作演示视频
`@
http://labfile.oss-cn-hangzhou.aliyuncs.com/courses/498/video/11-6.flv
@`

## 7. 总结

本节实验中我们学习了以下内容，任何不清楚的地方欢迎到[实验楼问答](https://www.shiyanlou.com/questions)与我们交流：

1. 安装 Docker Compose
2. 创建 Docker Compose 服务 - Web+redis网站
3. Docker Compose 服务管理

请务必保证自己能够动手完成整个实验，只看文字很简单，真正操作的时候会遇到各种各样的问题，解决问题的过程才是收获的过程。