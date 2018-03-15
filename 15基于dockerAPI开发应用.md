# 基于Docker API开发应用

## 1. 课程说明

课程为纯动手实验教程，为了能说清楚实验中的一些操作会加入理论内容。理论内容我们不会写太多，已经有太多好文章了，会精选最值得读的文章推荐给你，在动手实践的同时扎实理论基础。

学习过程中遇到的所有问题，都可随时在[实验楼问答](https://www.shiyanlou.com/questions)中提出，与老师和同学一起交流。

实验环境中可以联网，不受实验楼网络限制。

## 2. 学习方法

实验楼的Docker课程包含15个实验，每个实验都提供详细的步骤和截图，适用于有一定Linux系统基础，想快速上手Docker的同学。

学习方法是多实践，多提问。启动实验后按照实验步骤逐步操作，同时理解每一步的详细内容，如果有任何疑问，随时在[实验楼问答](https://www.shiyanlou.com/questions/)中提问，实验楼团队和我都会及时回复大家的所有问题。

如果实验开始部分有推荐阅读的材料，请务必先阅读后再继续实验，理论知识是实践必要的基础。

## 3. 本节内容简介

在本实验里我们将基于Docker API开发应用学习 Docker API。本实验将使用 Python 语言，应用尽量实现的简单，让不熟悉 Python 语言的同学也能够完成。

本节中，我们需要依次完成下面几项任务：

1. `docker-py` 安装与使用
2. 实现 `docker ps` 命令
3. 实现 `docker build` 命令


在实验之前，为了能够顺利连接 docker.io 我们使用阿里云的 Docker Hub 加速服务，在服务器上配置`/etc/default/docker`文件中的`DOCKER_OPTS`，然后再重启 Docker：

```
# 配置文件中添加 "--registry-mirror=https://n6syp70m.mirror.aliyuncs.com"
# 重启 Docker
$ sudo service docker restart
```

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1713timestamp1458794123429.png/wm)

## 4. 实验一： `docker-py` 安装与使用

本实验中将使用 `docker-py` 包进行实现。`docker-py` 是对 Docker Remote API 的 Python 封装，可以通过调用 Python 类和函数来实现 Docker Remote API 的操作，比如创建容器，镜像管理等。

`docker-py` 的详细文档见： <http://docker-py.readthedocs.org/en/latest/>

### 4.1 安装 `docker-py`

打开实验环境的 Xfce 终端，输入下面的命令进行安装：

```
sudo pip install docker-py==1.8.0
```

### 4.2 基本使用

安装完成后，我们可以在 python 中通过 `docker` 模块来使用。

`docker-py` 中的 `Client` 对象是用来连接和操作 Docker 服务或 Swarm 服务的基础类。

打开 Xfce 终端，输入 ipython，本节后续步骤在 ipython 下进行操作。

首先创建和初始化 `Client` 对象：


```
from docker import Client
cli = Client(base_url='tcp://127.0.0.1:4243')
```

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1717timestamp1459218544531.png/wm)

`cli` 是一个 `Client` 的实例，创建时需要输入连接的 URL 参数，如果连接本地可以使用默认的`unix://var/run/docker.sock` 地址，连接远程可以使用类似 `tcp://127.0.0.1:4243` 这样的地址，注意由于实验楼环境中的 Docker 配置文件修改过，没有监听在默认的2375端口，而是选择4243端口，所以在连接前请先查看 Docker 的配置文件 `/etc/default/docker`。

使用 `Client` 对象创建一个 ubuntu 容器：

```
container = cli.create_container(image='ubuntu:latest', command='/bin/bash')
container
```

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1717timestamp1459218997523.png/wm)

可以看到返回的是一个字典，包括容器的 ID 和 `Warnings` 警告信息，这也是 `Client` 创建操作通常的返回结果。

然后我们尝试列出当前系统的所有镜像：

```
images = cli.images()
images
```

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1717timestamp1459219028781.png/wm)

操作演示视频
`@
http://labfile.oss-cn-hangzhou.aliyuncs.com/courses/498/video/15-4.flv
@`

## 5. 实验二：实现 `docker ps` 命令

上一节中我们简单学习了 `docker-py` 的基本用法，如果对 Docker Remote API 或 docker 命令比较熟悉，应该很容易就可以根据文档学会 `docker-py` 的接口使用。

本节实验中我们将开发一个简单的工具，来查询并显示以下的容器信息，类似 `docker ps` 命令功能：

