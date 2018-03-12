# 网络管理

## 1. 课程说明

课程为纯动手实验教程，为了能说清楚实验中的一些操作会加入理论内容。理论内容我们不会写太多，已经有太多好文章了，会精选最值得读的文章推荐给你，在动手实践的同时扎实理论基础。

学习过程中遇到的所有问题，都可随时在[实验楼问答](https://www.shiyanlou.com/questions)中提出，与老师和同学一起交流。

实验环境中可以联网，不受实验楼网络限制。

## 2. 学习方法

实验楼的Docker课程包含15个实验，每个实验都提供详细的步骤和截图，适用于有一定Linux系统基础，想快速上手Docker的同学。

学习方法是多实践，多提问。启动实验后按照实验步骤逐步操作，同时理解每一步的详细内容，如果有任何疑问，随时在[实验楼问答](https://www.shiyanlou.com/questions/)中提问，实验楼团队和我都会及时回复大家的所有问题。

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

## 4. 实验一：Docker网络基本配置

### 4.1 默认情况

Docker服务启动时会自动创建一个 docker0 的虚拟网桥，后续新创建的容器都会有个虚拟接口连接到这个网桥：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1707timestamp1457323162645.png/wm)

Docker网桥会设置为NAT模式，自动分配一个网段，实验楼提供的容器环境中`docker0`的地址是`192.168.0.1`，每个容器都会自动分配的到一个IP地址：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1707timestamp1457323332152.png/wm)


可以通过`docker inspect shiyanlou`查看网络配置信息：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1707timestamp1457323427787.png/wm)

我们可以为 Docker 服务指定不同的网桥以及网段，这些配置都可以写在 `/etc/default/docker` 文件中，作为服务启动的参数。

### 4.2 配置文件 `/etc/default/docker` 

该文件为Ubuntu 14.04 操作系统中 Docker 服务启动时使用的配置文件，不同的操作系统位置会有不同。这个文件本身是个 `Shell` 脚本。

首先查看文件内容：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1707timestamp1457323497964.png/wm)

包含的项目都在上图中显示出来，我们如果要对网络进行配置，需要将必要的配置参数填入`DOCKER_OPTS`启动参数行。

### 4.3 修改配置文件

实验步骤如下：

1. 删除docker0虚拟网络
2. 创建一个新的虚拟网络shiyanlou0
3. 配置shiyanlou0的网段为192.168.100.1/24
4. 配置Docker使用新的虚拟网络shiyanlou0


首先，删除docker0，删除前需要删除所有容器并停止Docker服务：

```
# 安装brctl管理工具
sudo apt-get install bridge-utils

# 查看当前虚拟网络列表
sudo brctl show

# 停止docker服务
sudo service docker stop

# 停止docker0
sudo ip link set dev docker0 down

# 删除docker0
sudo brctl delbr docker0
```

操作过程截图如下，供参考：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1707timestamp1457323849043.png/wm)

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1707timestamp1457323855650.png/wm)

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1707timestamp1457323933933.png/wm)

然后，我们开始创建一个新的虚拟网络：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1707timestamp1457324105632.png/wm)

配置shiyanlou0的网络：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1707timestamp1457324220194.png/wm)

最后，我们根据需求去修改`/etc/default/docker`文件，Docker 网络常见的配置参数：

1. -b --bridge ：指定连接的网桥
2. --bip=CIDR：指定IP地址网段
3. --icc=true|false：是否允许容器间网络互通
4. --ip-forward=true|false：是否允许IP转发，可以对容器的外网访问进行限制

在`/etc/default/docker`中需要添加下面几个参数：

1. `-b=shiyanlou0` 指定使用shiyanlou0
2. `--icc=false` 关闭容器间互联
3. `--ip-forward=true` 打开IP转发

修改后的文件内容：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1707timestamp1457324770303.png/wm)

完成上述配置后，请不要忘记重启Docker服务配置文件才会生效。

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1707timestamp1457324814917.png/wm)

实验中我们修改了上面配置，可以通过创建新的容器来看下修改后的效果：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1707timestamp1457324869591.png/wm)

新容器创建时，IP地址被自动分配为`192.168.100.2`，注意所有新的配置都应该重启Docker服务。

当然，我们在容器启动时，`docker run` 命令也有网络相关的参数，可以在容器启动时进行针对单个容器的网络配置，常见的配置选项：

1. -h --hostname：配置容器主机名
2. --link：添加另外一个容器的链接，见后续实验内容
3. --net=bridge|none|container|host：设置容器的网络模式

我们创建一个新的容器，设置主机名为test1，网络模式为none：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1707timestamp1457324972954.png/wm)

网络模式none设置后，在容器中没有出现eth0网卡。感兴趣的同学可以尝试下其他的网络模式。

操作演示视频
`@
https://labfile.oss-cn-hangzhou.aliyuncs.com/courses/498/video/5-4.flv
@`

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

