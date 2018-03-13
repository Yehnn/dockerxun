# Docker API

## 1. 课程说明

课程为纯动手实验教程，为了能说清楚实验中的一些操作会加入理论内容。理论内容我们不会写太多，已经有太多好文章了，会精选最值得读的文章推荐给你，在动手实践的同时扎实理论基础。

学习过程中遇到的所有问题，都可随时在[实验楼问答](https://www.shiyanlou.com/questions)中提出，与老师和同学一起交流。

实验环境中可以联网，不受实验楼网络限制。

## 2. 学习方法

实验楼的Docker课程包含15个实验，每个实验都提供详细的步骤和截图，适用于有一定Linux系统基础，想快速上手Docker的同学。

学习方法是多实践，多提问。启动实验后按照实验步骤逐步操作，同时理解每一步的详细内容，如果有任何疑问，随时在[实验楼问答](https://www.shiyanlou.com/questions/)中提问，实验楼团队和我都会及时回复大家的所有问题。

如果实验开始部分有推荐阅读的材料，请务必先阅读后再继续实验，理论知识是实践必要的基础。

## 3. 本节内容简介

Docker 提供的API非常丰富，包括 Registry API，Docker Hub API，Docker OAuth API，Docker Remote API，在本节实验中，我们将通过实验学习 Docker Remote API。Docker Remote API 可以提供 `docker` 命令的全部功能，实验中我们使用 curl 命令来进行测试和学习。

本节中，我们需要依次完成下面几项任务：

1. Docker API 基本概念与认证
2. 使用 API 管理容器：创建，查看，删除等操作
3. 使用 API 管理镜像：创建，查看，删除等操作
4. 使用 API 管理数据卷：创建，查看，删除等操作
5. 使用 API 管理网络：创建，查看，删除等操作

在实验之前，为了能够顺利连接 docker.io 我们使用阿里云的 Docker Hub 加速服务，在服务器上配置`/etc/default/docker`文件中的`DOCKER_OPTS`，然后再重启 Docker：

```
# 配置文件中添加 "--registry-mirror=https://n6syp70m.mirror.aliyuncs.com"
# 重启 Docker
$ sudo service docker restart
```

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1713timestamp1458794123429.png/wm)

后续实验中学习的是 Docker Remote API V1.22 版本，对应的是实验楼环境中的 Docker 1.10 版本。参考 API 官方文档设计实验：

+ [Docker Remote API v 1.22](https://docs.docker.com/engine/reference/api/docker_remote_api_v1.22/)

## 4. 实验一：Docker Remote API 基本概念与认证


### 4.1 监听地址

Docker Remote API 是由 Docker 守护进程提供的，默认情况下 `/etc/default/docker` 配置文件中 Docker 守护进程启动参数会有一个`-H=unix://var/run/docker.sock`，这表示 Docker 守护进程在绑定到本地的一个 Unix Socket 上，可以由本地连接访问 Remote API，如果要提供远程访问，则需要绑定到网络接口上。例如，我们通常会添加下面的配置：

```
-H=tcp://0.0.0.0:2375
```

这句表示将 Docker 守护进程监听到所有的网络接口的2375端口上。

我们修改后的配置文件 `/etc/default/docker` 如下所示：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1716timestamp1459150454338.png/wm)

不要忘记重启 Docker 服务让配置文件起作用： `sudo service docker restart`

重启后我们可以进行测试：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1716timestamp1459150490614.png/wm)

为了避免每次`docker` 命令都输入`-H`参数，我们设置环境变量`DOCKER_HOST`：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1716timestamp1459150530366.png/wm)

现在 `docker` 命令连接的就是 Docker Remote API 的接口。

### 4.2 使用 TLS 认证

在前面的 `Docker安全` 实验中，我们学习了如何用 TLS 保护 Docker 守护进程，过程与此处完全相同，经过 TLS 服务器端证书保护的 Docker Remote API，需要客户端采用同样CA签发的证书才可以连接。

### 4.3 Remote API 学习方法

后续的实验中，我们将学习最常用的 API 接口，这些API 我们将使用 curl 命令进行实验。

例如 `GET /info` API，需要在Xfce 终端中输入下面的命令：

```
$ curl http://127.0.0.1:2375/info
```

输出的结果如下，类似 `docker info` 命令的输出，不过是个 JSON 格式的内容，可以使用 `python -mjson.tool` 命令对输出信息进行美化：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1716timestamp1459150776305.png/wm)

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1716timestamp1459150782951.png/wm)


对于 `POST ` 操作，仍然可以用 curl 进行测试：

```
$ curl -X POST -H "Content-Type: application/json" \
  http://127.0.0.1:2375/containers/create \
  -d '{
        "Image": "redis"
  }'  
```

执行过程如下，我们会创建一个 redis 容器，并得到JSON格式的输出结果：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1716timestamp1459150891037.png/wm)

操作演示视频
`@
http://labfile.oss-cn-hangzhou.aliyuncs.com/courses/498/video/14-4.flv
@`

## 5. 实验二：使用 API 管理容器：创建，查看，删除等操作


### 5.1 查看所有容器

