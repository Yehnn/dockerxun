# Docker API

## 1. 课程说明

课程为纯动手实验教程，为了能说清楚实验中的一些操作会加入理论内容。理论内容我们不会写太多，已经有太多好文章了，会精选最值得读的文章推荐给你，在动手实践的同时扎实理论基础。

实验环境中可以联网，不受实验楼网络限制。

## 2. 学习方法

实验楼的Docker课程包含15个实验，每个实验都提供详细的步骤和截图，适用于有一定Linux系统基础，想快速上手Docker的同学。

学习方法是多实践，多提问。启动实验后按照实验步骤逐步操作，同时理解每一步的详细内容。

如果实验开始部分有推荐阅读的材料，请务必先阅读后再继续实验，理论知识是实践必要的基础。

## 3. 本节内容简介

Docker 提供的 API 非常丰富，包括 Registry API，Docker Hub API，Docker OAuth API，Docker Remote API，在本节实验中，我们将通过实验学习 Docker Remote API。Docker Remote API 可以提供 `docker` 命令的全部功能，实验中我们使用 curl 命令来进行测试和学习。

本节中，我们需要依次完成下面几项任务：

1. Docker API 基本概念与认证
2. 使用 API 管理容器：创建，查看，删除等操作
3. 使用 API 管理镜像：创建，查看，删除等操作
4. 使用 API 管理数据卷：创建，查看，删除等操作
5. 使用 API 管理网络：创建，查看，删除等操作

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

查看 Docker API 的版本：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521542295486.png-wm)

## 4. Docker Remote API 基本概念与认证


### 4.1 监听地址

Docker Remote API 是由 Docker 守护进程提供的，默认情况下 Docker 守护进程可以由本地连接访问 Remote API，如果要提供远程访问，则需要绑定到网络接口上。例如，我们通常会添加下面的配置：

```
-H=tcp://0.0.0.0:2375
```

这句表示将 Docker 守护进程监听到所有的网络接口的 2375 端口上。

修改配置文件 `/etc/default/docker` ：

```bash
$ sudo vi /etc/default/docker
```

![实验楼](https://dn-simplecloud.shiyanlou.com/87971521539684174-wm)

> `-H=unix:///var/run/docker.sock` 是允许本地访问连接。
>
> 不要忘记重启 Docker 服务让配置文件起作用： `sudo service docker restart`

重启后我们可以进行测试：			

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1716timestamp1459150490614.png/wm)

为了避免每次`docker` 命令都输入`-H`参数，我们设置环境变量`DOCKER_HOST`：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1716timestamp1459150530366.png/wm)

现在 `docker` 命令连接的就是 Docker Remote API 的接口。

### 4.2 使用 TLS 认证

在前面的 `Docker安全` 实验中，我们学习了如何用 TLS 保护 Docker 守护进程，过程与此处完全相同，经过 TLS 服务器端证书保护的 Docker Remote API，需要客户端采用同样 CA 签发的证书才可以连接。

### 4.3 Remote API 使用方法

后续的实验中，我们将学习最常用的 API 接口，我们将使用 `curl` 命令进行实验。

例如 `GET /info` API，需要在 Xfce 终端中输入下面的命令：

```
$ curl http://127.0.0.1:2375/info
```

输出的结果如下，类似 `docker info` 命令的输出，不过是个 JSON 格式的内容，可以使用 `python -mjson.tool` 命令对输出信息进行美化：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1716timestamp1459150776305.png/wm)

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1716timestamp1459150782951.png/wm)

也可以打开 firefox 浏览器，输入如下地址然后回车查看到返回的 json 数据：

![实验楼](https://dn-simplecloud.shiyanlou.com/87971521541061909-wm)

对于 `POST ` 操作，仍然可以用 curl 进行测试：

```
$ curl -X POST -H "Content-Type: application/json" \
  http://127.0.0.1:2375/containers/create \
  -d '{
        "Image": "redis"
  }'
```

> 如果提示没有镜像的话，可以先使用 docker pull redis 拉取镜像。

执行过程如下，我们会创建一个 redis 容器，并得到 JSON 格式的输出结果：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521614320222.png-wm)

## 5. 使用 API 管理容器：创建，查看，删除等操作

### 5.1 查看所有容器

`GET /containers/json` 

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

类比命令：

```
$ docker container ls -a -n 1 -s
```

操作过程截图：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1716timestamp1459150967484.png/wm)

### 5.2 创建容器

`POST /containers/create`

创建容器的参数非常多，可以回忆下 `docker run` 的参数。

实验需求创建一个 nginx 容器，将容器的 80 端口映射到宿主机 80 端口，挂载宿主机的 `/home/shiyanlou/data` 目录作为数据卷到容器中的 `/data ` 目录。

该接口的所有参数都使用 JSON 格式。

定义输入的参数：

