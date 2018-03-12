# 网络管理

## 1. 课程说明

课程为纯动手实验教程，为了能说清楚实验中的一些操作会加入理论内容。理论内容我们不会写太多，已经有太多好文章了，会精选最值得读的文章推荐给你，在动手实践的同时扎实理论基础。

实验环境中可以联网，不受实验楼网络限制。

## 2. 学习方法

实验楼的Docker课程包含15个实验，每个实验都提供详细的步骤和截图，适用于有一定Linux系统基础，想快速上手Docker的同学。

学习方法是多实践，多提问。启动实验后按照实验步骤逐步操作，同时理解每一步的详细内容。

如果实验开始部分有推荐阅读的材料，请务必先阅读后再继续实验，理论知识是实践必要的基础。

## 3. 本节内容简介

Docker中的网络一直是比较弱的，在较早期的版本中只提供最基本的网络通信支持，比如下面这三种场景最常见：

1. 使用NAT方式连接外部网络
2. 映射容器和宿主机的端口，使外部可以访问容器中的应用
3. 容器间网络互联

本节中，我们需要依次完成下面几项任务：

1. Docker 网络基本配置
2. Docker 网络访问控制
3. Docker 容器端口映射
4. Docker 容器互联

本实验中我们需要改变 Docker 默认的网桥，网段，并阻止容器间互联，同时允许容器对外网访问。这是本次实验的基本需求。

## 3. 网络

> 在开始下面的内容之前，为了不出现命名上的冲突，也为了显示更为直观并且方便演示示例，首先需要将前面创建或启动的容器全部删除。可以使用下面两条命令达到这一效果：

```bash
# 暂停所有运行中的容器
$ docker container ls -q | xargs docker container stop

# 删除所有的容器
$ docker container ls -aq | xargs docker container rm
```

在我们安装 `Docker` 后，会自动创建三个网络。我们可以使用下面的命令来查看这些网络：

```bash
$ docker network ls
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516712155589.png/wm)

如上图所示，三种默认的网络，分别为 `bridge`，`host`，`none`。

### 3.1 bridge

`bridge`，即桥接网络，在安装 `docker` 后会创建一个桥接网络，该桥接网络的名称为 `docker0`。我们可以通过下面两条命令去查看该值。

```bash
# 查看 bridge 网络的详细信息，并通过 grep 获取名称项
$ docker network inspect bridge | grep name

# 使用 ifconfig 查看 docker0 网络
ifconfig
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516775691321.png/wm)

在上图中，我们可以查看到对应的值。默认情况下，我们创建一个新的容器都会自动连接到 `bridge` 网络。其详细信息如下所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516778041875.png/wm)

我们可以尝试创建一个容器，该容器会自动连接到 `bridge` 网络，例如我们创建一个名为 `shiyanlou001` 的容器：

```bash
$ docker container run -itd --name shiyanlou001 ubuntu /bin/bash

上述命令中默认使用 --network bridge ，即指定 bridge 网络，与下面的命令等同
$ docker container run -itd --name shiyanlou001 --network bridge ubuntu /bin/bash
```

创建后，再次查看 `bridge` 的信息：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516778620559.png/wm)

这时可以查看到相应的容器的网络信息，该容器在连接到 `bridge` 网络后，会从子网的地址池中获得一个 IP 地址，即上图中的 `192.168.0.2`。

使用 `docker container attach shiyanlou001` 命令，也可查看相应的地址信息：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516778879129.png/wm)

> 如果提示没有找到 `ifconifg` 命令，可以通过如下命令安装：
>
> ```bash
> $ apt update
> $ apt install net-tools
> ```

并且对于连接到默认的 `bridge` 之间的容器可以通过 IP 地址互相通信。例如我们启动一个 `shiyanlou002` 的容器，它可以与 `shiyanlou001` 通过 IP 地址进行通信。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516877067391.png/wm)

