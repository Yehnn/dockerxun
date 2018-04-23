# 搭建自己的Docker Registry

## 1. 课程说明

课程为纯动手实验教程，为了能说清楚实验中的一些操作会加入理论内容。理论内容我们不会写太多，已经有太多好文章了，会精选最值得读的文章推荐给你，在动手实践的同时扎实理论基础。

实验环境中可以联网，不受实验楼网络限制。
## 2. 学习方法

实验楼的Docker课程包含14个实验，每个实验都提供详细的步骤和截图，适用于有一定Linux系统基础，想快速上手Docker的同学。

学习方法是多实践，多提问。启动实验后按照实验步骤逐步操作，同时理解每一步的详细内容。

如果实验开始部分有推荐阅读的材料，请务必先阅读后再继续实验，理论知识是实践必要的基础。

## 3. 本节内容简介

在本实验里我们将自己动手搭建一个类似 Docker Hub 的 Docker Registry。

本节中，我们需要依次完成下面几项任务：

1. Docker Registry 的部署和配置
2. 使用 Registry 管理仓库和镜像
3. Docker Registry的配置

本次实验的需求是搭建一个 Docker Registry，通过该 Registry  管理仓库和镜像。

Docker Registry 是一个用来管理 Docker 镜像的服务，本身也是一个 Docker 容器。大部分情况下都可以使用Docker Hub，私有的 Docker Registry 使用场景主要在当需要对容器镜像存储进行完全控制或需要把镜像管理进行集成的情况。

## 4. Docker Registry 部署

由于 Docker Registry 已经被制作成一个Docker镜像，所以安装部署非常简单，只需要按照我们通常的`docker run`就可以，如果本地没有 `registry` 的镜像，则会自动从 Docker Hub 上获取。

需要注意的是 Registry 默认的对外服务端口是 5000，如果我们宿主机上运行的 Registry 需要对外提供服务，可以通过映射端口的方式提供。

本节实验中我们使用 `registry:2` 镜像，这个镜像为 2.0 版本的 Registry。

部署 Docker Registry 的命令：

```
docker run -d -p 5000:5000 --restart=always --name registry registry:2
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521102681964.png-wm)

停止和删除 Registry 只需要用容器管理实验中学到的 `docker container stop` 和 `docker container rm` 命令。

## 5. 使用 Registry 管理仓库和镜像

以下示例从 Docker Hub 中拉取 redis 镜像，并将其重新标记为 my-redis ，然后将其推送到本地 Registry 。

### 5.1 推送镜像

运行后我们进行简单的测试，可以先将本地的一个 redis 镜像推送到 Registry 上：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521103308999.png-wm)

首先使用 `docker tag` 对 `redis` 镜像进行操作，会发现 `docker image ls` 列表中多了一个`localhost:5000/my-redis:latest` 的镜像。

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521103496106.png-wm)

上述命令将 `localhost:5000/my-redis` 镜像推送到 `localhost:5000` 的 Docker Registry 上。

注意 Docker 镜像的命名规则 `localhost:5000/my-redis` 中，`localhost:5000` 表示 Registry 的地址和端口。

### 5.2 获取镜像

如果我们需要从 `localhost:5000` 的Registry上下载镜像，只需要简单的 `docker pull`  命令就可以完成。

首先删除本地的 `localhost:5000/redis:latest` 和 `redis:latest` 镜像，以便从 Registry 下载镜像：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521103811855.png-wm)

然后从 Registry 中下载我们刚才上传的镜像。

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521104057288.png-wm)

`docker pull` 命令执行的过程中发现所有的数据层都已经存在本地。再次检查 `docker image ls` 就会看到新的`localhost:5000/redis:latest ` 镜像。

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521104217265.png-wm)

## 6. Docker Registry的配置

### 6.1 镜像存储

如果我们希望将 Registry 里的镜像都存储在一个数据卷中，这样做的好处是我们可以在宿主机上对该数据卷进行备份和监控。请回忆我们的数据卷参数 `-v`，此外，Registry中存储镜像的目录是 `/var/lib/registry`。

为了避免端口及命名冲突，将先前实验创建的 Registry 删除，然后在本机创建一个存储路径 `/home/shiyanlou/data` ：

```bash
$ docker container rm -f registry
$ mkdir /home/shiyanlou/data
```

创建一个 Registry，使用宿主机上的 `/home/shiyanlou/data` 目录作为存储镜像的数据卷：

```bash
$ docker run -d \
  -p 5000:5000 \
  --restart=always \
  --name registry \
  -v /home/shiyanlou/data:/var/lib/registry \
  registry:2
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521104741517.png-wm)

