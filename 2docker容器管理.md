# 容器管理

## 1. 课程说明

课程为纯动手实验教程，为了能说清楚实验中的一些操作会加入理论内容。理论内容我们不会写太多，已经有太多好文章了，会精选最值得读的文章推荐给你，在动手实践的同时扎实理论基础。

学习过程中遇到的所有问题，都可随时在[实验楼问答](https://www.shiyanlou.com/questions)中提出，与老师和同学一起交流。

实验环境中可以联网，不受实验楼网络限制。

## 2. 学习方法

实验楼的Docker课程包含15个实验，每个实验都提供详细的步骤和截图，适用于有一定Linux系统基础，想快速上手Docker的同学。

学习方法是多实践，多提问。启动实验后按照实验步骤逐步操作，同时理解每一步的详细内容，如果有任何疑问，随时在[实验楼问答](https://www.shiyanlou.com/questions/)中提问，实验楼团队和我都会及时回复大家的所有问题。

如果实验开始部分有推荐阅读的材料，请务必先阅读后再继续实验，理论知识是实践必要的基础。

## 3. 本节内容简介

容器是Docker的一个基本概念，每个容器中都运行一个应用并为该应用提供完整的运行环境。本实验将详细学习Docker容器的创建，运行管理操作。需要依次完成下面几项任务：

1. 创建第一个容器
2. 查看容器信息
3. 容器创建
4. 管理容器运行状态
5. 容器导出及导入

## 4. 实验一：创建第一个容器

还记得上一节实验中我们如何创建一个持续运行的容器吗？在这里我们回顾下创建的步骤：

如果我们需要一个保持运行的容器呢，最简单的方法就是给这个容器一个可以保持的应用，比如`bash`，运行 ubuntu  容器并进入容器的 bash：

```
$ docker run -t -i ubuntu /bin/bash
```

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1702timestamp1456718056669.png/wm)

上面命令的说明：

1. `-t`：分配一个 `pseudo-TTY`
2. `-i`：`--interactive`参数缩写，表示交互模式，如果没有 `attach` 保持 STDIN 打开状态
3. `ubuntu`：运行的镜像名称，默认为`latest` 标签
4. `/bin/bash`：容器中运行的应用

通过这个简单的命令，我们现在进入了新创建容器的bash中，在bash里执行的任何命令都不会影响到我们的宿主机，可以随意操作。你可以看到主机名和环境变量 `HOSTNAME` 都已经显示为容器的ID了。

在这个bash下，我们可以进行各种Ubuntu系统上的操作，当然因为Docker本身的限制，有些涉及到磁盘、网络、设备等Linux特权命令是无法执行的，可以试试`reboot`命令，会提示你`shutdown: Unable to shutdown system`。

如何退出这个bash呢？有两种方法，两种方法的效果完全不同：

1. 直接 `exit`，这时候 bash 程序终止，容器进入到停止状态
2. 使用组合键退出，仍然保持容器运行，我们可以随时回来到这个bash中来，组合键是 `Ctrl-p Ctrl-q`，你没有看错，是两组组合键，先同时按下`Ctrl`和p，再按`Ctrl`和q。就可以退出到我们的宿主机了。

上述第二种方法比较常用，此时使用 `docker ps` 查看，能看到容器仍然在运行中：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1702timestamp1456718063612.png/wm)

**注意：**有些浏览器把`Ctrl-Q`作为退出标签的快捷键，可能在执行上述命令时实验楼的页面会关掉，不过没有关系，重新打开实验楼的网站登陆后点击头像旁边的`继续实验`就能够再次回到实验桌面。

如果想再次回到刚才的bash中，只需要使用 `docker attach`命令就可以再次连接到运行的bash里：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1702timestamp1456718083623.png/wm)

**注意：** 命令后面的参数是容器的ID，并不需要输入完整的数字，只要能唯一定位这个容器即可，通常输入4位就足够了。

我们创建了第一个容器后，将会先实践一些容器信息查看的命令。通过这些命令我们可以在宿主机上了解到容器的运行情况。

**注意：**下面的命令都是针对该容器执行的，参数中的容器ID请替换成你实际实验中创建的容器ID。

## 5. 实验二：查看容器信息

### 5.1 查看容器列表 - `docker ps`

`docker ps` 命令最常用，可以列出所有容器的信息，默认情况下只显示运行状态的容器。

必要的参数在上一节实验中已经介绍过了，这里可以进行回顾。

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid3858labid1702timestamp1456718092203.png/wm)

几个最常用的参数：

+ `-a`：查看所有容器，含停止运行的
+ `-l`：查看刚启动的容器
+ `-q`：只显示容器ID

我们查看所有容器的ID列表：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1704timestamp1457313185523.png/wm)

### 5.2 查看容器内进程信息 - `docker top`