> 如果提示没有找到 `ping` 命令，可使用如下命令安装：
>
> ```bash
> $ apt update 
> $ apt install iputils-ping
> ```
>
> 其具体的实现原理可以参考链接 [Linux 上的基础网络设备](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516877067391.png/wm)，以及涉及到[网桥的工作原理](https://segmentfault.com/a/1190000009491002)

上述的操作我们通过 `ping` 命令演示了 `IP` 相关的内容。但是对于应用程序来讲，如果需要在外部进行访问，我们还会涉及到端口的使用，而 `Docker` 对于 `bridge` 网络使用端口的方式为设置端口映射，通过 `iptables` 实现。

下面我们通过 `iptables` 来为大家演示 docker 实现端口映射的方式，主要针对 `nat` 表和 `filter` 表：

1. 首先删除掉上面创建的两个容器。这里不再给出具体的命令

2. 这时，我们查看 `nat` 表的转发规则，使用如下命令：

```bash
$ sudo iptables -t nat -nvL
```

3. 由于此时并未创建 docker 容器，nat 表中没有什么特殊的规则。接下来，我们使用上一节构建的 `shiyanlou:1.0` 镜像创建一个容器 `shiyanlou001`，并将本机的端口 `10001` 映射到容器中的 `80` 端口上，在浏览器中可以通过 `localhost:10001` 访问容器 `shiyanlou001` 的 `apache` 服务，命令如下：

```
$ docker run -d -p 10001:80 --name shiyanlou001 shiyanlou:1.0
```

> `docker run` 命令的 `-p` 参数是通过端口映射的方式，将容器的端口发布到主机的端口上。其使用格式为 `-p ip:hostPort:containerPort`。并且还可以指定范围，例如 `-p 10001-10100:1-100`，代表将容器 `1-100` 的端口映射到主机上的 `10001-10100`端口上，两者一一对应。

4. 创建成功后，我们可以在浏览器中输入 `localhost:10001` 访问到容器`shiyanlou001` 的 `apache` 服务，并查看此时 `iptables` 中 `nat` 表和 `filter` 表的规则，其中分别新增了一条比较重要的内容，如下图所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517189337316.png/wm)

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517189420438.png/wm)

5. 接下来，再次使用镜像 `shiyanlou:1.0` 来启动一个容器 `shiyanlou002`，这次我们不指定端口映射，通过手动修改 `nat` 表的方式来模拟实现：

```bash
$ docker run -d --name shiyanlou002 shiyanlou:1.0
```

6. 获取容器 `shiyanlou002` 的 ip 地址，如果按步骤操作此 ip 为 `192.168.0.3`。此时我们想通过主机的 `10002` 端口访问容器 `shiyanlou002` 的 `80` 端口，就可以添加一条规则：

```
# 添加一条规则，大致解释为将从非 docker0 接口上，目的端口为 10002 的 tcp 报文，修改其目的地址为 192.168.0.3:80

$ sudo iptables -t nat -A DOCKER ! -i docker0 -p tcp --dport 10002 -j DNAT --to-destination 192.168.0.3:80
```

7. 添加成功后我们在本地发出的 `localhost:10002` 请求会被定位到 `192.168.0.3:80` 上，但是在将请求转发到 `docker0` 网桥上时，对于默认的 `filter` 表中的 `FOEWARD` 链的规则是 `DROP`，因此我们还需要在 `filter` 表中设置相应的规则：

```bash
$ sudo iptables -t filter -A FORWARD ! -i docker0 -o docker0 -p tcp -d 192.168.0.3 -j ACCEPT --dport 80

或者你也可以选择将其加到由 docker 定义的 DOCKER 链中，上面的命令和下面的命令选择其中的一个即可

$ sudo iptables -t filter -A DOCKER ! -i docker0 -o docker0 -p tcp -d 192.168.0.3 -j ACCEPT --dport 80
```

9. 此时我们就能够通过 `192.168.0.3:80` 访问容器 `shiyanlou002` 中的 `apache` 服务了。 即通过 `iptables` 的方式实现了容器 `shiyanlou002` 上 `80` 端口到主机 `10002` 端口的映射。

10. 最后，为了不影响后面实验的进行，这里我们删除掉手动添加的规则，并删除容器。

