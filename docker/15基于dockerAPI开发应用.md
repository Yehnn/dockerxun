# 基于 Docker API 开发应用

## 1. 课程说明

课程为纯动手实验教程，为了能说清楚实验中的一些操作会加入理论内容。理论内容我们不会写太多，已经有太多好文章了，会精选最值得读的文章推荐给你，在动手实践的同时扎实理论基础。

实验环境中可以联网，不受实验楼网络限制。

## 2. 学习方法

实验楼的Docker课程包含 15个实验，每个实验都提供详细的步骤和截图，适用于有一定Linux系统基础，想快速上手Docker的同学。

学习方法是多实践，多提问。启动实验后按照实验步骤逐步操作，同时理解每一步的详细内容。

如果实验开始部分有推荐阅读的材料，请务必先阅读后再继续实验，理论知识是实践必要的基础。

## 3. 本节内容简介

在本实验里我们将基于 Docker API 开发应用学习 Docker API。本实验将使用 Python 语言，应用尽量实现的简单，让不熟悉 Python 语言的同学也能够完成。

本节中，我们需要依次完成下面几项任务：

1. `docker` 包的安装与使用
2. 获取容器信息
3. 使用 Dockerfile 创建镜像


对于 `Docker` 的镜像仓库来说，国内访问速度较慢，我们添加一个阿里云提供的 `Docker` 镜像加速器。

首先，我们需要添加编辑 `/etc/docker/daemon.json` 文件，加入如下内容：

```bash
{
  "registry-mirrors": ["https://n6syp70m.mirror.aliyuncs.com"]
}
```

修改之后，需要重启 `docker` 服务，让修改生效。使用如下命令：

```bash
$ sudo service docker restart
```

## 4. `docker` 包的安装与使用

本实验中将使用 `docker` 包进行实现。 `docker` 是对 Docker Remote API 的 Python 封装，可以通过调用 Python 类和函数来实现 Docker Remote API 的操作，比如创建容器，镜像管理等。

>  `docker` 包的详细文档见： <http://docker-py.readthedocs.org/en/latest/>  。其中包含的方法很多，要学会举一反三，学会使用官方文档。

### 配置虚拟环境

使用 `virtualenv` 配置虚拟环境：

```bash
$ cd ~
# 安装 virtualenv
$ sudo pip install virtualenv

# 使用 python3.4 ,venv 为虚拟环境目录名
$ virtualenv -p /usr/bin/python3.4 venv

# 激活虚拟环境
$ source venv/bin/activate
```

### 安装 ipython

```bash
$ pip install ipython
```

### 4.1 安装 `docker` 包

打开实验环境的 Xfce 终端，输入下面的命令进行安装：

```
pip install docker
```

### 4.2 基本使用

安装完成后，我们可以在 python 中通过 `docker` 模块来使用。

打开 Xfce 终端，输入 ipython，本节后续步骤在 ipython 下进行操作。

连接和操作 Docker 服务或 Swarm 服务的基础类需要首先实例化客户端：


```
import docker
client=docker.DockerClient(base_url='tcp://127.0.0.1:2375')
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521714956880.png-wm)

> 此处配置一个连接地址 `tcp://127.0.0.1:2375` 实例化一个客户端。

之前我们用 `curl` 命令调用 Docker Remote API 启动了 nginx 容器，这里我们用 python 实现同样效果。绑定主机的 `/home/shiyanlou/data` 目录和容器的 `/data` 目录，绑定主机端口 `84` 和容器端口 `80` ：

```python
container=client.containers.run(name='syl_nginx',image='nginx',volumes={'/home/shiyanlou/data':{'bind':'/data','mode':'rw'}},ports={'80/tcp':84},detach=True)
```

> `detach` 表示立即在后台启动容器，类似 `docker run -d` 。这里为 true，返回了容器对象。

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521772891008.png-wm)

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521772989578.png-wm)

接下来用浏览器访问地址 `localhost:84` 可看到 nginx 欢迎页面：