`docker top` 命令查看容器中运行的进程信息，显示容器中进程的PID，UID，PPID，时间，tty等信息。

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1704timestamp1457313238992.png/wm)

上图中的输出结果由于终端大小限制，造成列表压缩到两行。

### 5.3 查看容器输出信息 - `docker logs`

获取容器的输出信息可以使用 `docker logs`命令，我们使用 `docker attach` 回到刚才创建的`/bin/bash`容器中，写一个循环输出信息的脚本，然后再使用`Ctrl-P Ctrl-Q`组合键退出。

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1704timestamp1457313599084.png/wm)

在宿主机的终端中，我们可以用`docker logs`命令查看输出信息。

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1704timestamp1457313646454.png/wm)

`docker logs` 只会显示截止到当前的所有输出，如果想动态查看实时输出，也可以加`-f`参数，类似`tail`命令：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1704timestamp1457313725110.png/wm)

### 5.4 查看容器详细信息 - `docker inspect`

`docker inspect` 查看容器的细节信息，包括创建时间，操作命令，端口映射信息，IP地址等等。

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1704timestamp1457313874553.png/wm)

这个命令不只可以查看容器的详细信息，也支持镜像的详细信息。默认输出JSON格式的信息，可以通过`-f`指定输出的项目。

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1704timestamp1457313990650.png/wm)

上述命令中我们查看了网络配置信息中的IP地址和Gateway地址。


### 5.5 查看容器的运行信息 - `docker stats`

`docker stats ` 可以查看到运行状态容器的CPU，内存及网络使用率。在实际工作中，我们通常会把这个命令的输出连接到类似Logstash一类的服务用来分析。

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1704timestamp1457314116133.png/wm)

这个命令的输出是实时刷新的（类似Linux上的`top`命令），如果需要退出可以使用`Ctrl-C`组合键。

### 5.6 查看容器中的修改 - `docker diff`

`docker diff` 查看容器中对镜像做了哪些变化。

实验过程如下：

1. 先执行`docker diff` 查看现有的容器中的变化，发现没有任何文件变化
2. 连接到容器内部，`Ctrl-C`中断先前实验的死循环
3. 再创建几个文件
4. 退出到宿主机
5. 再次使用`docker diff`命令查看是否有新的修改


![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1704timestamp1457314388834.png/wm)

输出的信息中`A` 表示添加，后面的三个新建文件的路径。可以尝试下修改或删除文件会有怎样的`diff`输出。

### 5.7 连接到容器中 - `docker attach` 

`docker attach` 可以进入到容器操作。当我们容器后台运行时，有需要的话也可以再次连接进入到容器中。

这个命令在上述的实验中已经多次用到了，不再提供单独的操作。

操作演示视频
`@
https://labfile.oss-cn-hangzhou.aliyuncs.com/courses/498/video/2-5.flv
@`

## 6. 实验三：创建容器

创建一个容器的命令是 `docker run`。还记得先前实验中学习的`Hello, Shiyanlou`及`/bin/bash`容器如何创建的吗？在上一节中我们学习了最基本的容器创建方式，本节内容我们将通过实例来学习更详细的`docker run`参数。

`docker run`命令的执行步骤：

1. 查找镜像或下载镜像
2. 创建容器
3. 分配文件系统及虚拟网络（网桥，接口，IP地址），其中容器中的DNS默认挂载宿主机的`/etc/resolve.conf`和 `/etc/hosts`。
4. 执行应用，默认执行镜像中指定的CMD参数，也可以在`docker run`后面跟应用来覆盖CMD命令。

如果容器中的应用执行完成，则容器进入到终止状态。

`docker run` 的参数非常多，本实验中我们设定要创建的容器配置：

1. 设置容器名称 `shiyanlou`（使用`--name`，如果不加该参数，Docker会随机产生一个名字）。
2. 设置容器的主机名 `shiyanlou`(使用`--hostname`参数）
3. 设定网络信息，这里只使用一个简单的参数设置MAC地址（`--mac-address`参数）
4. 设置资源限制，设置容器中最大的进程数，包括soft和hard两个限制值（使用`-ulimit nproc=...`等参数）

创建容器过程中也可以挂载数据卷，数据卷在下一节实验中会详细介绍。这里不过多涉及。

根据上述的需求我们通过查询 `docker run --help` ，使用相关参数，创建符合要求的容器：

```
docker run --name shiyanlou --hostname shiyanlou --mac-address 00:01:02:03:04:05 --ulimit nproc=1024:2048 -t -i ubuntu /bin/bash 
```

进入容器中我们可以对一些参数进行验证：


![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1704timestamp1457315426985.png/wm)

容器中的 `ulimit` 不会有任何输出，查看实际的`ulimit`信息可以在宿主机上使用`docker inspect`查看：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1704timestamp1457315508767.png/wm)

