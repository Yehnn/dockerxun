# Kubernetes

## 1. 课程说明

课程为纯动手实验教程，为了能说清楚实验中的一些操作会加入理论内容。理论内容我们不会写太多，已经有太多好文章了，会精选最值得读的文章推荐给你，在动手实践的同时扎实理论基础。

实验环境中可以联网，不受实验楼网络限制。

## 2. 学习方法

实验楼的 Docker 课程包含 15 个实验，每个实验都提供详细的步骤和截图，适用于有一定 Linux 系统基础，想快速上手 Docker 的同学。

学习方法是多实践，多提问。启动实验后按照实验步骤逐步操作，同时理解每一步的详细内容。

如果实验开始部分有推荐阅读的材料，请务必先阅读后再继续实验，理论知识是实践必要的基础。

## 3. 本节内容简介

在本实验里我们将通过实验学习 Kubernetes 项目。Kubernetes 是谷歌开源的一个 Docker 集群管理工具，支持为Docker 容器提供自动扩展，资源调度，服务发现等功能。

Kubernetes 的核心概念：

1. Cluster：一组物理机或虚拟机及其他资源构成的资源池。
2. Node：指的是一个运行 Kubernetes 的物理机或虚拟机节点
3. Pod：最基本的部署单位，对应一个应用的实例，包含若干个容器。
4. Replication controller：管理 Pod 的生命周期，控制Pod的扩容缩容。
5. Service：类似一个基本的负载均衡的路由代理，为 Pod 提供唯一的地址命名，解决服务发现问题。
6. Label：基于key:value对来组织对象组。

Kubernetes 集群的主要架构是一个中心化的 master。

由于实验楼只提供了一台实验机，所以无法搭建集群，实验操作只能以单台服务器作为演示，请见谅。

Kubernetes 的知识点非常多，可以写成一本很厚的书，所以本节实验的定位仅仅是帮助你搭建一个 Kubernetes 集群环境并创建一个多容器的Pod。更深入的学习，可以参考详细的官方教程，好在 Kubernetes 除了安装教程在国内用起来问题比较多外，其他教程操作性非常好。可以好好利用实验楼的环境进行学习。

本节中，我们需要依次完成下面几项任务：

1. 部署单节点的 Kubernetes 集群
2. 在集群上基于 Pod 创建多个容器

在开始实验之前，推荐先阅读下面的文章，了解 Kubernetes 的架构特点和概念：

+ [杨章显 - Kubernetes系统架构简介](http://www.infoq.com/cn/articles/Kubernetes-system-architecture-introduction)


**注意：** Kubernetes 是一个很强大的系统，本节实验仅仅从基本的用户角度学习最常用的功能，从而帮助你入门 Kubernetes 系统，但一些集群管理的细节仍然需要在使用时查询官方文档，或者利用实验楼的环境详细实践一遍官方的文档。见 [Kubernetes 官方文档](http://kubernetes.io/docs/)。


在实验之前，为了能够顺利连接 docker.io 我们使用阿里云的 Docker Hub 加速服务，在服务器上配置`/etc/default/docker`文件中的`DOCKER_OPTS`，然后再重启 Docker：


```
# 配置文件中添加 DOCKER_OPTS="--registry-mirror=https://n6syp70m.mirror.aliyuncs.com -host=tcp://127.0.0.1:4243 -H=unix:///var/run/docker.sock"
# 重启 Docker
$ sudo service docker restart
```

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1713timestamp1458794123429.png/wm)

## 4. 实验一：部署单节点的 Kubernetes 集群

### 4.1 下载镜像

安装 Kubernetes 的方法很多，最简单的方式是直接使用 Docker 容器，实验楼的环境已经提供了一个 Docker 服务器，我们在这个服务器上部署一个单节点的 Kubernetes 集群。

最终部署的结构图（来自 Kubernetes 官方文档）：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1715timestamp1458885850715.png/wm)

我们将要安装的 Kubernetes 版本是 1.6.0 的 alpha 版本。由于谷歌的服务器国内网络无法连接，我们选择了在Docker Hub上的一个 Kubernetes 镜像。