将本地的 `localhost:5000/my-redis`  镜像推送到 Registry，然后查看 `/home/shiyanlou/data` 目录的变化：

```bash
$ docker push localhost:5000/my-redis
$ ls /home/shiyanlou/data
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521105335725.png-wm)

会看到`/home/shiyanlou/data` 目录多了一个 `docker` 文件夹。在 `/home/shiyanlou/data/docker/registry/v2/repositories` 中可以看到刚刚上传的镜像。

除了使用数据卷做镜像存储之外，Registry 还支持将镜像存储到 亚马逊的 S3，OpenStack 的 Swift/Glance等存储后端。

详细的配置方式可以见文档 [Registry存储配置选项](https://docs.docker.com/registry/configuration/#storage) 。


### 6.2 认证机制

Registry 支持多种认证方式，这里仅仅介绍基本的认证方式，如果我们为 Registry 配置一个域名对外提供服务，需要首先配置 `TLS` ，可参考[运行可从外部访问的 Registry](https://docs.docker.com/registry/deploying/#customize-the-storage-back-end) 。在本实验中，我们暂时不适用域名的方式。

1. 首先我们需要创建一个保存用户名和密码的认证文件夹，并使用 `htpasswd` 创建一个认证信息：

```bash
$ mkdir /home/shiyanlou/auth
$ docker run --entrypoint htpasswd registry:2 -Bbn shiyanlouuser shiyanloupass > auth/htpasswd
```

认证信息中，用户名为 shiyanlouuser，密码是 shiyanloupass。

2. 删除 Registry

因为我们一会要创建一个 registry 容器，所以先删除我们之前创建的 registry 容器。

```bash
$ docker container rm -f registry
```

3. 启动 Registry，使用该认证信息，需要将 `/home/shiyanlou/auth` 挂载到 Registry 的 `/auth` 目录：

```bash
$ docker run -d \
  -p 5000:5000 \
  --restart=always \
  --name registry \
  -v `pwd`/auth:/auth \
  -e "REGISTRY_AUTH=htpasswd" \
  -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
  -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
  registry:2
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521108681547.png-wm)

接下来使用如下命令推送 `my-redis` 到本地 Registry，你会发现推送失败。为了可以通过认证向 Registry 推送本地的镜像，我们需要首先使用 `docker login` 配置认证信息，登录时会提示输入用户名和密码，我们输入之前配置的 `shiyanlouuser` 和 `shiyanloupass` 即可。

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521108975264.png-wm)

**注意：**Linux上的密码输入默认不显示，不是实验楼的系统卡住了。