操作演示视频
`@
https://labfile.oss-cn-hangzhou.aliyuncs.com/courses/498/video/2-6.flv
@`

## 7. 实验四：容器运行状态

### 7.1 守护状态

首先需要了解的概念是容器的守护状态，类似于守护进程，需要为`run`命令增加参数`-d`，此时容器在后台以守护状态（Daemonized）形式运行。

创建一个守护状态的容器：

```
docker run -d ubuntu /bin/bash -c "while true; do echo 'hello shiyanlou'; sleep 1; done"
```

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1704timestamp1457315792638.png/wm)


会启动一个守护状态的容器在后台运行，使用 `docker attach` 登录上去可以看到循环输出 `hello shiyanlou` 的字符串。

### 7.2 容器运行管理

本节实验中，我们需要练习启动，停止，重启容器的若干命令。这些命令用来管理从容器创建后到删除的整个生命周期。

#### 停止容器 `docker stop`

停止运行状态的容器，进入到终止状态。停止状态的容器可以通过 `docker ps -a` 查看到。

首先使用 `docker stop shiyanlou` 命令停止名称为shiyanlou的容器：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1704timestamp1457315863441.png/wm)

使用`docker ps -a`查看容器状态：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1704timestamp1457315928126.png/wm)

#### 启动容器 `docker start`

启动停止状态的容器。再次启动名称为shiyanlou的容器：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1704timestamp1457315965339.png/wm)

#### 重启容器 `docker restart`

可以将运行状态的容器终止，然后重新启动。

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1704timestamp1457315994098.png/wm)

#### 杀死容器 `docker kill`

跟进程相同，有的时候正常的终止操作不起作用时，我们需要使用 `kill` 命令杀死进程，在`docker kill`可以处理异常的运行状态的容器，强制退出：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1704timestamp1457316032870.png/wm)


#### 暂停和恢复容器 `docker pause/unpause`

类似Windows操作系统的睡眠，我们可以先临时将容器的运行挂起，不再使用CPU资源，当需要的时候再恢复成正常的运行状态。

先启动shiyanlou容器，再执行`pause`操作：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1704timestamp1457316092752.png/wm)

恢复shiyanlou容器：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1704timestamp1457316126208.png/wm)

#### 删除容器 `docker rm`

当一个容器不再需要时，我们可以删除这个容器。对于停止的容器直接执行`docker rm 容器ID`，对于运行状态的容器也可以执行`docker rm -f 容器ID强制删除`。

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1704timestamp1457316161305.png/wm)

操作演示视频
`@
https://labfile.oss-cn-hangzhou.aliyuncs.com/courses/498/video/2-7.flv
@`

## 8. 实验五：导出和导入容器

导出和导入容器操作可以将容器导出到压缩包，并可将压缩包导入到Docker系统中成为镜像，为容器的迁移和镜像的制作提供支持。

### 8.1 容器导出 `docker export`

导出容器快照到本地的tar包。导出后的文件可以拷贝到其他 Docker 服务器上执行导入命令形成新的镜像，我们在实验楼的环境中进行测试。

实验过程：

1. 查看当前环境中的容器，选择需要导出的容器
2. 查看该容器修改的内容
3. 导出容器到tar包，保存到`/home/shiyanlou`目录
4. 查看导出的tar包

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1704timestamp1457316343636.png/wm)

需要注意的是，当容器导出后，容器仍然在Docker环境中运行，只是拷贝了一份内容到tar包。

### 8.2 容器导入 `docker import`

我们执行导入命令，将该文件加载到docker系统中，文件加载后会成为镜像，命令执行时需要制定导入后生成的镜像的名字：

```
cat shiyanlou.tar | docker import - shiyanlou:1.0
```

执行导入后，使用`docker images` 查看是否有新的镜像产生：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1704timestamp1457316515145.png/wm)

docker import 命令比较灵活，也可以直接从URL链接进行导入。所以可以记住这是一种创建镜像的方式，将容器导出后拷贝到目标服务器然后导入成镜像。

使用新镜像创建容器，查看是否与导出的容器内容一致：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1704timestamp1457316576910.png/wm)

操作演示视频
`@
https://labfile.oss-cn-hangzhou.aliyuncs.com/courses/498/video/2-8.flv
@`

## 9. 总结

本节实验中我们学习了以下内容，任何不清楚的地方欢迎到[实验楼问答](https://www.shiyanlou.com/questions)与我们交流：

1. 创建第一个容器
2. 查看容器信息
3. 容器创建
4. 管理容器运行状态
5. 容器导出及导入

请务必保证自己能够动手完成整个实验，只看文字很简单，真正操作的时候会遇到各种各样的问题，解决问题的过程才是收获的过程。