安装方法就是启动一个 Kubernetes 容器，如上图所示，这个容器中包含了所有必要的组件：etcd，master，kubelet 等等。

这个容器的镜像是`hyperkube`，启动过程中会不断连接到 google 的服务器获取所需的镜像 etcd 和 pause，为了避免因为网络原因造成的安装失败，我们需要从国内时速云提供的镜像处下载相应镜像后伪装成google 网站的镜像，伪装的方法很简单就是做 TAG。

下载所需的镜像：

```
docker pull index.tenxcloud.com/google_containers/hyperkube:v1.6.0-alpha.0
docker pull index.tenxcloud.com/google_containers/etcd:2.2.5
docker pull index.tenxcloud.com/google_containers/pause:3.0
```

为下载的镜像打上google原版镜像的标签：

```dockerfile
docker tag index.tenxcloud.com/google_containers/hyperkube:v1.6.0-alpha.0 gcr.io/google_containers/hyperkube:v1.6.0-alpha.0
docker tag index.tenxcloud.com/google_containers/etcd:2.2.5 gcr.io/google_containers/etcd:2.2.5
docker tag index.tenxcloud.com/google_containers/pause:3.0 gcr.io/google_containers/pause:3.0
```

我们下载的镜像列表：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1715timestamp1459145546328.png/wm)


### 4.2 启动Kubernetes容器

启动命令：

```
docker run \
    --volume=/:/rootfs:ro \
    --volume=/sys:/sys:ro \
    --volume=/var/lib/docker/:/var/lib/docker:rw \
    --volume=/var/lib/kubelet/:/var/lib/kubelet:rw \
    --volume=/var/run:/var/run:rw \
    --net=host \
    --pid=host \
    --privileged=true \
    --name=kubelet \
    -d \
    gcr.io/google_containers/hyperkube:v1.6.0-alpha.0 \
    /hyperkube kubelet \
        --containerized \
        --hostname-override="127.0.0.1" \
        --address="0.0.0.0" \
        --api-servers=http://localhost:8080 \
        --config=/etc/kubernetes/manifests \
        --cluster-dns=10.0.0.10 \
        --cluster-domain=cluster.local \
        --allow-privileged=true --v=2
```

上面的参数包含两部分：

1. Docker 容器启动参数，包含对Kubernetes容器的各种权限和网络设置，以及挂载宿主机上的一系列目录
2. Kubernetes 参数，主要是网络设置，监听的端口设置，其中的cluster-dns 和 cluster-domain 都是用来设置集群的DNS。

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1715timestamp1459145595933.png/wm)

启动的过程需要一定时间，并会创建若干个pause，etcd，apiserver等容器。最终当apiserver容器启动后，整个Kubernetes才算完全启动，可以通过 `docker ps` 查看启动的所有容器。

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1715timestamp1459145796546.png/wm)

### 4.3 安装客户端


启动后我们如果要管理 Kubernetes 集群，还需要一个命令行工具 kubectl，下载并拷贝到执行路径：

```
$ cd /home/shiyanlou
$ wget http://labfile.oss.aliyuncs.com/courses/498/kubectl
$ sudo mv kubectl /usr/local/bin
$ sudo chmod a+x /usr/local/bin/kubectl
```

创建初始化的配置：

```
$ kubectl config set-cluster shiyanlou-cluster --server=http://localhost:8080
$ kubectl config set-context shiyanlou-cluster --cluster=shiyanlou-cluster
$ kubectl config use-context shiyanlou-cluster
```

现在我们的Kubernetes集群已经部署完成，集群名称为 shiyanlou-cluster。

查看当前集群中的所有 Node：

```
kubectl get nodes
```

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1715timestamp1459145871190.png/wm)

可以发现我们的 Kubernetes 集群中只有一个节点。

在集群上运行一个容器：

```
kubectl run nginx --image=nginx --port=80
docker ps
```

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1715timestamp1459145973525.png/wm)

查看帮助，我们可以在实验楼的环境上尝试非常多的命令，可以结合 `docker ps` 和 `docker inspect` 来对比查看 Kubernetes 是如何部署 Docker 容器的：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1715timestamp1459146127732.png/wm)

### 4.4 基本信息与API使用

