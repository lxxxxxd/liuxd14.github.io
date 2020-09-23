# Docker容器 （一）安装

docker-ce是docker公司开源的容器解决方案，让我们从安装docker-ce开始。

docker公司面向windows，mac，linux平台提供了详细的安装指导，见docker官方文档：https://docs.docker.com/get-docker/ 。

笔者选择了linux的一个发行版本CentOS，其他的系统平台或者发行版本，读者朋友可以参考上文的docker官方文档。

系统版本和配置要求：docker对于CentOS系统有一些要求，为了避免安装失败，请确保您的系统满足这些要求。

（1）由于docker-ce底层依赖一些内核特性，CentOS请选择CentOS 7或者以上的版本；

（2）打开centos-extras软件仓库配置；

（3）推荐使用overlay2存储驱动，文件分层一种实现方式，以后会带您参观的，请稍安勿躁；

安装步骤：为了避免用户安装时候被坑，docker社区已经开发了完整的方案，并形成了具有详细步骤的运维资料。

（1）卸载老版本的docker，现在软件包名称为docker-ce，老版本的docker和docker-ce在架构上有了进行了调整，输入如下命令：
```
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```
（2）条条大路通罗马，有三种方式可以安装docker-ce，这里我们选择最常用的从软件源上进行安装的方式。

安装docker社区为docker-ce提供的软件源。安装软件源依赖yum-config-manager二进制工具，其包含在yum-utils软件包中。
```
$ sudo yum install -y yum-utils

$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```
安装docker-ce所需要的软件包，安装的软件包有点多，这涉及到docker架构的演化和调整，后面的会给您进行解说的。
```
$ sudo yum install docker-ce docker-ce-cli containerd.io
```
到目前为止，docker-ce已经安装到了您的机器上。

现在让我们尽情的玩耍吧，先从一个hello-world开始，这是老传统了，哈哈
```
$ docker run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```
读者朋友们，hello-world输出不仅是docker世界向您发来的第一声问候，而且具有如下作用：

（1）检验您的docker-ce安装的是否有问题；

（2）向您简要介绍了这个命令后面的工作原理；
```
docker-ce是一个客户端/服务端(C/S)架构的软件，客户端向服务端发送命令，服务端负责执行命令并响应客户端。
hello-world的输出中有个单词image，这个代表镜像，是一堆分层打包的文件，在您启动容器的时候就为您准备好的，docker hub就是这种文件的一个仓库。
```
（3）把您带到一个镜像世界(docker hub)https://hub.docker.com/ ;

（4）把最全的开发文档免费地送到您面前 https://docs.docker.com/get-started/ ;

问题1：其实上面说了这么多，我们就在做一件事情，即安装docker-ce。但是如果换您来设计这个安装方案，您应该怎么设计？应该从哪些方面来考虑？

（1）操作系统平台，linux，win，mac

（2）体系结构，arm，amd64,ppc64,mips64le

（3）操作系统软件版本要求和配置要求

（5）软件包安装依赖，推荐的软件配置选项，安装后进行验证

问题2：您觉得docker-ce的部署方案是否有缺陷，如果有缺陷，那些地方还可以改进？