### 3.2 自定义网络

对于默认的 `bridge` 网络来说，使用端口可以通过端口映射的方式来实现，并且在上面的内容中我们也演示了容器之间通过 `IP` 地址互相进行通信。但是对于默认的 `bridge` 网络来说，每次重启容器，容器的 `IP` 地址都是会发生变化的，因为对于默认的 `bridge` 网络来说，并不能在启动容器的时候指定 ip 地址，在启动单个容器时并不容易看到这一区别。

#### 旧版的容器互联

容器间都是通过在 `/etc/hosts` 文件中添加相应的解析，通过容器名，别名，服务名等来识别需要通信的容器。

这里，我们启动两个容器，来演示旧的容器互联：

1. 首先启动一个名为 `shiyanlou001` 的容器，使用镜像 `busybox`：

```
$ docker run -it --rm --name shiyanlou001 busybox /bin/sh
```

2. 这时打开一个新的终端，启动一个名为 `shiyanlou002` 的容器，并使用 `--link` 参数与容器 `shiyanlou001` 互联。

```
$ docker run -it --rm --name shiyanlou002 --link shiyanlou001 busybox /bin/sh
```

> docker run 命令的 `--link` 参数的格式为 `--link <name or id>:alias`。格式中的 `name` 为容器名，`alias` 为别名。即可以通过 `alias` 访问到该容器。

如下图所示，左侧为 `shiyanlou001`，右侧为 `shiyanlou002`：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517193383367.png/wm)

3. 如果此时 `shiyanlou001` 容器退出，这时我们启动一个 `shiyanlou003`，再次启动一个 `shiyanlou001`：

```
$ docker run -itd --name shiyanlou003 --rm busybox /bin/sh

$ docker run -it --name shiyanlou001 --rm busybox /bin/sh
```

按照顺序分配的原则，此时 `shiyanlou003` 的 IP 地址为 `192.168.0.2`，容器 `shiyanlou001` 的 IP 地址为 `192.168.0.4`。并且此时容器 `shiyanlou002` 中 `/etc/hosts` 文件的解析依旧不变，所以不能获取到正确的解析：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517194194401.png/wm)

如上所示，旧的容器 `shiyanlou002` 通过 `--link` 连接到 `shiyanlou001`。而在 `shiyanlou001` 重启后，由于 IP 地址的变化，此时 `shiyanlou002` 并不能正确的访问到 `shiyanlou001`。

除了使用 `--link` 链接的方式来达到容器间互联的效果，在 `docker` 中，容器间的通信更应该使用的是自定义网络。

#### 自定义网络

docker 在安装时会默认创建一个桥接网络，除了使用默认网络之外，我们还可以创建自己的 `bridge` 或 `overlay` 网络。

如下所示，我们创建一个名为 `network1` 的桥接网络，简单命令如下：

```
$ docker network create network1

$ docker network ls
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517195281317.png/wm)

创建成功后，可以使用 `ifconfig` 或者 `ip addr show` 命令查看该桥接网络的网络接口信息，如下所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517195572337.png/wm)

而对于该网络的详细信息可以通过 `docker network inspect network1` 命令来查看，如下图所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517195691681.png/wm)

其相应的网络接口名称和子网都是由 docker 随机生成，当然，我们也可以手动指定：

```
# 首先删除掉刚刚创建的 network1 
$ docker network rm network1

# 再次创建 network1，指定子网
$ docker network create -d bridge --subnet=192.168.16.0/24 --gateway=192.168.16.1 network1
```

此时，我们可以运行一个容器 `shiyanlou001`，指定其网络为 `network1`，使用 `--network network1`：

```
$ docker run -it --name shiyanlou001 --network network1 --rm busybox /bin/sh
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517196736819.png/wm)

使用 `exit` 退出该容器使其自动删除，这时我们再次创建该容器，但是不指定其 `--network`：