```
$ curl -X POST -H "Content-Type: application/json" \
http://127.0.0.1:2375/containers/create?name=test_nginx \
-d '{
    "Image": "nginx",
    "HostConfig": {
        "Binds": ["/home/shiyanlou/data:/data"],
        "PortBindings": {"80/tcp": [{"HostPort": "81"}]}
    }
}'
```

操作过程：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521703653568.png-wm)

创建后使用 `docker container inspect` 验证。

此时用浏览器访问地址 `localhost:81` 可以看到 nginx 的页面：

![实验楼](https://dn-simplecloud.shiyanlou.com/87971521703795744-wm)

### 5.3 删除指定的容器

`DELETE /containers/(id)`

如果尝试删除一个运行中的容器，需要使用参数`force=1`。

操作过程首先查找容器ID，然后使用 curl 执行 DELETE 操作：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521704169448.png-wm)

### 5.4 其他接口

其他常用的接口使用方法，主要参数都是容器的ID，下面列出常用 docker 命令对应的 API：

1. `docker container inspect`：`GET /containers/(id)/json`
2. `docker container top`：`GET /containers/(id)/top`
3. `docker container logs`：`GET /containers/(id)/logs`
4. `docker container export`：`GET /containers/(id)/export`
5. `docker container start`：`POST /containers/(id)/start`
6. `docker container attach`：`POST /containers/(id)/attach`

可以参照上面三个操作进行实验。

## 6. 使用 API 管理镜像：创建，查看，删除等操作

### 6.1 查看所有镜像

`GET /images/json`

查看当前系统中的所有镜像：

```bash
$ curl http://127.0.0.1:2375/images/json | python -mjson.tool 
```

![实验楼](https://dn-simplecloud.shiyanlou.com/87971521704613733-wm)

### 6.2 拉取镜像

`POST /images/create`

从 Docker Hub 拉取 busybox 镜像：

```
$ curl -X POST http://127.0.0.1:2375/images/create\?fromImage\=busybox:ubuntu-14.04
```

![实验楼](https://dn-simplecloud.shiyanlou.com/87971521706974381-wm)

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

1. `docker image inspect`：`GET /images/(name)/json`
2. `docker image tag`：`POST /images/(name)/tag`
3. `docker image push`: `POST /images/(name)/push`
4. `docker image build`：`POST /build`
5. `docker search`：`GET /images/search`

## 7. 使用 API 管理数据卷：创建，查看，删除等操作

### 7.2 创建数据卷

`POST /volumes/create`

创建一个名字为 shiyanlou 的数据卷，可以在返回信息中看到数据卷的挂载点

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521705366146.png-wm)

### 7.1 查看数据卷

`GET /volumes`

查看系统中的所有数据卷：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521705472113.png-wm)

### 7.3 删除数据卷

`DELETE /volumes/(name)`

删除刚刚创建的数据卷 shiyanlou：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521705583679.png-wm)

## 8. 使用 API 管理网络：创建，查看，删除等操作

### 8.1 列出系统中所有的网络

`GET /networks`

列出系统中所有的网络:

![实验楼](https://dn-simplecloud.shiyanlou.com/87971521705724622-wm)

### 8.2 创建新的网络

`POST /networks/create`

创建一个新的网络，名字为shiyanlou，驱动类型为`bridge`，配置网段为`172.10.0.0/16`。

创建过程：

```bash
$ curl -X POST -H "Content-Type: application/json" \
http://127.0.0.1:2375/networks/create \
-d '{
    "Name": "shiyanlou",
    "Driver": "bridge",
    "IPAM": {"Config": [{"Subnet": "172.10.0.0/16"}]}
}'
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521707347839.png-wm)

### 8.3 连接容器到网络

`POST /networks/(id)/connect`

创建一个 redis 容器：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521708131392.png-wm)

连接容器到新创建的shiyanlou网络：

```bash
curl -X POST -H "Content-Type: application/json" http://127.0.0.1:2375/networks/shiyanlou/connect -d '{"Container": "e132d"}'
```

> e132d 为 容器 id。

### 8.4 删除网络

`DELETE /networks/(id)`

首先把关联的容器断开，然后删除网络：

```bash
$ curl -X POST -H "Content-Type: application/json" http://127.0.0.1:2375/networks/shiyanlou/disconnect -d '{"Container": "e132d"}'

$ curl -X DELETE http://127.0.0.1:2375/networks/shiyanlou
$ curl http://127.0.0.1:2375/networks/shiyanlou
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521708475096.png-wm)

## 9. 总结

本节实验中我们学习了以下内容：

1. Docker API 基本概念与认证
2. 使用 API 管理容器：创建，查看，删除等操作
3. 使用 API 管理镜像：创建，查看，删除等操作
4. 使用 API 管理数据卷：创建，查看，删除等操作
5. 使用 API 管理网络：创建，查看，删除等操作

请务必保证自己能够动手完成整个实验，只看文字很简单，真正操作的时候会遇到各种各样的问题，解决问题的过程才是收获的过程。