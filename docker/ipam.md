## IPAM Driver

During the Network and Endpoints lifecycle, the CNM model controls the IP address assignment for network and endpoint interfaces via the IPAM driver(s). Libnetwork has a default, built-in IPAM driver and allows third party IPAM drivers to be dynamically plugged. On network creation, the user can specify which IPAM driver libnetwork needs to use for the network's IP address management. This document explains the APIs with which the IPAM driver
needs to comply, and the corresponding HTTPS request/response body relevant for remote drivers.

Remote IPAM driver
On the same line of remote network driver registration (see remote.md for more details), libnetwork initializes the ipams.remote package with the Init() function. It passes a ipamapi.Callback as a parameter, which implements RegisterIpamDriver(). The remote driver package uses this interface to register remote drivers with libnetwork's NetworkController, by supplying it in a plugins.Handle callback. The remote drivers register and communicate with libnetwork via the Docker plugin package. The ipams.remote provides the proxy for the remote driver processes.

在网络和端点生命周期中，CNM模型通过IPAM驱动程序控制网络和端点接口的IP地址分配。 Libnetwork具有默认的内置IPAM驱动程序，并允许动态插入第三方IPAM驱动程序。在创建网络时，用户可以指定哪个IPAM驱动程序libnetwork需要用于网络的IP地址管理。本文档说明了IPAM驱动程序需要遵循的API，以及与远程驱动程序相关的相应HTTPS请求/响应正文。

### 远程IPAM驱动程序

在远程网络驱动程序注册的同一行（有关更多详细信息，请参见remote.md），libnetwork使用Init（）函数初始化ipams.remote软件包。它传递一个ipamapi.Callback作为参数，该参数实现RegisterIpamDriver（）。远程驱动程序包通过在plugins.Handle回调中提供该接口，使用此接口向libnetwork的NetworkController注册远程驱动程序。远程驱动程序通过Docker插件包进行注册并与libnetwork通信。 ipams.remote提供了远程驱动程序进程的代理。
