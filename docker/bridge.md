## 网桥驱动（Bridge Driver）

网桥驱动基于iptables和linux网桥来实现的，为容器提供网络连接。网桥驱动创建了一个默认名叫docker0的网桥。在每一个网桥和endpoit之间附加一个veth对。

## 配置

网桥驱动支持从Docker Daemon传进来的Flags。

## 使用

这个驱动支持默认的 “bridge”网络，不能被用作其他的网络。