```
$ docker run -it --name shiyanlou001 --rm busybox /bin/sh
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517196983288.png/wm)

此时，该容器连接到默认的 `bridge` 网络，这时，可以新打开一个终端，在其中运行如下命令，将 `shiyanlou001` 连接到 `network1` 网络中：

```
# 在新打开的终端中运行，将容器 shiyanlou001 连接到 network1 网络中
$ docker network connect network1 shiyanlou001

# 这时再次在容器 `shiyanlou001` 中使用 `ifconfig` 命令
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517197247336.png/wm)

如上图中所示，出现了一个 `eth1` 接口，此时，`eth0` 连接到默认的 `bridge` 网络，`eth1` 连接到 `network1` 网络。

对于自定义的网络来说，docker 嵌入的 `DNS` 服务支持连接到该网络的容器名的解析。这意味着连接到同一个网络的容器都可以通过容器名去 `ping` 另一个容器。

如下所示，启动两个容器，连接到 `network1`：

```
$ docker run -itd --name shiyanlou_1 --network network1 --rm busybox /bin/sh

$ docker run -it --name shiyanlou_2 --network network1 --rm busybox /bin/sh
```

启动之后，由于上述的两个容器都是连接到 `network1` 网络，所以可以通过容器名 `ping` 通：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517198037471.png/wm)

除此之外，在用户自定义的网络中，是可以通过 `--ip` 指定 IP 地址的，而在默认的 `bridge` 网络不能指定 IP 地址：

```
# 连接到 network1 网络，运行成功
$ docker run -it --network network1 --ip 192.168.16.100 --rm busybox /bin/sh

# 连接到默认的 bridge 网络，下面的命令运行失败
$ docker run -it --rm busybox --ip 192.168.0.100 --rm busybox /bin/sh
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517198404575.png/wm)

### 3.3 host 和 none

`host` 网络，容器可以直接访问主机上的网络。

例如，我们启动一个容器，指定网络为 `host`：

```
$ docker run -it --network host --rm busybox /bin/sh
```

如下所示，该容器可以直接访问主机上的网络：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517214736450.png/wm)

`none` 网络，容器中不提供其它网络接口。

```
$ docker run -it --nerwork none --rm busybox /bin/sh
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517214875513.png/wm)

## 5. 实验二：Docker 网络访问控制

容器默认是通过docker0的NAT模式访问外部网络，如果我们希望限制容器访问，有很多种办法：

### 5.1 限制容器访问外网

在上面的实验中，我们创建的shiyanlou容器是可以连接外网的，是通过shiyanlou0虚拟交换机进行了NAT转换：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1707timestamp1457325181829.png/wm)

限制容器访问外网，可以关闭IP转发，设置方法是启动Docker时`--ip-forward=false`。

`sudo vim /etc/default/docker` 修改配置文件：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1707timestamp1457325218047.png/wm)

在重启Docker服务之前，Ubuntu系统上需要先清除iptables规则和`/proc/`下的转发配置：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1707timestamp1457447433197.png/wm)


**注意：** 如果有容器正在运行时重启Docker服务，所有运行的容器都会被强制停止。Docker服务重启后也不会重启被停止的容器。

**注意：** 设置`/proc`下的值需要临时切换到root下才可以。单纯的sudo仍然会显示权限不够。

创建新的容器，查看配置效果：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1707timestamp1457447573602.png/wm)

容器启动时会有网络转发失效的提示，并且无法ping通baidu。

### 5.2 限制容器间的访问

限制容器间的访问，可以设置`--icc`参数，或设置`iptables`参数。默认容器间可以互相访问，通过设置`--icc=false`可以禁止。

修改配置文件：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1707timestamp1457326057291.png/wm)

重启Docker服务后创建新的容器site1，site2，查看site1的IP地址是`192.168.100.3`，site2中对该IP进行ping：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1707timestamp1457326159124.png/wm)

如果需要打开容器间访问，只需要设置`--icc=true`。

操作演示视频
`@
https://labfile.oss-cn-hangzhou.aliyuncs.com/courses/498/video/5-5.flv
@`

## 6. 实验三：Docker 容器端口映射

运行`docker run`的时候可以使用`-P`参数进行端口映射，这个参数不需要指定任何宿主机的端口，会自动从`49000~49900`端口中选择一个映射到容器中开放的端口。

