# 数据卷管理

## 1. 课程说明

课程为纯动手实验教程，为了能说清楚实验中的一些操作会加入理论内容。理论内容我们不会写太多，已经有太多好文章了，会精选最值得读的文章推荐给你，在动手实践的同时扎实理论基础。

学习过程中遇到的所有问题，都可随时在[实验楼问答](https://www.shiyanlou.com/questions)中提出，与老师和同学一起交流。

实验环境中可以联网，不受实验楼网络限制。

## 2. 学习方法

实验楼的Docker课程包含15个实验，每个实验都提供详细的步骤和截图，适用于有一定Linux系统基础，想快速上手Docker的同学。

学习方法是多实践，多提问。启动实验后按照实验步骤逐步操作，同时理解每一步的详细内容，如果有任何疑问，随时在[实验楼问答](https://www.shiyanlou.com/questions/)中提问，实验楼团队和我都会及时回复大家的所有问题。

如果实验开始部分有推荐阅读的材料，请务必先阅读后再继续实验，理论知识是实践必要的基础。

## 3. 本节内容简介

Docker中的数据可以存储在类似于虚拟机磁盘的介质中，在Docker中称为数据卷（`Data Volume`）。数据卷可以用来存储Docker应用的数据，也可以用来在Docker容器间进行数据共享。

数据卷呈现给Docker容器的形式就是一个目录，支持多个容器间共享，修改也不会影响镜像。使用Docker的数据卷，类似在系统中使用 mount 挂载一个文件系统。

本节中，我们需要依次完成下面几项任务：

1. 创建数据卷
2. 管理数据卷权限
3. 挂载宿主机文件
4. 使用数据卷容器共享数据
5. 数据卷备份

## 4. 实验一：创建数据卷

容器管理实验中我们学习的命令 `docker run` 用来创建容器，可以在使用改命令时添加 `-v` 参数，就可以创建并挂载一个到多个数据卷到当前运行的容器中，`-v`的作用是将宿主机的一个目录作为容器的数据卷挂载到容器中，使宿主机和容器之间可以共享一个目录，如果本地路径不存在，Docker也会自动创建。

本节实验中，我们挂载2个数据卷到新创建的容器上：

```
# 创建两个目录
mkdir /tmp/data1 /tmp/data2 

# 分别将两个目录挂载到新创建的容器上
docker run -t -i --name shiyanlou -v /tmp/data1:/data1 -v /tmp/data2:/data2 ubuntu /bin/bash
```

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1705timestamp1457318723377.png/wm)

上述命令中 `-v` 参数可以使用多次，并挂在多个数据卷到容器中。后面的参数信息中冒号前面是宿主机的本地目录，冒号后面是容器中的挂载目录。

使用 `docker inspect shiyanlou` 查看shiyanlou容器中的数据卷信息：

```
docker inspect shiyanlou
```

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1705timestamp1457318777167.png/wm)

进入容器后我们可以查看和使用容器卷，尝试向这个容器卷中写入数据，然后在宿主机中查看是否存在：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1705timestamp1457318827112.png/wm)

可以看到容器中挂载的数据卷具备可写权限，那么如何对数据卷的权限进行管理呢？比如如何创建一个只读的数据卷呢？

操作演示视频
`@
https://labfile.oss-cn-hangzhou.aliyuncs.com/courses/498/video/3-4.flv
@`


## 5. 实验二：数据卷权限

挂载的数据卷默认为可读写权限，除非外部文件系统做了特殊限制，在 `docker run`的时候也可以执行为`只读`权限：

```
# 创建一个数据卷目录
mkdir /tmp/readonlydata

# 以只读的方式挂载到shiyanlouro容器上
docker run -t -i --name shiyanlouro -v /tmp/readonlydata:/readonlydata:ro ubuntu /bin/bash

```

上面的命令中参数很简单，`ro`表示 `readonly`，挂载后的数据卷就是只读权限了，这时候我们再次尝试向数据卷中写入：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1705timestamp1457319220814.png/wm)

除了可以挂载目录之外，文件也可以作为数据卷挂载到容器中。

操作演示视频
`@
https://labfile.oss-cn-hangzhou.aliyuncs.com/courses/498/video/3-5.flv
@`

## 6. 实验三：挂载宿主机上的文件

在本实验中，我们想让所有的容器都可以共享宿主机的`/etc/apt/sources.list`，从而只需要改变宿主机的apt源就能够影响到所有的容器。

```
docker run -t -i --name shiyanloufile -v /etc/apt/sources.list:/etc/apt/sources.list:ro ubuntu /bin/bash
```

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1705timestamp1457319473180.png/wm)

如果我们想共享一个数据卷给多个容器怎么办，比如设想一个场景，我们有两个处理上传数据的应用运行在不同的容器中，但需要同时读取同一个文件夹下的文件，此时，最好的方式是使用数据卷容器。

操作演示视频
`@
https://labfile.oss-cn-hangzhou.aliyuncs.com/courses/498/video/3-6.flv
@`

## 7. 实验四：数据卷容器

如果需要在多个容器间共享数据，并希望永久保存这些数据，最好的方式是使用数据卷容器，类似于一个提供网络文件共享服务的NFS服务器。

数据卷容器创建方法跟普通容器一样，只需要指定宿主机的一个文件夹作为数据卷即可，使用`docker create`命令创建但不启动数据卷容器：

```
docker create -v /shiyanloudata --name shiyanloudb ubuntu /bin/true
```

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1705timestamp1457319886170.png/wm)

其他使用该数据卷容器的容器创建时候需要使用`--volumes-from`参数，指定该容器名称或ID：

```
docker run --volumes-from shiyanloudb ...
```

创建site1和site2两个容器挂载数据卷容器shiyanloudb：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1705timestamp1457320149651.png/wm)

可以连接到这两个容器中对数据卷进行操作，并查看彼此之间是否已经有了共享文件：


![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1705timestamp1457320222435.png/wm)

操作演示视频
`@
https://labfile.oss-cn-hangzhou.aliyuncs.com/courses/498/video/3-7.flv
@`

## 8. 实验五：备份数据卷

继续使用实验四的环境，我们对数据卷容器中的数据进行备份，备份方法：

1. 创建一个新的容器
2. 挂载数据卷容器
3. 挂载宿主机本地目录作为数据卷
4. 将数据卷容器的内容备份到宿主机本地目录挂载的数据卷中
5. 完成备份操作后容器销毁

请按照上述步骤对数据卷容器shiyanloudb中的数据进行备份：

```
# 创建备份目录
mkdir /tmp/backup

# 创建备份容器
docker run --rm --volumes-from shiyanloudb -v /tmp/backup:/backup ubuntu tar cvf /backup/shiyanloudb.tar /shiyanloudata
```

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1705timestamp1457320587308.png/wm)

操作演示视频
`@
https://labfile.oss-cn-hangzhou.aliyuncs.com/courses/498/video/3-8.flv
@`

## 9. 总结

本节实验中我们学习了以下内容，任何不清楚的地方欢迎到[实验楼问答](https://www.shiyanlou.com/questions)与我们交流：

1. 创建数据卷
2. 管理数据卷权限
3. 挂载宿主机文件
4. 使用数据卷容器共享数据
5. 数据卷备份

请务必保证自己能够动手完成整个实验，只看文字很简单，真正操作的时候会遇到各种各样的问题，解决问题的过程才是收获的过程。