查看 Kubernetes 的配置信息：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1715timestamp1459149542585.png/wm)

其中`http://localhost:8080` 就是 API 服务监听的地址。Kubernetes 详细的接口文档可见[http://kubernetes.io/docs/api/](http://kubernetes.io/docs/api/)。

可以使用 curl 命令根据 API 文档对接口进行调用实验：

```
curl http://localhost:8080/api
```

### 4.5 卸载 Kubernetes

**注意：本步骤可以最后操作，不要轻易把好不容易创建的集群删掉**

删除当前的 Kubernetes，只需要使用 `docker rm` 清除所有容器即可，特别注意如果你有需要保存的容器的话不要这么用：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1715timestamp1459143576142.png/wm)


操作演示视频
`@
http://labfile.oss-cn-hangzhou.aliyuncs.com/courses/498/video/13-4.flv
@`

## 5. 实验二：在集群上基于Pod创建多个容器

本节内容我们将基于 Kubernetes，创建一个具有两个容器的 Pod。这个应用在先前 Compose 的实验中已经学习过了，是一个 Python Flask 的Web应用，使用 Redis 作为数据后端。

### 5.1 开发应用

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

这个程序很简单，哪怕不动 Python 也没有关系，简单的说明下程序的作用：

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

这时候我们可以使用 `docker build` 创建 web 镜像：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1715timestamp1459146563933.png/wm)

### 5.3 创建Pod

Pod 的概念类似我们先前 Compose 中的服务，也可以通过编写 Yaml 格式的配置文件来创建一个 Pod。

配置文件中需要详细描述该应用对应的每个容器的镜像及配置，我们在本实验中需要创建一个具备两个容器的 Web 服务，第一个容器是 python 容器，第二个是 redis 容器。首先完成描述文件：

```
apiVersion: v1
kind: Pod
metadata:
    labels:
        name: shiyanlouweb
    name: shiyanlouweb
spec:
    containers:
    - name: shiyanlou
      image: shiyanlou:v1.0
      ports:
      - containerPort: 80
        hostPort: 80
    - name: redis
      image: redis
      ports:
      - containerPort: 6379
```

配置文件保存到 `/home/shiyanlou/shiyanlou.yaml`。

使用 `kubectl` 创建 Pod：

```
kubectl create -f /home/shiyanlou/shiyanlou.yaml
```

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1715timestamp1459147373383.png/wm)


查看当前集群中的所有 Pod：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1715timestamp1459147433366.png/wm)

这个时候我们可以访问`http://localhost` 来查看这个应用：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1715timestamp1459148184929.png/wm)

发现应用是无法连接redis的，因为通过上面的方法没有实现 Docker 的`--link` 动态链接参数。因为 Kubernete 集群中 Pod 包含的多个容器有可能运行在不同的服务器节点上，并且 redis 服务有可能有多台，那么单机上的 `--link` 就失去了意义。如何重新设计让我们的应用可以起作用呢？一种简单的方案是可以为 Kubernetes 集群配置 DNS 服务，根据 Pod 配置文件为容器生成其他容器可访问的域名。

上面的配置文件是一个非常简单的Pod配置，详细的配置实例可见：[http://kubernetes.io/docs/user-guide/pods/multi-container/](http://kubernetes.io/docs/user-guide/pods/multi-container/)

### 5.4 查看Pod信息

查看Pod的详细信息：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1715timestamp1459149144148.png/wm)

其中包括：

1. Pod基本信息，包括IP地址，镜像，创建时间等
2. Pod中的所有容器运行状态
3. Pod中的 Volume 信息
4. Pod中的事件

### 5.5 删除Pod

删除名字为 shiyanlouweb 的Pod：

```
kubectl delete pod shiyanlouweb
```

操作演示视频
`@
http://labfile.oss-cn-hangzhou.aliyuncs.com/courses/498/video/13-5.flv
@`

## 8. 总结

本节实验中我们学习了以下内容：

1. 部署单节点的 Kubernetes 集群
2. 在集群上基于Pod创建多个容器

请务必保证自己能够动手完成整个实验，只看文字很简单，真正操作的时候会遇到各种各样的问题，解决问题的过程才是收获的过程。