`GET /cont

实验需求是查找所有的容器（包含关机状态的容器），显示最后创建的一个，同时返回容器的大小。

针对这个需求我们将用到这个接口的三个参数：

1. all=1 所有的容器（包含关机状态的容器）
2. limit=1 显示最后创建的一个
3. size=1 返回容器的大小

执行的命令：

```
$ curl http://127.0.0.1:2375/containers/json\?all\=1\&limit\=1\&size\=1
```

注意：这里不加 \ 会报错

操作过程截图：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1716timestamp1459150967484.png/wm)

### 5.2 创建容器

`POST /containers/create`

创建容器的参数非常多，可以回忆下`docker run`的参数。

实验需求创建一个 nginx 容器，将容器的80端口映射到宿主机80端口，挂载宿主机的 `/home/shiyanlou/data` 目录作为数据卷到容器中的`/data`目录。

该接口的所有参数都使用 JSON 格式。

定义输入的参数：

```
{
    "Image": "nginx",
    "Mounts": [
        {
            "Source": "/home/shiyanlou/data",
            "Destination": "/data"
        }
    ],
    "HostConfig": {
        "PortBindings": { "80/tcp": [{ "HostPort": "80" }] }
    }
}
```

操作过程：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1716timestamp1459153021567.png/wm)

创建后使用 `docker inspect` 验证。

### 5.3 删除指定的容器

`DELETE /containers/(id)`

此处我们尝试删除一个运行中的容器，需要使用参数`force=1`。

操作过程首先查找容器ID，然后使用 curl 执行 DELETE 操作：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1716timestamp1459153142948.png/wm)

### 5.4 其他接口

其他常用的接口使用方法，主要参数都是容器的ID，下面列出常用 docker 命令对应的 API：

1. `docker inspect`：`GET /containers/(id)/json`
2. `docker top`：`GET /containers/(id)/top`
3. `docker logs`：`GET /containers/(id)/logs`
4. `docker export`：`GET /containers/(id)/export`
5. `docker start`：`POST /containers/(id)/start`
6. `docker attach`：`POST /containers/(id)/attach`

可以参照上面三个操作进行实验。

## 6. 实验三：使用 API 管理镜像：创建，查看，删除等操作

### 6.1 查看所有镜像

`GET /images/json`

查看当前系统中的所有镜像：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1716timestamp1459153213483.png/wm)

### 6.2 拉取镜像

`POST /images/create`

从 Docker Hub 拉取 busybox 镜像：

```
curl -X POST http://127.0.0.1:2375/images/create?fromImage=busybox
```

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1716timestamp1459153317873.png/wm)

该接口执行的过程中，镜像下载的进度也会输出到屏幕上。

### 6.3 删除镜像

`DELETE /images/(name)`

删除刚刚拉取的busybox镜像：

```
curl -X DELETE http://127.0.0.1:2375/images/busybox
```

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1716timestamp1459153404158.png/wm)

### 6.4 其他接口


其他常用的接口使用方法，主要参数都是容器的ID，下面列出常用 docker 命令对应的 API：

1. `docker inspect`：`GET /images/(name)/json`
2. `docker tag`：`POST /images/(name)/tag`
3. `docker push`: `POST /images/(name)/push`
4. `docker build`：`POST /build`
5. `docker search`：`GET /images/search`

操作演示视频
`@
http://labfile.oss-cn-hangzhou.aliyuncs.com/courses/498/video/14-6.flv
@`

## 7. 实验四：使用 API 管理数据卷：创建，查看，删除等操作

### 7.1 查看数据卷

`GET /volumes`

查看系统中的所有数据卷：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1716timestamp1459153445304.png/wm)

### 7.2 创建数据卷

`POST /volumes/create`

创建一个名字为shiyanlou的数据卷，可以在返回信息中看到数据卷的挂载点

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1716timestamp1459153615772.png/wm)

### 7.3 删除数据卷

`DELETE /volumes/(name)`

删除刚刚创建的数据卷shiyanlou:

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1716timestamp1459153611072.png/wm)

操作演示视频
`@
http://labfile.oss-cn-hangzhou.aliyuncs.com/courses/498/video/14-7.flv
@`

## 8. 实验五：使用 API 管理网络：创建，查看，删除等操作

### 8.1 列出系统中所有的网络

`GET /networks`

列出系统中所有的网络:

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1716timestamp1459153658834.png/wm)

### 8.2 创建新的网络

`POST /networks/create`

创建一个新的网络，名字为shiyanlou，驱动类型为`bridge`，配置网段为`172.10.0.0/16`。

创建过程：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1716timestamp1459153884028.png/wm)

### 8.3 连接容器到网络

`POST /networks/(id)/connect`

创建一个redis容器：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1716timestamp1459154187545.png/wm)

连接容器到新创建的shiyanlou网络：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1716timestamp1459154155538.png/wm)

### 8.4 删除网络

`DELETE /networks/(id)`

首先把关联的容器断开，然后删除网络：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1716timestamp1459154148073.png/wm)

操作演示视频
`@
http://labfile.oss-cn-hangzhou.aliyuncs.com/courses/498/video/14-8.flv
@`

## 9. 总结

本节实验中我们学习了以下内容，任何不清楚的地方欢迎到[实验楼问答](https://www.shiyanlou.com/questions)与我们交流：

1. Docker API 基本概念与认证
2. 使用 API 管理容器：创建，查看，删除等操作
3. 使用 API 管理镜像：创建，查看，删除等操作
4. 使用 API 管理数据卷：创建，查看，删除等操作
5. 使用 API 管理网络：创建，查看，删除等操作

请务必保证自己能够动手完成整个实验，只看文字很简单，真正操作的时候会遇到各种各样的问题，解决问题的过程才是收获的过程。