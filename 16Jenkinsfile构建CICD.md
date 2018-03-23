# Jenkins 构建 CI,CD

## 1. 课程说明

课程为纯动手实验教程，为了能说清楚实验中的一些操作会加入理论内容。理论内容我们不会写太多，已经有太多好文章了，会精选最值得读的文章推荐给你，在动手实践的同时扎实理论基础。

实验环境中可以联网，不受实验楼网络限制。

## 2. 学习方法

实验楼的Docker课程包含 15个实验，每个实验都提供详细的步骤和截图，适用于有一定Linux系统基础，想快速上手Docker的同学。

学习方法是多实践，多提问。启动实验后按照实验步骤逐步操作，同时理解每一步的详细内容。

如果实验开始部分有推荐阅读的材料，请务必先阅读后再继续实验，理论知识是实践必要的基础。

## 3. 本节内容简介

持续集成（英语：Continuous Integration，缩写 `CI`），是一种软件工程的流程，将软件中个人的工作副本持续的整合到共用主干的一种举措。

持续交付（英语：Continuous Delivery，缩写 `CD`）指的是将该软件产品的产出过程在一个短时间内完成，让软件的构建，测试，生产等变得更快更频繁。这种方式可以减少软件开发的成本与时间，减少风险。

> 对于 CI,CD 概念的理解，推荐大家阅读 [阮一峰的文章：持续集成是什么](http://www.ruanyifeng.com/blog/2015/09/continuous-integration.html)

本节中，我们需要依次完成下面几项任务：

1. Jenkins 安装部署
2. Jenkins 简单使用

## 4. Jenkins 的安装

Jenkins 是一个开源的软件项目，可用于自动执行构建，测试，交付或部署软件有关的各种任务。

其安装过程如下所示：

1. Jenkins 的安装在实验环境中，可以直接通过 `apt` 来完成，首先需要添加相应的源：

```
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
```

2. 除此之外，还需要安装 `java`，这里使用 `PPA` 的源去安装 `Oracle Java 8`：

> 若在实验环境（docker）中已经有安装 Java，可省略此步骤

```
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer
```

3. 更新索引库并直接安装

```
sudo apt-get update
sudo apt-get install jenkins
```

4. 启动服务

```
sudo service jenkins start
```

5. 上述命令运行成功之后，我们就可以打开浏览器，访问 `http://localhost:8080`，然后等待 Jenkins 页面出现。该页面会提示我们输入自动生成的密码，进行解锁：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517800624647.png/wm)

6. 我们需要在上图中提供的保存密码文件的路径中去查找到相应的密码

```
$ sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517800732741.png/wm)

7. 由于是随机生成的密码，所以实际显示的会与图片中的不一致。下面我们将获得的密码填入浏览器中的输入框，并点击 `Contiune` 按钮

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517453845217.png/wm)

8. 点击之后，会弹出一个窗口，提示我们安装相应的插件，这里直接选择建议安装的插件即可（但是安装太多的插件会影响性能，可以在安装成功之后，通过系统管理 > 管理插件页面来安装或删除插件）。

9. 插件安装完成之后，需要创建一个管理员用户，如下所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517454971463.png/wm)

10. 最后，我们就可以开始使用 Jenkins 了：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517455022891.png/wm)

Jenkins 安装操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week11/6-1.mp4
@`


## 5. 创建一个新任务

对于我们的软件来说，对其的每一个改变（例如，修改一行代码），我们都必须经过一个复杂的发布过程，即将改变后的软件重新发布。而 Jenkins Pipeline（一般直接称为 Pipeline）提供一套可扩展的工具，通过 `Pipeline Domain Specific Language(DSL) syntax` 将这个过程建模成代码，这些定义会被写入一个文件中，被称为 Jenkinsfile，用于实现自动化的持续交付。下面我们通过具体的示例来感受其效果：

### 5.1 启用 Blue-Ocean

