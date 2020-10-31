## Docker容器（二）解剖docker-ce软件包

在上一篇文章里面我重点介绍了在CentOS上如何部署docker-ce，这篇文章我将解剖软件包来探索软件包里面的组件，研究软件包的源头。

### 解剖软件包
我们要探索软件包里面的细节，就要知道常见的几种软件包格式和打包工具，比如说，debian系统上deb格式的软件包和apt工具，CentOS系统上的rpm格式软件包和rpm工具集等。承接上文，我们选取CentOS下rpm的docker-ce软件包进行探索。
回忆上一篇文章，我们使用如下命令来安装docker-ce所依赖的软件包：
```
sudo yum install docker-ce docker-ce-cli containerd.io
```
现在我们需要一个工具来查看这三个软件包里面都有什么内容。要找到这样的工具，我们只需要使用合适的关键字，在百度上进行搜索。选择什么样的关键字进行搜索呢？这依赖你拥有一些离答案更近的知识。
我们现在使用的是rpm的包格式，使用yum工具来安装软件包。因此，我们首先能想到的就是查看yum的help信息，或者查看rpm包管理工具集中是否有合适的命令可以使用。我对rpm是个外行，
我想很快速的完成我的目标，我就用百度搜索了“rpm查看安装文件”，很快找到一个的博客 https://blog.csdn.net/oLiHongMing/article/details/79951788, 博客给了一条命令：
```
rpm -qal {pkg-name}
```
如果你对rpm的包管理工具很熟练，大可不必像我这样。下面我们逐一对这三个软件包 docker-ce，docker-ce-cli和containerd.io进行解剖，对软件包里面的组件以注释的方式进行解释说明。
docker-ce:
```
# rpm -qal docker-ce
/usr/bin/docker-init                                    //？？？？
/usr/bin/docker-proxy                                   //封装了docker的网络实现方案libnetwork
/usr/bin/dockerd                                        //docker CLI的后端守护进程
/usr/lib/systemd/system/docker.service                  //systemd是linux里面进行应用管理的框架，docker本质也是一个应用，这里定义了docker这个应用的配置信息
/usr/lib/systemd/system/docker.socket                   //？？？？
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
/etc/containerd                                         //？？？？？
/etc/containerd/config.toml                             //containerd启动依赖的配置文件
/usr/bin/containerd                                     //二进制，dockerd的后端服务进程
/usr/bin/containerd-shim                                //二进制，容器运行时和容器生命周期管理的中间桥梁，封装了面向不同平台上的容器运行时的接口
/usr/bin/containerd-shim-runc-v1                        //containerd-shim的特殊版本
/usr/bin/ctr                                            //containerd的客户端，提供了操作CNI（容器运行时接口）的界面
/usr/bin/runc                                           //linux平台上容器运行时实现
/usr/lib/systemd/system/containerd.service              //containerd本质也是linux应用，使用systemd进行管理，这是配置文件
/usr/share/doc/containerd.io-1.2.13                     //版本发布说明文档
/usr/share/doc/containerd.io-1.2.13/README.md
/usr/share/licenses/containerd.io-1.2.13
/usr/share/licenses/containerd.io-1.2.13/LICENSE
/usr/share/man/man1/containerd-config.1
/usr/share/man/man1/containerd.1
/usr/share/man/man1/ctr.1
/usr/share/man/man5/containerd-config.toml.5
```

### 软件包源头

上文我们已经对docker-ce的软件包进行了解剖和说明，接下来我们需要研究软件包的源头：软件包是如何构建的？软件包构建依赖那些开源项目？
在开源社区，docker-ce在github上创建了一个开源项目，专门用于docker-ce软件包的发布，这里是项目地址：https://github.com/docker/docker-ce ，可惜这个项目已经过时了，现在最新的docker-ce版本20.10版本已经不打算在这里进行发布了，这个项目将会被归档。
|     组件名称     |             项目名称/项目地址            |               功能简述                |
|----------------|----------------------------------------|--------------------------------------|
|    docker      | cli/https://github.com:docker/cli      | docker-ce客户端命令行工具               |

为了构建出适用于某个操作系统平台上的软件包，docker社区也创建了一个开源项目，项目地址；https://github.com/docker/docker-ce-packaging ,