1. 容器的名称
2. 使用的镜像
3. 创建时间
4. ID
5. 进程信息
6. 运行状态
7. IP地址
8. 映射的端口
9. 网络读写状态

使用到的 API：

1. `cli.containers()` 获得容器列表及基本信息
2. `cli.inspect_container()` 类似`docker inspect`，获得容器详细的信息
3. `cli.top()` 类似 `docker top` 获得容器的进程执行信息
4. `cli.stats()` 类似 `docker stats` 获得容器的网络读写等状态信息

实验可以继续在 ipython 中执行，最后将执行无误的代码合并成 python 脚本。

为了本次实验，我们需要多创建几个容器：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1717timestamp1459219214325.png/wm)

所有的容器都是停止状态，我们启动其中的两个：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1717timestamp1459219299752.png/wm)

首先我们需要获得所有的容器的基本信息：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1717timestamp1459219359420.png/wm)

返回的 containers 列表中已经包含了我们所需的大部分信息，下一步是继续获取其他相关信息。

使用 `results` 字典作为返回信息的存储，使用其他接口获得数据 例如 `cli.inspect_container()` 等：

```
results = {}
for container in containers:
    id = container['Id']
    containerData = {}
    
    # 存储容器基本信息和inspect信息
    containerData['container'] = container
    containerData['inspect'] = cli.inspect_container(id)

    containerData['top'] = {}
    containerData['stats'] = {}
    
    # 判断容器是否在运行
    if container['Status'].startswith('Up'):
        containerData['top'] = cli.top(id)
        # stream 设置为False只有当前状态被返回
        containerData['stats'] = cli.stats(id, stream=False)
    results[id] = containerData
```

上述程序的输出：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1717timestamp1459222675577.png/wm)

循环输出程序所需的项目：

```
for id in results.keys():
    print '<<<'*20
    result = results[id]
    basic = result['container']
    inspect = result['inspect']
    top = result['top']
    stats = result['stats']
    print 'ID:', id
    print 'Name:', basic['Names']
    print 'Image:', basic['Image']
    print 'Created Time:', basic['Created']
    print 'IP:', inspect['NetworkSettings']['IPAddress']
    print 'Ports:', inspect['NetworkSettings']['Ports']
    print 'Status:', basic['Status']
    
    # 如果是运行状态才输出进程信息和网络读写状态
    if basic['Status'].startswith('Up'):
        print 'NetworkStats:'
        
        # 获得网络读写数据
        networkdict = stats['networks']['eth0']
        print '\t','\t'.join(networkdict.keys())
        print '\t','\t'.join([ str(value) for value in networkdict.values()])
        
        # 获得进程运行信息
        print 'Process:'
        print '\t','\t'.join(top['Titles'])
        for item in top['Processes']:
            print '\t','\t'.join(item)
    
    print '>>>'*20

```

输出的每个容器的信息：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1717timestamp1459223101373.png/wm)

上述代码块合并后就是最终的执行程序，本实验主要学习如何通过 `docker-py` 获取容器的信息，更多有用的信息需要通过查看 API 输出的结果来学习。

## 6. 实验三：实现 `docker build` 命令

本节实验中，我们将实现一个从 Dockerfile 或容器创建镜像的工具，类似 `docker build` 和 `docker commit` 命令。该工具通过输入不同的参数来执行不同的操作，如果输入参数为 Dockerfile 所在的路径则根据 Dockerfile 创建镜像，如果输入的参数是容器名称，则将容器的修改提交并创建一个新的镜像，因此该工具的输入参数定义：

1. Dockerfile 所在的路径或容器名称
2. 新镜像的TAG

本实验中将会用到的 API 接口为：

1. `cli.build()` `docker build` 对应的 API 调用
2. `cli.commit()` `docker commit` 对应的 API 调用
3. `cli.inspect_container()` 查询输入的容器名称是否有效

### 6.1 程序结构

我们将要实现的程序放在 `/home/shiyanlou/build-image.py`，使用 vim 或 gedit 打开该文件输入下面的内容，初始化 `Client` 对象：

```
from docker import Client

cli = Client(base_url='tcp://127.0.0.1:4243')
```

程序逻辑很简单，包含两部分内容：

1. 处理参数：判断是否输入的是两个参数，读取第一个参数并判断是否为容器名称或Dockerfile路径
2. 调用接口：如果参数为容器名，则停止容器并 commit 所有的修改到新的镜像，如果参数为路径，则调用 build 接口由 Dockerfile 创建新镜像