![实验楼](https://dn-simplecloud.shiyanlou.com/87971521773045986-wm)

然后我们尝试列出当前系统的所有镜像：

```
client.images.list()
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521773141826.png-wm)

## 5. 获取容器信息

上面我们简单学习了docker 包的基本用法，接下来学习如何获取容器信息。

使用到的 API：

1. `client.containers.list()` 获得容器列表及基本信息
2. 容器对象的各种属性

实验可以继续在 ipython 中执行，最后将执行无误的代码合并成 python 脚本。

为了本次实验，我们需要多创建几个容器：

```python
c1=client.containers.run(image='redis',detach=True)
c2=client.containers.create(image='nginx',volumes={'/home/shiyanlou/data':{'bind':'/data','mode':'rw'}},ports={'80/tcp':85})
```

nginx 使用的 create 创建，并没有启动，我们来启动它：

```python
c2.start()
```

首先我们需要获得所有的容器的基本信息：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521775587125.png-wm)

> `all = True` 代表列出所有容器，不加这个参数只会列出运行中的容器。

返回的 containers 列表中已经包含了我们所需的大部分信息，下一步是继续获取其他相关信息。

使用 `results` 字典作为返回信息的存储，获得容器的各种属性，如容器 id 前 10 个字符，容器的镜像，容器的运行进程，容器的流统计信息。

```
results={}

container_list=client.containers.list()

for container in container_list:
     results['short_id']=container.short_id
     results['image']=container.image
     results['top']=container.top()
     results['stats']=container.stats()
```

上述程序的输出：

![实验楼](https://dn-simplecloud.shiyanlou.com/87971521784976450-wm)

## 6. 使用 Dockerfile 创建镜像

学习了获取容器，接下来我们学习 python 的 docker 包从 dockerfile 创建镜像。需要调用 `client.images.buid()` 方法。

### 6.1 程序结构

我们将要实现的程序放在 `/home/shiyanlou/build-image.py`，使用 vim 或 gedit 打开该文件输入下面的内容，初始化 `Client` 对象：

```python
import docker

client=docker.DockerClient(base_url='tcp://127.0.0.1:2375')
```

程序逻辑很简单，包含两部分内容：

1. 处理参数：判断是否输入的是两个参数，读取第一个参数判断是否为路径并且路径下存在 Dockerfile。
2. 如果存在 Dockerfile，就调用创建镜像的 API 创建镜像。


### 6.2 处理逻辑

判断是否输入了两个参数：

```python
import sys

if len(sys.argv) != 3:
    print "Parameter ERROR:args number"
    sys.exit()
```

判断第一个参数是否为路径并且路径存在 Dockerfile，如果存在则创建镜像，如果不存在，就返回错误信息：

```python
import os

if os.path.isdir(sys.argv[1]):
	dockerFile = os.path.join(sys.argv[1],'Dockerfile')
	if os.path.exists(dockerFile):
	    #调用创建镜像的 API
		image=client.images.build(path=sys.argv[1],tag=sys.argv[2])
        #打印镜像信息
		print(image)
        #列出所有镜像
		print(client.images.list())
else:
	print("build error,not found Dockerfile")
```

> `path` ：Dockerfile 所在路径
>
> `tag` ：定义的镜像标签

完整的程序文件内容如下：

```
import sys
import os
import docker

client=docker.DockerClient(base_url='tcp://127.0.0.1:2375')

if len(sys.argv) != 3:
	print("Parameter ERROR:args number")
	sys.exit()

if os.path.isdir(sys.argv[1]):
	dockerFile = os.path.join(sys.argv[1],'Dockerfile')
	if os.path.exists(dockerFile):
		image=client.images.build(path=sys.argv[1],tag=sys.argv[2])
		print(image)
		print(client.images.list())
else:
	print("build error,not found Dockerfile")
```

### 6.4 测试程序

创建测试所需的 Dockerfile 文件：

```
$ mkdir /home/shiyanlou/imagetest
$ cd /home/shiyanlou/imagetest
$ vim Dockerfile
```

输入 Dockerfile 的内容可以很简单：

```
FROM ubuntu:latest

ENV HOSTNAME=shiyanlou
```

从 Dockerfile 构建容器：

```
$ python build-image.py /home/shiyanlou/imagetest build_test
```

![实验楼](https://dn-simplecloud.shiyanlou.com/87971521795675096-wm)

查看最终创建的镜像：

```
$ docker image ls
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521796489930.png-wm)

## 7. 总结

本节实验中我们学习了以下内容：

1. python `docker` 包的安装与使用
2. 获取容器信息
3. 使用 Dockerfile 创建镜像

请务必保证自己能够动手完成整个实验，只看文字很简单，真正操作的时候会遇到各种各样的问题，解决问题的过程才是收获的过程。