1. 接下来的实验我们将通过 Jenkins 可视化程度更高的 Blue Ocean 来进行下面的实验，该插件需要安装，如下图所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517815267870.png/wm)

2. 选择 `Blue Ocean` 插件进行安装，安装完成后，直接返回首页，开始使用该插件：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517815287098.png/wm)

### 5.2 新建节点

对于软件的构建和测试等操作来说，我们都需要有运行该节点的机器，所以我们需要创建一个节点，选择 系统管理>>管理节点：

1. 首先选择系统管理中的管理节点

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517816859810.png/wm)

2. 选择创建节点：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517817253920.png/wm)

3. 选择启动方法，并在弹出框中添加证书

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517817659323.png/wm)

4. 输入相应的用户名和密码，即本地环境的相应的用户名和密码

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517817644938.png/wm)

5. 证书创建完成后，可以在下拉框中，选择相应的证书：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517817937490.png/wm)

6. 点击保存 `Save` 之后，该节点就创建成功了。

### 5.3 github 项目

对于实际需要构建和发布的软件项目，我们用 github 上的项目来演示，首先创建一个简单的 web 项目，并将其发布到自己的 github 上，该项目的目录结构如下所示：

```
项目名为 test_git，目录结构如下所示：

test_git
|----app.py
|----requirements.txt
|----README.md


其中 requirements.txt 文件中有一行内容:
-----------------
flask==0.10
-----------------


而 app.py 文件内容如下所示：
---------------------
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return 'Hello Shiyanlou002!'

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8001, debug=True)
---------------------
```

并且为了能够从 github 上获得我们的项目，还需要创建一个证书：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517819494670.png/wm)

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517819600998.png/wm)

### 5.4 开始

1. 打开浏览器，并输入 `localhost:8080`，显示结果如下：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517463933648.png/wm)

2. 如上图所示，选择图中标注的内容。`Open Blue Ocean` 是 Jenkins 发布的可视化程度更高的 UI 界面，进入后选择 `创建新的 Pipeline`：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517465773008.png/wm)

3. 选择代码仓库，并创建一个用于自动认证的 `key`，如下图所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517465918693.png/wm)

4. 创建 `token`：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517466199952.png/wm)

5. 创建成功后，进行复制，并粘贴到 jenkins 页面：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517466559684.png/wm)

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517466565068.png/wm)

6. 选择 `test_git` 后，点击 `创建 Pipelines`，显示如下图所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517818872018.png/wm)

7. 这时我们点击右侧的 `Add step`，根据提示分别选择 `Change current directory` 以及 `Git` 步骤（在 `Git` 的步骤中，需要输入刚刚创建的 github 账户证书的 ID）：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517819986853.png/wm)

8. 再次添加一个运行 `build` 和 `run` 的步骤，在 `build` 中，我们将使用 `sudo pip -r requirements.txt` 命令安装 `flask` 包。在 `run` 步骤中，运行 `app.py` 文件。

9. 点击右上角的 `Save` 并提交：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517824244458.png/wm)

10. 运行结果如下图所示:

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517825222068.png/wm)

11. 这时我们就能够通过浏览器访问相应的页面了，`http://localhost:8000`：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517825289349.png/wm)

对于上述所有的步骤而言，会直接在项目目录下生成一个 Jenkinsfile 的文件：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517907197961.png/wm)

如果对前面的第 7~8 步不熟悉的同学，也可以先直接编辑上图中的 Jenkinsfile 文件，其它步骤照常操作即可。

在每一次我们提交代码到项目时，都可以运行该构建流程，来测试是否能够正确的运行。甚至于可以设定在提交时，自动执行该过程。除了上面简单的示例之外，Jenkins 还有非常多的插件，能够实现更复杂的功能。可以根据自己的需求去构建相应的流程。


## 6. 总结

在本节内容中，我们通过 Jenkins 完成了一个自动化测试的流程。