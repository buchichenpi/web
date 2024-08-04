---
title: 亲自探索容器
date: 2024-08-04 18:51:00
tags:
- 容器
- k8s
categories: k8s实战笔记
---

理想情况下，直接在Linux计算机上安装Docker，这样就不必处理在**宿主操作系统内运行**的**虚拟机**中**运行容器**的额外复杂性。但是，如果使用的是macOS或Windows，并且不知道如何设置Linux虚拟机，那么Docker Desktop应用程序可以完成设置。运行容器的Docker命令行（CLI）工具将安装在宿主操作系统上，但Docker守护进程（daemon）将在虚拟机内运行，它所创建的所有容器也将如此。

Docker平台由许多组件组成，但只需要安装Docker Engine即可运行容器。如果使用的是macOS或Windows，需要先安装Docker Desktop。Docker文档地址：[https://docs.docker.com/get-docker/](https://docs.docker.com/get-docker/)。

## 运行一个hello-world容器
为了回显'hello-world'，需要先从Docker Hub中拉取一个叫`busybox`的镜像。`busybox`是一个可执行文件，它结合了许多标准的UNIX命令行工具，如echo、ls、gzip。当然，还有其他的镜像可以做到执行回显命令。

可以使用单个`docker run`命令完成下载指定的映像和在其中运行的命令。要运行简单的Hello world容器，请执行以下列表中显示的命令：
```bash
$ docker run busybox echo "Hello World"
Unable to find image 'busybox:latest' locally
latest: Pulling from library/busybox
213a27df5921: Pull complete
Digest: sha256:9ae97d36d26566ff84e8893c64a6dc4fe8ca6d1144bf5b87b2b85a32def253c7
Status: Downloaded newer image for busybox:latest
Hello World
```

### 当运行容器的时候发生了什么？
下图展示了当运行`docker run`命令的时候发生了什么
![17](../../assets/image/k8s-in-action/17.png)
将镜像下载到您的计算机后，Docker 守护进程根据该镜像创建了一个容器，并在其中执行了 echo 命令。该命令将文本打印到标准输出，然后进程终止，容器停止。

### 运行其他的镜像
运行其他现有的容器镜像与运行busybox镜像非常相似。实际上，它通常甚至更简单，因为通常不需要像在前面的示例中的echo命令那样指定要执行的命令。要执行的命令通常写在镜像本身中，但可以在运行时覆盖它。

例如，如果想运行Redis数据存储，可以在[http://hub.docker.com](http://hub.docker.com)或其他公共注册中心找到镜像名称。对于Redis来说，其中一个镜像被称为redis:alpine，所以可以像这样运行它：
```bash
$ docker run redis:alpine
```
### 理解容器tags
以redis容器为例，打开hub里redis的搜索结果[https://hub.docker.com/_/redis/tags](https://hub.docker.com/_/redis/tags)可以看到latest、buster, alpine，同样也有5.0.7-buster, 5.0.7-alpine等等。

Docker 允许你在同一名称下拥有同一镜像的多个版本或变体。每个变体都有一个唯一的标签（tag）。如果你引用镜像时没有明确指定标签，Docker 会假设你指的是特殊的 latest 标签。在上传一个新版本的镜像时，镜像的制作者通常会同时用实际的版本号以及 latest 来标记它。当你想要运行一个镜像的最新版本时，应该使用 latest 标签而不是指定具体的版本号。

即使是对于单个版本，镜像通常也有几种变体。对于 Redis5.0.7，有 5.0.7-buster 和 5.0.7-alpine。它们两个都包含相同版本的 Redis，但是它们所基于的基础镜像不同。5.0.7-buster 是基于 Debian 的“Buster”版本，而 5.0.7-alpine 是基于 Alpine Linux 基础镜像，这是一个非常精简的镜像，总共只有 5MB——它只包含了你在典型 Linux 发行版中看到的一小部分已安装二进制文件。

要运行镜像的特定版本和/或变体，请在镜像名称中指定标签。例如，要运行 5.0.7-alpine 标签的镜像，你将执行以下命令：
```bash
$ docker run redis:5.0.7-alpine
```

## 创建容器化的 Node.js Web 应用程序
下面创建一个简单的 Node.js Web 应用程序，并将其打包成一个容器镜像。这个应用程序将接受 HTTP 请求，并响应其正在运行的计算机的主机名。

这样，你将看到，尽管容器中的应用程序像其他任何进程一样在宿主机上运行，但它看到的主机名与宿主机的主机名不同。这在以后将应用程序部署到 Kubernetes 并进行扩展（水平扩展，即运行应用程序的多个实例）时非常有用。
```node.js
Listing 2.2 A simple Node.js web application: app.js
const http = require('http');
const os = require('os');

const listenPort = 8080;

console.log("Kubia server starting...");
console.log("Local hostname is " + os.hostname());
console.log("Listening on port " + listenPort);

var handler = function(request, response) {
  let clientIP = request.connection.remoteAddress;
  console.log("Received request for "+request.url+" from "+clientIP);
  response.writeHead(200);
  response.write("Hey there, this is "+os.hostname()+". ");
  response.write("Your IP is "+clientIP+". ");
  response.end("\n");
};

var server = http.createServer(handler);
server.listen(listenPort);
```

## 创建一个Dockerfile来构建容器映像
构建容器镜像的时候需要创建一个包含多个指令的dockfile。首先，构建一个app.js，文件内容如上所示。然后，创建一个Dockfile，文件内容包含如下指令：
```bash
FROM node:12
ADD app.js /app.js
ENTRYPOINT ["node", "app.js"]
```
``FROM`` 行定义了将要使用的容器镜像作为起点（即基础镜像）。在这个例子中，使用了 Node.js 的容器镜像，tag为12。在第二行中，将本地目录中的 app.js 文件以相同的名称（app.js）添加到镜像的根目录下。最后，在第三行中，指定了当执行镜像时 Docker 应该运行的命令，即``node app.js``。
为什么选择这个特定的镜像作为基础镜像。因为这个应用程序是一个 Node.js 应用程序，所以需要在镜像包含 node 二进制文件来运行该应用程序。也可以使用包含此二进制文件的任何镜像构造自己的镜像，甚至可以使用像 fedora 或 ubuntu 这样的 Linux 发行版基础镜像，并在构建镜像时将 Node.js 安装到容器中。但是，由于 node 镜像已经包含了运行 Node.js 应用程序所需的一切，因此从头开始构建镜像并没有意义。然而，在一些组织中，使用特定的基础镜像并在构建时向其添加软件可能是强制性的。

## 构建容器镜像
Dockerfile 和 app.js 文件是构建镜像所需的一切。现在，将使用下一个列表中的命令构建名为 kubia:latest 的镜像：
```
$ docker build -t kubia:latest .
```
-t 选项指定了所需的镜像名称和标签，末尾的点（.）指定了包含 Dockerfile 和构建上下文（构建过程所需的所有文件）的目录的路径。

当构建过程完成后，新创建的镜像将存储在计算机的本地镜像库中。可以通过列出本地镜像来查看它，如下面的列表所示：
```bash 
$ docker images
```
### 理解如何构建镜像（image）
下图展示了构建镜像的过程：
![18](../../assets/image/k8s-in-action/18.png)





<br>
<br>
