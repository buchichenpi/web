---
title: 容器运行hello-world
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

## 当运行容器的时候发生了什么？
下图展示了当运行`docker run`命令的时候发生了什么
![17](../../assets/image/k8s-in-action/17.png)
将镜像下载到您的计算机后，Docker 守护进程根据该镜像创建了一个容器，并在其中执行了 echo 命令。该命令将文本打印到标准输出，然后进程终止，容器停止。

## 运行其他的镜像
运行其他现有的容器镜像与运行busybox镜像非常相似。实际上，它通常甚至更简单，因为通常不需要像在前面的示例中的echo命令那样指定要执行的命令。要执行的命令通常写在镜像本身中，但可以在运行时覆盖它。

例如，如果想运行Redis数据存储，可以在[http://hub.docker.com](http://hub.docker.com)或其他公共注册中心找到镜像名称。对于Redis来说，其中一个镜像被称为redis:alpine，所以可以像这样运行它：
```bash
$ docker run redis:alpine
```




<br>
<br>
