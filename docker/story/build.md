## Docker容器（二）软件包和组件

在上一篇文章里面我重点介绍了在CentOS上如何部署docker-ce，这篇文章我将解剖软件包来探索软件包里面的组件，研究组件由来和功能。

### 解剖软件包
我们要探索软件包里面的细节，就要知道常见的几种软件包格式和打包工具，比如说，debian系统上deb格式的软件包和apt工具，CentOS系统上的rpm格式软件包和rpm工具集等。承接上文，我们选取CentOS下rpm的docker-ce软件包进行探索。
回忆上一篇文章，我们使用如下命令来安装docker-ce所依赖的软件包：
```
sudo yum install docker-ce docker-ce-cli containerd.io
```
现在我们需要一个工具来查看这三个软件包里面都有什么内容。要找到这样的工具，我们只需要使用合适的关键字，在百度上进行搜索。选择什么样的关键字进行搜索呢？这依赖你拥有一些离答案更近的知识。
我们现在使用的是rpm的包格式，使用yum工具来安装软件包。因此，我们首先能想到的就是查看yum的help信息，或者查看rpm包管理工具集中是否有合适的命令可以使用。我对rpm是个外行，
我想很快速的完成我的目标，我就用百度搜索了“rpm查看安装文件”，很快找到一个的博客 https://blog.csdn.net/oLiHongMing/article/details/79951788，博客给了一条命令：rpm -qal {pkg-name}.
如果你对rpm的包管理工具很熟练，大可不必像我这样。下面我们逐一对这三个软件包 docker-ce，docker-ce-cli和containerd.io进行解剖。
docker-ce:
```
# rpm -qal docker-ce
/usr/bin/docker-init
/usr/bin/docker-proxy
/usr/bin/dockerd
/usr/lib/systemd/system/docker.service
/usr/lib/systemd/system/docker.socket
[root@iz2zebexqcz69hfh1b5avwz ~]# 
```
docker-ce-cli:
```
# rpm -qal docker-ce-cli                                //cli（command line interface）docker-ce-cli 封装了docker的命令行接口
/usr/bin/docker                                         //放在 /usr/bin/ 目录下，docker是二进制文件
/usr/libexec/docker/cli-plugins/docker-app              //从目录层级 docker/cli-plugins 来看应该是cli的一个插件
/usr/libexec/docker/cli-plugins/docker-buildx           //同上
/usr/share/bash-completion/completions/docker           
/usr/share/doc/docker-ce-cli-19.03.12
/usr/share/doc/docker-ce-cli-19.03.12/LICENSE           //文档，协议，项目维护者列表，公告，README
/usr/share/doc/docker-ce-cli-19.03.12/MAINTAINERS
/usr/share/doc/docker-ce-cli-19.03.12/NOTICE
/usr/share/doc/docker-ce-cli-19.03.12/README.md
/usr/share/fish/vendor_completions.d/docker.fish
/usr/share/man/man1/docker-attach.1.gz                  //manual手册
/usr/share/man/man1/docker-build.1.gz
...
```
containerd.io:
```
# rpm -qal containerd.io
/etc/containerd
/etc/containerd/config.toml
/usr/bin/containerd
/usr/bin/containerd-shim
/usr/bin/containerd-shim-runc-v1
/usr/bin/ctr
/usr/bin/runc
/usr/lib/systemd/system/containerd.service
/usr/share/doc/containerd.io-1.2.13
/usr/share/doc/containerd.io-1.2.13/README.md
/usr/share/licenses/containerd.io-1.2.13
/usr/share/licenses/containerd.io-1.2.13/LICENSE
/usr/share/man/man1/containerd-config.1
/usr/share/man/man1/containerd.1
/usr/share/man/man1/ctr.1
/usr/share/man/man5/containerd-config.toml.5
```
### 组件从何而来

### 组件基本功能介绍