其他认证方式的配置参数，可以参考文档 [Registry 认证配置](https://docs.docker.com/registry/configuration/#auth) 。

### 6.3 详细配置

**注意：本部分没有实验操作，仅仅是简单介绍 Registry 更详细的配置方法。**

Registry 的配置信息都存储在Registry中的`/etc/docker/registry/config.yml`文件。最简单的配置方法是直接将修改好的 YAML 配置文件挂载到 Registry 容器中。

如果只修改少数几个配置信息，可以通过设置环境变量来改变，比如修改 `storage/filesystem/rootdirectory`，只需要使用 `docker run` 的 `-e` 参数设置相应的`REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY`变量。

本节简单介绍下最常用的配置信息，详细信息可见 [Docker Registry配置](https://docs.docker.com/registry/configuration/)。

#### 日志级别

设置 `REGISTRY_LOG_LEVEL`，`REGISTRY_LOG_FORMATTER` 来设置日志输出级别和格式，级别可以设置为 `error`，`warn`，`info`，`debug`，默认为 `info `级别。输出格式可以为 `json`，`text`，`logstash`。

#### Hook

Hook 设置可以配置根据日志信息发送邮件。配置范例如下表所示，该例子来自 Docker 官方文档：

```
hooks:
  - type: mail
    levels:
      - panic
    options:
      smtp:
        addr: smtp.sendhost.com:25
        username: sendername
        password: password
        insecure: true
      from: name@sendhost.com
      to:
        - name@receivehost.com
```

#### 存储设置

前面的实验中，我们已经简单接触了本地数据卷存储的设置方式。此外，还可以设置多种存储后端。

每种配置的参数列表（来自Docker 官方文档）:

```
storage:
  filesystem:
    rootdirectory: /var/lib/registry
  azure:
    accountname: accountname
    accountkey: base64encodedaccountkey
    container: containername
  gcs:
    bucket: bucketname
    keyfile: /path/to/keyfile
    rootdirectory: /gcs/object/name/prefix
  s3:
    accesskey: awsaccesskey
    secretkey: awssecretkey
    region: us-west-1
    bucket: bucketname
    encrypt: true
    secure: true
    v4auth: true
    chunksize: 5242880
    rootdirectory: /s3/object/name/prefix
  rados:
    poolname: radospool
    username: radosuser
    chunksize: 4194304
  swift:
    username: username
    password: password
    authurl: https://storage.myprovider.com/auth/v1.0 or https://storage.myprovider.com/v2.0 or https://storage.myprovider.com/v3/auth
    tenant: tenantname
    tenantid: tenantid
    domain: domain name for Openstack Identity v3 API
    domainid: domain id for Openstack Identity v3 API
    insecureskipverify: true
    region: fr
    container: containername
    rootdirectory: /swift/object/name/prefix
  oss:
    accesskeyid: accesskeyid
    accesskeysecret: accesskeysecret
    region: OSS region name
    endpoint: optional endpoints
    internal: optional internal endpoint
    bucket: OSS bucket
    encrypt: optional data encryption setting
    secure: optional ssl setting
    chunksize: optional size valye
    rootdirectory: optional root directory
  inmemory:
  delete:
    enabled: false
  cache:
    blobdescriptor: inmemory
  maintenance:
    uploadpurging:
      enabled: true
      age: 168h
      interval: 24h
      dryrun: false
  redirect:
    disable: false
```


#### 认证设置

前面的实验已经学习过基本的htpasswd认证方式，除此之外，还支持Token方式的认证，Token方式可以使用独立的认证服务器。配置实例：

```
auth:
  token:
    realm: token-realm
    service: token-service
    issuer: registry-token-issuer
    rootcertbundle: /root/certs/bundle
```

#### HTTP设置

此处包含一些 Registry 提供的 HTTP 服务与协议的基本配置，包括监听地址与端口，TLS 等配置路径：

```
http:
  addr: localhost:5000
  net: tcp
  prefix: /my/nested/registry/
  host: https://myregistryaddress.org:5000
  secret: asecretforlocaldevelopment
  tls:
    certificate: /path/to/x509/public
    key: /path/to/x509/private
    clientcas:
      - /path/to/ca.pem
      - /path/to/another/ca.pem
  debug:
    addr: localhost:5001
  headers:
    X-Content-Type-Options: [nosniff]
```


## 7. 总结

本节实验中我们学习了以下内容：

1. Docker Registry 的部署和配置
2. 使用 Registry 管理仓库和镜像
3. Docker Registry 的配置

请务必保证自己能够动手完成整个实验，只看文字很简单，真正操作的时候会遇到各种各样的问题，解决问题的过程才是收获的过程。