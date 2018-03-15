# Docker 安全

## 1. 课程说明

课程为纯动手实验教程，为了能说清楚实验中的一些操作会加入理论内容。理论内容我们不会写太多，已经有太多好文章了，会精选最值得读的文章推荐给你，在动手实践的同时扎实理论基础。

实验环境中可以联网，不受实验楼网络限制。

## 2. 学习方法

实验楼的 Docker 课程包含 15 个实验，每个实验都提供详细的步骤和截图，适用于有一定 Linux 系统基础，想快速上手 Docker 的同学。

学习方法是多实践，多提问。启动实验后按照实验步骤逐步操作，同时理解每一步的详细内容。

如果实验开始部分有推荐阅读的材料，请务必先阅读后再继续实验，理论知识是实践必要的基础。

## 3. 本节内容简介

在本实验里我们将通过实验体会几个Docker中的安全设置。

本节中，我们需要依次完成下面几项任务：

1. 使用证书加固Docker Daemon安全
2. 设置特权级运行的容器：`--privileged=true`
3. 设置容器权限白名单：`--cap-add`
4. Docker AppArmor配置

Docker 的安全已经随着容器技术的推广受到越来越多的关注，本节实验中我们将体会几个最常用的Docker安全相关的配置。在开始实验之前，推荐阅读关于Docker 安全性的文章：

1. [Flux7 Docker系列教程（五）：Docker 安全](https://segmentfault.com/a/1190000002711383)
2. [Docker安全性探讨与实践：实践篇](http://www.infoq.com/cn/news/2014/09/docker-safe)

## 4. 实验一：使用证书加固Docker Daemon安全

Docker Daemon启动的服务对外提供的是HTTP接口，为了增强HTTP连接的安全性，我们通过设置TLS来认证客户端是可信的，只有通过证书验证的客户端才可以连接Docker Daemon。

通常情况下服务器和客户端证书都需要通过第三方CA签发，在本实验中为了操作方便，我们使用的是自签名的证书。

### 4.1 创建CA证书

首先创建一组CA私钥和公钥，先创建CA私钥，注意需要输入密码，这个密码是不会显示的，请务必记住，下面每次使用CA签名时都需要输入：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1712timestamp1458200907131.png/wm)

然后使用`ca-key.pem`创建用来签名的公钥`ca.pem`，输入必要的信息：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1712timestamp1458200958524.png/wm)

### 4.2 服务端证书配置

创建服务器的key和证书`server.csr`，然后使用CA证书签发服务器证书：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1712timestamp1458201045852.png/wm)


上面的步骤中`extfile.cnf`文件的作用是添加允许连接的IP地址，注意服务器证书创建时需要输入服务器的域名，在本实验中我们是用的是`localhost`。

### 4.3 客户端证书配置

客户端的证书用来连接Docker Daemon服务，同服务器端证书的操作类似，先创建key和csr证书，然后使用CA签发：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1712timestamp1458201181073.png/wm)

现在查看`/home/shiyanlou`目录下所有创建的证书和key文件：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1712timestamp1458201290334.png/wm)

### 4.4 启动Docker服务

修改`/etc/default/docker`配置文件，使 Docker Daemon以TLS证书保护的方式启动：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1712timestamp1458201332908.png/wm)

修改保存配置文件，然后重新启动 Docker 服务：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1712timestamp1458201355237.png/wm)


### 4.5 客户端连接Docker

直接使用不增加证书参数的方式连接并执行`docker ps`，系统会返回连接错误信息，使用客户端连接则可以正确执行：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1712timestamp1458201427850.png/wm)


为了后续实验的方便，我们创建一个alias，来避免输入docker 命令的TLS参数：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1712timestamp1458201477095.png/wm)

操作演示视频
`@
https://labfile.oss-cn-hangzhou.aliyuncs.com/courses/498/video/10-4.flv
@`


## 5. 实验二：设置特权级运行的容器：`--privileged=true`