而容器中开放的端口号如何获得呢？可以写在创建镜像的Dockerfile中：

```
FROM ubuntu:latest

EXPOSE 80
```

基于该Dockerfile创建一个新的镜像shiyanloutest：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1707timestamp1457339059381.png/wm)

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1707timestamp1457339066242.png/wm)

此时由该镜像创建的容器，如果使用了`-P`参数，则会自动分配一个宿主机的端口进行映射：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1707timestamp1457339206039.png/wm)

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1707timestamp1457339213623.png/wm)

如果想指定映射端口，可以使用`-p`参数，请注意一个宿主机的端口只能绑定到一个容器，如果该端口已经有进程在用则不可以绑定。并且`-p`参数可以绑定多个端口：

```
docker run -ti -p 80:80 -p 5000:5000 --name shiyanlou3 ubuntu
```

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1707timestamp1457339580918.png/wm)

`docker inspect shiyanlou3` 查看容器端口映射情况：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1707timestamp1457339587405.png/wm)

如果想映射到指定的本地地址，可以增加IP参数，比如映射到`127.0.0.1`地址，只需要将参数写成 `-p 127.0.0.1:80:80`。

查看指定容器的端口映射可以使用`docker port`命令：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1707timestamp1457339759107.png/wm)

`Docker` 的端口映射也是通过`iptables`规则来设置的，当完成映射后，使用`iptables`命令查看是否新增了规则：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1707timestamp1457339765821.png/wm)

操作演示视频
`@
https://labfile.oss-cn-hangzhou.aliyuncs.com/courses/498/video/5-6.flv
@`

## 7. 实验四：Docker 容器互联

### 7.1 实验场景

设定一个场景，我们的应用包含多个容器：

1. 容器1：Web前端服务 端口80
2. 容器2：App应用服务 端口80
3. 容器3：Redis数据库服务 端口6379

我们创建容器后，如何让这三个容器连接共同构成我们所需的Web应用服务呢？

此时我们并不想将每个容器的端口都通过映射的方式暴露出来，那么Docker又提供了怎样的方案呢？

这里需要介绍Docker强大的`--link`参数，完美支持容器间互联。

### 7.2 实验分析

需要创建三个容器，分别根据作用命名，在创建过程中使用`--link`参数让容器间建立安全的互联通道。

`docker run`命令的`--link`参数可以在不映射端口的前提下为两个容器间建立安全连接。`--link`参数可以连接一个或多个容器到将要创建的容器。

三个容器命名为：web，app，db，连接方式分别是web连接app，app连接db。其中web和app容器使用上面创建的`EXPOSE 80`端口的镜像`shiyanloutest:1.0`。

### 7.3 开始实验

按照`7.2`中的分析，依次创建三个容器：

```
docker run -d --name db redis
docker run -ti --name app --link db:db shiyanloutest:1.0 /bin/bash
docker run -ti --name web --link app:app shiyanloutest:1.0 /bin/bash
```

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1707timestamp1457345560042.png/wm)

进入到web容器中可以查看到`--link`参数为了连接容器做了哪些改变：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1707timestamp1457345574708.png/wm)

可以看到环境变量和`/etc/hosts`文件的变化，在web容器中可以`ping app`进行测试。

通过`--link`参数可以把几个容器绑定在一起，并且不需要向外部公开内部应用容器和数据库容器的端口号，只需要将对外提供服务的web服务端口80开放即可。

操作演示视频
`@
https://labfile.oss-cn-hangzhou.aliyuncs.com/courses/498/video/5-7.flv
@`

## 8. 总结

本节实验中我们学习了以下内容，任何不清楚的地方欢迎到[实验楼问答](https://www.shiyanlou.com/questions)与我们交流：

1. Docker 网络基本配置
2. Docker 网络访问控制
3. Docker 容器端口映射
4. Docker 容器互联

请务必保证自己能够动手完成整个实验，只看文字很简单，真正操作的时候会遇到各种各样的问题，解决问题的过程才是收获的过程。

