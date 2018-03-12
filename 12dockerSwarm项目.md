# Docker Swarm 项目

## 1. 课程说明

课程为纯动手实验教程，为了能说清楚实验中的一些操作会加入理论内容。理论内容我们不会写太多，已经有太多好文章了，会精选最值得读的文章推荐给你，在动手实践的同时扎实理论基础。

学习过程中遇到的所有问题，都可随时在[实验楼问答](https://www.shiyanlou.com/questions)中提出，与老师和同学一起交流。

实验环境中可以联网，不受实验楼网络限制。

## 2. 学习方法

实验楼的Docker课程包含15个实验，每个实验都提供详细的步骤和截图，适用于有一定Linux系统基础，想快速上手Docker的同学。

学习方法是多实践，多提问。启动实验后按照实验步骤逐步操作，同时理解每一步的详细内容，如果有任何疑问，随时在[实验楼问答](https://www.shiyanlou.com/questions/)中提问，实验楼团队和我都会及时回复大家的所有问题。

如果实验开始部分有推荐阅读的材料，请务必先阅读后再继续实验，理论知识是实践必要的基础。

## 3. 本节内容简介

在本实验里我们将通过实验学习 Docker Swarm 项目。Docker Swarm 是一个Docker集群部署和管理工具，可以把一组 Docker 服务器虚拟成一台容器服务器，并提供标准的 Docker API。所有可以直接连接 Docker 服务器的工具都可以连接 Docker Swarm。

由于实验楼只提供了一台实验机，所以无法搭建集群，有部分实验操作只能以单台服务器作为演示，请见谅。

本节中，我们需要依次完成下面几项任务：

1. 下载 Docker Swarm
2. 部署和管理 Swarm 集群

在开始实验之前，推荐先阅读下面的文章，了解Swarm的架构特点和概念：

1. [孙宏亮 - 深入浅出Docker Swarm](http://www.csdn.net/article/2015-01-26/2823714)

在实验之前，为了能够顺利连接 docker.io 我们使用阿里云的 Docker Hub 加速服务，在服务器上配置`/etc/default/docker`文件中的`DOCKER_OPTS`，然后再重启 Docker：


```
# 配置文件中添加 "--registry-mirror=https://n6syp70m.mirror.aliyuncs.com"
# 重启 Docker
$ sudo service docker restart
```

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1713timestamp1458794123429.png/wm)

## 4. 实验一：下载 Swarm

部署 Swarm 可以直接使用 Docker 的 Swarm 容器。这个 Swarm 容器既可以作为 Swarm 管理服务也可以运行在每个服务器节点上作为 Agent 服务。

从 Docker Hub pull swarm容器：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1714timestamp1458808496212.png/wm)

然后我们像执行一个命令那样操作 Swarm 容器来创建一个集群。


操作演示视频
`@
http://labfile.oss-cn-hangzhou.aliyuncs.com/courses/498/video/12-4.flv
@`

## 5. 实验二：部署 Swarm 集群

本节实验我们将用 Swarm 容器创建集群。默认情况下 Swarm 会使用 Docker Hub的发现服务：`discovery.hub.docker.com`，由于网络原因使用这个服务的话经常会出现节点信息无法找到的情况，因此我们替换成另外一种本地的 etcd 来提供信息发现的服务（Swarm还支持 Consul 和 Zookeeper等）。

### 5.1 安装 etcd

etcd 是一个服务注册和发现的工具。安装方法只是将二进制包下载下来放到系统路径下：

```
$ cd /home/shiyanlou
$ wget http://labfile.oss.aliyuncs.com/courses/498/etcd-v2.0.9-linux-amd64.tar.gz
$ tar zxvf etcd-v2.0.9-linux-amd64.tar.gz
$ cd etcd-v2.0.9-linux-amd64/
$ sudo mv etcd /usr/local/bin
$ sudo mv etcdctl /usr/local/bin
$ sudo chmod a+x /usr/local/bin/etcd
$ sudo chmod a+x /usr/local/bin/etcdctl
```

在启动 etcd 之前，我们先确定 etcd 监听的端口，可以使用 `ifconfig` 查看实验楼提供的主机有若干个接口都有IP地址，为了方便容器中连接，如果是多台服务器的集群，请选择其他服务器可以连接的网络接口，本实验中我们选择使用 Docker0 虚拟网桥的 IP 地址 `192.168.0.1`，保证每个 Docker 容器都可以连接。

打开一个Xfce终端来启动 etcd 服务，启动需要配置的必要参数：

```
$ export ETCD_IP=192.168.0.1
$ etcd --name etcd0 --initial-advertise-peer-urls http://$ETCD_IP:2380 \
                  --listen-peer-urls http://$ETCD_IP:2380 \
                  --listen-client-urls http://$ETCD_IP:2379 \
                  --advertise-client-urls http://$ETCD_IP:2379 \
                  --initial-cluster etcd0=http://$ETCD_IP:2380
```

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1714timestamp1458898918304.png/wm)

注意保持这个Xfce终端不要关闭，并且不退出 etcd。打开新的 XFce 终端继续下面的实验。

### 5.2 配置 Docker 守护进程

默认情况下 Docker 服务不提供远程连接，需要对`/etc/default/docker`文件进行修改，要配置 `--host=tcp://xxxx`。