有的时候我们需要容器具备更多的权限，比如操作内核模块，控制swap交换分区，挂载USB磁盘，修改MAC地址等。本实验中我们给予容器这些权限，仅仅通过一个简单的`--privileged=true` 的参数。

为了对比，我们先后创建两台容器shiyanlou和prishiyanlou，后者具备`--privileged=true`参数和特权。

首先创建一个不具备特权参数的容器shiyanlou，并查看容器的IP地址和MAC地址信息：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1712timestamp1458201529861.png/wm)

尝试修改MAC地址，会收到`Operation not permitted`报错信息：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1712timestamp1458201559515.png/wm)

尝试创建并挂载一个iso文件，同样，也会收到错误信息：


![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1712timestamp1458201611619.png/wm)

从shiyanlou容器中退出，我们创建一个具备`--privileged=true` 参数的prishiyanlou容器，看是否有不同：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1712timestamp1458201662092.png/wm)


在这个容器中，我们首先尝试修改MAC地址，修改成功后使用ifconfig命令查看验证：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1712timestamp1458201689816.png/wm)


再次创建一个iso文件，并mkfs.ext3 格式化分区后，mount命令挂载到`/mnt`，验证发现可以成功挂载：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1712timestamp1458201747483.png/wm)


`Ctrl-P Ctrl-Q` 退出容器后查看两个容器中的配置信息：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1712timestamp1458201783453.png/wm)

操作演示视频
`@
https://labfile.oss-cn-hangzhou.aliyuncs.com/courses/498/video/10-5.flv
@`

## 6. 实验三：设置容器权限白名单：`--cap-add`

`--privileged=true` 的权限非常大，接近于宿主机的权限，为了防止用户的滥用，需要增加限制，只提供给容器必须的权限。此时Docker 提供了权限白名单的机制，使用`--cap-add`添加必要的权限。

为了能够修改MAC地址，我们给予新的容器capshiyanlou一个`NET_ADMIN`的权限。创建该容器，注意命令中的参数：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1712timestamp1458201833312.png/wm)


尝试修改MAC地址，验证`--cap-add`是否起到作用：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1712timestamp1458201867193.png/wm)

退出容器后，可以在`docker inspect`命令中查看容器的必要配置：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1712timestamp1458201894538.png/wm)


操作演示视频
`@
https://labfile.oss-cn-hangzhou.aliyuncs.com/courses/498/video/10-6.flv
@`

## 7. 实验四：Docker AppArmor 配置

AppArmor 是一个安全框架，类似于SELinux，用来控制应用程序的各种权限，Docker也有一个 AppArmor 的配置文件，用来限制容器中对系统的访问权限，查看配置文件的内容如下：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1712timestamp1458202007300.png/wm)

可以使用`sudo apparmor_status` 命令查看当前系统中 AppArmor 配置的状态。

如果需要可以对 Docker 的AppArmor配置进行修改，但一定要小心，否则会造成很多奇怪的问题。详细的AppArmor文件配置可以参考文章[《Apparmor——Linux内核中的强制访问控制系统》](http://www.cnblogs.com/-Lei/archive/2013/02/24/2923947.html)及[官方wiki](http://wiki.apparmor.net/index.php/Main_Page)。

有的时候不知道如何修改，也可以简单的将AppArmor中的Docker配置为`complain`模式，这种模式下只会记录应用程序的越权操作，而不会阻止。需要首先安装`apparmor-utils`，然后再用`aa-complain`设置：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1712timestamp1458202234347.png/wm)

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1712timestamp1458202248789.png/wm)

这种设置可以快速解决一些问题，比如默认情况下Docker环境中无法使用gdb调试中的断点。但这种方式并不推荐，相当于把AppArmor的安全关闭。


## 8. 总结

本节实验中我们学习了以下内容：

1. 使用证书加固Docker Daemon安全
2. 设置特权级运行的容器：`--privileged=true`
3. 设置容器权限白名单：`--cap-add`
4. Docker AppArmor配置

请务必保证自己能够动手完成整个实验，只看文字很简单，真正操作的时候会遇到各种各样的问题，解决问题的过程才是收获的过程。