### 6.2 处理参数

判断是否输入了两个参数：

```
import sys

if len(sys.argv) != 3:
    print "Parameter ERROR:args number"
    sys.exit()
```

使用变量 buildFrom 来指代是从 Dockerfile 还是 容器 创建镜像，默认值为 `Dockerfile`：

```
buildFrom = 'Dockerfile'
validParam = False
```

判断第一个参数是否为路径并且路径存在 Dockerfile：

```
import os

if os.path.isdir(sys.argv[1]):
    dockerFile = os.path.join(sys.argv[1], 'Dockerfile')
    if os.path.exists(dockerFile):
        validParam = True
```

如果 validParam 仍然为 False，则继续判断输入的参数是不是容器名称：

```
if not validParam:
    try:
        info = cli.inspect_container(sys.argv[1])
        validParam = True
        buildFrom = 'Container'
    except:
        print "Parameter ERROR:container not found"
        sys.exit()
```

### 6.3 调用接口

根据 buildFrom 值进行判断来选择创建镜像的方法：

```
if buildFrom == 'Dockerfile':
    cli.build(path=sys.argv[1], tag=sys.argv[2], quiet=False)
elif buildFrom == 'Container':
    repo, tag = sys.argv[2].split(':')
    cli.commit(container=sys.argv[1], repository=repo, tag=tag, message='build image from container')
else:
    print "Parameter ERROR:buildFrom"
```

上面两个接口调用的参数说明：

1. path Dockerfile 所在的路径
2. tag 新建镜像的标签
3. quiet 是否输出build的过程信息
4. container 容器的名称
5. message docker commit 提交时的消息

需要注意的是 `cli.build()` 里 tag 可以包含 repository 的信息，而 `cli.commit()` 接口 repository 和 tag 则要分开写。

将上面所有的代码组合成一个完整的程序文件：

```
from docker import Client
import sys
import os

cli = Client(base_url='tcp://127.0.0.1:4243')

if len(sys.argv) != 3:
    print "Parameter ERROR:args number"
    sys.exit()
    
buildFrom = 'Dockerfile'
validParam = False


if os.path.isdir(sys.argv[1]):
    dockerFile = os.path.join(sys.argv[1], 'Dockerfile')
    if os.path.exists(dockerFile):
        validParam = True
if not validParam:
    try:
        info = cli.inspect_container(sys.argv[1])
        validParam = True
        buildFrom = 'Container'
    except:
        print "Parameter ERROR:container not found"
        sys.exit()

if buildFrom == 'Dockerfile':
    cli.build(path=sys.argv[1], tag=sys.argv[2], quiet=False)
elif buildFrom == 'Container':
    repo, tag = sys.argv[2].split(':')
    cli.commit(container=sys.argv[1], repository=repo, tag=tag, message='build image from container')
else:
    print "Parameter ERROR:buildFrom"
```

### 6.4 测试程序

创建测试所需的 Dockerfile 文件：

```
mkdir /home/shiyanlou/shiyanloutest
cd /home/shiyanlou/shiyanloutest
vim Dockerfile
```

输入 Dockerfile 的内容可以很简单：

```
FROM ubuntu:latest

ENV HOSTNAME=shiyanlou
```

创建测试所需的 Redis 容器：

```
docker run -d --name shiyanlouredis redis
```

从 Dockerfile 创建镜像 shiyanloutest:v1.0 ：

```
python build-image.py /home/shiyanlou/shiyanloutest shiyanloutest:v1.0
```

从 容器创建镜像 shiyanlouredis:v1.0：

```
python build-image.py shiyanlouredis shiyanlouredis:v1.0
```

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1717timestamp1459227053560.png/wm)

查看最终创建的镜像：

```
docker images
```

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1717timestamp1459227022193.png/wm)

本实验学习了几个新的 API 调用并复习了 Docker 创建镜像的两种方法。

## 7. 总结

本节实验中我们学习了以下内容，任何不清楚的地方欢迎到[实验楼问答](https://www.shiyanlou.com/questions)与我们交流：

1. `docker-py` 安装与使用
2. 实现 `docker ps` 命令
3. 实现 `docker build` 命令

请务必保证自己能够动手完成整个实验，只看文字很简单，真正操作的时候会遇到各种各样的问题，解决问题的过程才是收获的过程。