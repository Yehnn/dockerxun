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

1. 使用证书加固 Docker Daemon安 全
2. 设置特权级运行的容器：`--privileged=true`
3. 设置容器权限白名单：`--cap-add`
4. Docker Bench Security

Docker 的安全已经随着容器技术的推广受到越来越多的关注，本节实验中我们将体会几个最常用的Docker安全相关的配置。在开始实验之前，推荐阅读关于Docker 安全性的文章：

1. [Flux7 Docker系列教程（五）：Docker 安全](https://segmentfault.com/a/1190000002711383)
2. [Docker安全性探讨与实践：实践篇](http://www.infoq.com/cn/news/2014/09/docker-safe)

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
## 4. 使用证书加固Docker Daemon安全

Docker Daemon 启动的服务对外提供的是 HTTP 接口，为了增强 HTTP 连接的安全性，我们通过设置 TLS 来认证客户端是可信的，只有通过证书验证的客户端才可以连接 Docker Daemon。

通常情况下服务器和客户端证书都需要通过第三方 CA 签发，在本实验中为了操作方便，我们使用的是自签名的证书。

### 设置环境变量

有时候在创建 CA 密钥的时候可能会产生一个 `unable to write 'random state'` 的报错，我们需要先设置一个 `RANDFILE` 的环境变量：

```bash
$ export RANDFILE=.rnd
```

### 4.1 创建 CA 证书

首先创建一组 CA 私钥和公钥，先创建 CA 私钥，注意需要输入密码，这个密码是不会显示的，请务必记住，下面每次使用 CA 签名时都需要输入：

```bash
$ openssl genrsa -aes256 -out ca-key.pem 4096

Generating RSA private key, 4096 bit long modulus
................................................++
...................................++
e is 65537 (0x10001)
Enter pass phrase for ca-key.pem:        #此处输入你想设置的密码
Verifying - Enter pass phrase for ca-key.pem:  #再次输入
```

然后使用 `ca-key.pem` 创建用来签名的公钥 `ca.pem` ，输入必要的信息：

```bash
$ openssl req -new -x509 -days 365 -key ca-key.pem -sha256 -out ca.pem 

Enter pass phrase for ca-key.pem:  #输入之前设置的密码
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:CN
State or Province Name (full name) [Some-State]:Beijing
Locality Name (eg, city) []:Beijing
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Shiyanlou
Organizational Unit Name (eg, section) []:CourseTeam
Common Name (e.g. server FQDN or YOUR name) []:localhost
Email Address []:shiyanlou@shiyanlou.com
```

### 4.2 服务端证书配置

创建服务器的 key 和证书 `server.csr`，然后使用 CA 证书签发服务器证书：

```bash
$ openssl genrsa -out server-key.pem 4096

Generating RSA private key, 4096 bit long modulus
............................................................++
................++
e is 65537 (0x10001)
$ openssl req -subj "/CN=localhost" -sha256 -new -key server-key.pem -out server.csr

$ echo subjectAltName = IP:127.0.0.1 >> extfile.cnf
#设置仅用于服务器身份验证
$ echo extendedKeyUsage = serverAuth >> extfile.cnf

$ openssl x509 -req -days 365 -sha256 -in server.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out server-cert.pem -extfile extfile.cnf 

Signature ok
subject=/CN=localhost
Getting CA Private Key
Enter pass phrase for ca-key.pem:   #输入之前设置的密码
```

上面的步骤中 `extfile.cnf` 文件的作用是添加允许连接的 IP 地址，注意服务器证书创建时需要输入服务器的域名，在本实验中我们是用的是 `localhost`。

### 4.3 客户端证书配置

客户端的证书用来连接Docker Daemon服务，同服务器端证书的操作类似，先创建 key 和 csr 证书，然后使用 CA签发：

```bash
$ openssl genrsa -out client-key.pem 4096

Generating RSA private key, 4096 bit long modulus
.......................................++
...................++
e is 65537 (0x10001)

$ openssl req -subj '/CN=client' -new -key client-key.pem -out client.csr 

$ echo extendedKeyUsage = clientAuth > client-extfile.cnf

$ openssl x509 -req -days 365 -sha256 -in client.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out cert.pem -extfile client-extfile.cnf

Signature ok
subject=/CN=client
Getting CA Private Key
Enter pass phrase for ca-key.pem:   #输入之前设置的密码
```

现在查看 `/home/shiyanlou` 目录下所有创建的证书和key文件：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1712timestamp1458201290334.png/wm)

已经生成了 cert.pem 和 server-cert.pem ，我们就可以删除两个证书签名请求了：

```bash
$ rm -v client.csr server.csr
```

为防止意外损坏密钥，可以删除其写入权限：

```bash
$ chmod -v 0400 ca-key.pem server-key.pem client-key.pem
$ chmod -v 0444 ca.pem server-cert.pem cert.pem
```

### 4.4 配置服务端

在配置之前我们需要先停止 docker 服务：

```bash
$ sudo service docker stop
```

在服务器端运行如下命令让 Docker 守护进程只接受提供 CA 信任的证书的客户端连接：

```bash
dockerd --tlsverify --tlscacert=ca.pem --tlscert=server-cert.pem --tlskey=server-key.pem \
  -H=0.0.0.0:2376
```

**注意：** 保持运行状态，不要关掉此命令窗口或者使用 ctrl + c 中断。

### 4.5 客户端连接 Docker

直接使用不增加证书参数的方式连接并执行 `docker image ls` ，系统会返回不能连接。而使用客户端连接则可以正确执行。新开一个终端命令窗口执行如下命令：

```bash
docker --tlsverify --tlscacert=ca.pem --tlscert=cert.pem --tlskey=client-key.pem -H=127.0.0.1:2376 image ls
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521195451476.png-wm)

为了后续实验的方便，我们创建一个alias，来避免输入 docker  命令的 TLS 参数：

```bash
$ alias docker='docker --tlsverify --tlscacert=ca.pem --tlscert=cert.pem --tlskey=c
lient-key.pem -H=127.0.0.1:2376'

$ docker image ls
```


## 5. 设置特权级运行的容器：`--privileged=true`


有的时候我们需要容器具备更多的权限，比如操作内核模块，控制swap交换分区，挂载USB磁盘，修改MAC地址等。本实验中我们给予容器这些权限，仅仅通过一个简单的`--privileged=true` 的参数。

为了对比，我们先后创建两台容器shiyanlou和prishiyanlou，后者具备`--privileged=true`参数和特权。

首先创建一个不具备特权参数的容器shiyanlou，并查看容器的IP地址和MAC地址信息：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1712timestamp1458201529861.png/wm)

尝试修改 MAC 地址，会收到 `Operation not permitted` 报错信息：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1712timestamp1458201559515.png/wm)

尝试创建并挂载一个 iso 文件，同样，也会收到错误信息：


![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1712timestamp1458201611619.png/wm)

从 shiyanlou 容器中退出，我们创建一个具备 `--privileged=true`  参数的 prishiyanlou 容器，看是否有不同：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1712timestamp1458201662092.png/wm)


在这个容器中，我们首先尝试修改 MAC 地址，修改成功后使用 ifconfig 命令查看验证：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1712timestamp1458201689816.png/wm)


再次创建一个 iso 文件，并 mkfs.ext3 格式化分区后，mount 命令挂载到 `/mnt`，验证发现可以成功挂载：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1712timestamp1458201747483.png/wm)


`Ctrl-P Ctrl-Q` 退出容器后查看两个容器中的配置信息：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1712timestamp1458201783453.png/wm)

## 6. 设置容器权限白名单：`--cap-add`

`--privileged=true` 的权限非常大，接近于宿主机的权限，为了防止用户的滥用，需要增加限制，只提供给容器必须的权限。此时 Docker 提供了权限白名单的机制，使用 `--cap-add` 添加必要的权限。

为了能够修改 MAC 地址，我们给予新的容器 capshiyanlou 一个 `NET_ADMIN ` 的权限。创建该容器，注意命令中的参数：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1712timestamp1458201833312.png/wm)


尝试修改MAC地址，验证 `--cap-add ` 是否起到作用：

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid1712timestamp1458201867193.png/wm)

退出容器后，可以在 `docker container inspect` 命令中查看容器的必要配置：

```bash
$ docker container inspect -f {{.HostConfig.Privileged}} capshiyanlou
false

$ docker container inspect -f {{.HostConfig.CapAdd}} capshiyanlou
{[NET_ADMIN]}
```

## 7. Docker Bench Security

Docker Benchmark Security 是一个用于 docker 安全检查的应用程序，它会去检查下面的这些项目，并提供警告信息。它是一个开源工具，参见 [GitHub-Docker Bench Security](https://github.com/docker/docker-bench-security/) 。

- 主机配置
- Docker 守护进程配置
- Docker 守护进程配置文件
- 容器镜像和构建文件
- 容器运行时间
- Docker 安全操作


接下来我们就来安装 Docker Bench Security：

```bash
$ docker run -it --net host --pid host --userns host --cap-add audit_control \
    -e DOCKER_CONTENT_TRUST=$DOCKER_CONTENT_TRUST \
    -v /var/lib:/var/lib \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v /usr/lib/systemd:/usr/lib/systemd \
    -v /etc:/etc --label docker_bench_security \
    docker/docker-bench-security
```

![实验楼](https://dn-simplecloud.shiyanlou.com/87971521798043023-wm)

会显示出很多的提示信息：

![实验楼](https://dn-simplecloud.shiyanlou.com/87971521798200035-wm)

> `PASS` ：这些项目都是很稳固的，不需要关注，pass 越多越好。
>
> `WARN` ：需要修复的项目。
>
> `INFO` ：如果这些项目和你的设置和安全需要相关，建议检查和修复这些项目。
>
> `NOTE` ：一些建议。

## 8. 总结

本节实验中我们学习了以下内容：

1. 使用证书加固 Docker Daemon安 全
2. 设置特权级运行的容器：`--privileged=true`
3. 设置容器权限白名单：`--cap-add`
4. Docker Bench Security

请务必保证自己能够动手完成整个实验，只看文字很简单，真正操作的时候会遇到各种各样的问题，解决问题的过程才是收获的过程。









