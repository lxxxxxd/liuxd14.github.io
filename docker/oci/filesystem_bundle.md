翻译自：https://github.com/opencontainers/runtime-spec/blob/master/bundle.md

## 文件系统打包

### 容器格式

这一部分定义了用于以文件系统包编码容器的格式（把文件以某种方式进行组织，容纳了必要数据和元数据，来满足各种符合标准的容器运行时在上面执行标准的操作）。例如，MacOS应用bundle和这个也是类似的。

包的定义只需要考虑容器和他的配置如何存储在本地文件系统上，进而可以被符合标准的容器运行时所使用。

一个标准的容器包容纳了加载和运行容器需要的所有的信息。包括如下两条：

（1）config.json:包含配置数据。REQUIRED文件必须（MUST）存在于root包目录，必须（MUST）以config.json命名。更多细节参考config.json。

（2）容器的根文件系统：如果root.path属性被设置，root.path引用根文件系统路径。

bundle 
     |__config.json
     |__rootfs