实验楼的环境中已经配置了`--host=tcp://127.0.0.1:4243` ，但只支持本地的连接，为了能够让远程访问我们修改为`--host=0.0.0.0:4243`，注意4243端口也是实验楼修改的，并不是 Docker 默认的，修改后的配置文件：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1714timestamp1458899282994.png/wm)

注意修改后不要忘记重新启动 Docker 服务才会其效果：

```
$ sudo service docker restart
```

### 5.3 将服务器注册进集群

为了将服务器注册到集群中，需要在服务器上运行 swarm 容器并执行join命令加入到上面创建的集群中：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1714timestamp1458899390245.png/wm)

上面的命令需要在每一个加入集群的服务器节点上执行，其中的参数：

1. `--addr` 设置本地服务器的 Docker Engine 监听的地址
2. `etcd://xxxx/swarm` 表示 etcd 服务的地址，地址后的`/swarm`是一个前缀，用来表示该地址存储的是swarm集群的信息。

当这个命令执行后，Swarm将作为Agent运行在服务器节点上。可以通过`docker ps`查看到新的容器已经被创建。

由于实验楼环境限制，我们现在只有一台服务器节点，监听的地址也使用了本地`192.168.0.1`地址。

### 5.4 创建集群管理容器

当所有的服务器节点上都已经运行了 Swarm Agent后，我们创建管理容器 Swarm Manager 来进行中心化的管理，管理服务器可以选择运行在任意一个服务器节点上：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1714timestamp1458899495039.png/wm)

上面的命令中与5.3的操作类似，最大的不同是执行的 Swarm 命令是`manage`，etcd 仍然使用与上述命令相同的地址。`-H=0.0.0.0:2375` 表示个 Swarm Manager 监听的地址。

创建的管理容器把端口进行了映射，将容器的2375端口映射到了宿主机的8080端口。

现在我们的集群就已经创建完成，在这个集群里我们只有一台Docker服务器，Swarm Agent和Manager容器都运行在这一台服务器上。如果我们要添加其他的服务器，只需要在服务器上运行Swarm Agent容器，类似5.3的操作。


操作演示视频
`@
http://labfile.oss-cn-hangzhou.aliyuncs.com/courses/498/video/12-5.flv
@`

## 6. 实验三：使用 Docker Swarm 集群

当Swarm集群建立后，如果操作和使用呢？

### 6.1 列出服务器

使用 Swarm 的`list`命令可以列出当前集群中的服务器，注意仍然需要增加 etcd 参数：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1714timestamp1458899728290.png/wm)

### 6.2 配置 DOCKER_HOST

Swarm 提供的接口可以让 docker 命令直接连接并使用集群。只需要配置 DOCKER_HOST 环境变量为Swarm Manager容器提供服务的地址：

```
export DOCKER_HOST=tcp://127.0.0.1:8080
```

配置后我们执行 `docker info` 查看是否已经连接到了Swarm集群：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1714timestamp1458899810876.png/wm)

返回的信息中可以看到包含了集群中服务器Node的数量，Swarm的版本等信息，容器的数量和状态信息。

此时我们执行 `docker` 其他命令操作的不再是一台独立的服务器，而是一个集群。

这时候使用 `docker ps` 是看不到任何容器的，但 `docker ps -a` 可以看到 Docker Agent 和 Docker Manager。请思考下为什么会这样？

### 6.3 其他管理命令

#### 创建和查看容器

创建的命令与先前容器管理实验中学习的完全一样：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1714timestamp1458899980000.png/wm)

`docker ps` 输出的结果中除了容器名称外还包括了一个所在的节点的名字。

#### 查看其他资源

查看镜像列表：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1714timestamp1458900236321.png/wm)

其中镜像列表与单机情况没有区别，不过如果要使用 `docker pull` 的时候会把镜像 pull 到集群中的每一台服务器节点上。

查看Swarm集群中的网络：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1714timestamp1458900241588.png/wm)

现在列出的网络都包含节点的主机名，表示都是服务器上的本地网络，Swarm支持`overlay`的全局网络配置，实验楼的单机环境无法支持，大家可以阅读 [Swarm网络配置](https://docs.docker.com/swarm/networking/) 学习，有问题欢迎到[实验楼问答](https://www.shiyanlou.com/questions/3522)中交流。

### 6.4 扩展实验

更进一步的学习可以参考 [Swarm 官方文档](https://docs.docker.com/swarm/)。

其中比较关键的地方包括：

1. [为Swarm配置TLS](https://docs.docker.com/swarm/configure-tls/)
2. [Swarm Discovery策略](https://docs.docker.com/swarm/discovery/)
3. [Swarm 高可用性配置](https://docs.docker.com/swarm/multi-manager-setup/)


操作演示视频
`@
http://labfile.oss-cn-hangzhou.aliyuncs.com/courses/498/video/12-6.flv
@`

## 7. 总结

本节实验中我们学习了以下内容，任何不清楚的地方欢迎到[实验楼问答](https://www.shiyanlou.com/questions)与我们交流：

1. 安装 Docker Swarm
2. 部署和管理 Swarm 集群


请务必保证自己能够动手完成整个实验，只看文字很简单，真正操作的时候会遇到各种各样的问题，解决问题的过程才是收获的过程。