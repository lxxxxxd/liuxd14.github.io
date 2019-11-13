# IPAM Driver

在网络和端点生命周期中，CNM模型通过IPAM驱动程序控制网络和端点接口的IP地址分配。 Libnetwork具有默认的内置IPAM驱动程序，并允许动态插入第三方IPAM驱动程序。在创建网络时，用户可以指定哪个IPAM驱动程序libnetwork需要用于网络的IP地址管理。本文档说明了IPAM驱动程序需要遵循的API，以及与远程驱动程序相关的相应HTTPS请求/响应正文。

## 远程IPAM驱动程序

在远程网络驱动程序注册的同一行（有关更多详细信息，请参见remote.md），libnetwork使用Init（）函数初始化ipams.remote软件包。它传递一个ipamapi.Callback作为参数，该参数实现RegisterIpamDriver（）。远程驱动程序包通过在plugins.Handle回调中提供该接口，使用此接口向libnetwork的NetworkController注册远程驱动程序。远程驱动程序通过Docker插件包进行注册并与libnetwork通信。 ipams.remote提供了远程驱动程序进程的代理。

### Protocol (协议)

和远程网络驱动所用的协议一样

### 握手

在驱动程序注册期间，libnetwork将查询远程驱动程序的默认本地和全局地址空间字符串以及驱动程序功能。可以在本文档的相应部分中找到更详细的信息。

### 数据存储要求

远程驱动程序负责管理其数据库。

### IPAM规范

远程IPAM驱动程序必须满足以下请求：

    GetDefaultAddressSpaces

    RequestPool

    ReleasePool

    Request address

    Release address

以下各节说明上述每个请求的语义，何时在 network/endpoint 生命周期内调用它们以及远程驱动程序HTTP请求/响应的相应的payload。

### IPAM配置和流程

libnetwork用户可以在创建网络时通过NetworkOptionIpam setter函数提供与IPAM相关的配置。

    func NetworkOptionIpam（ipamDriver string，addrSpace string，ipV4 [] * IpamConf，ipV6 [] * IpamConf，opts map[string]string）NetworkOption

调用方必须提供IPAM驱动程序名称，并可以提供地址空间，用于IPv4的IpamConf结构列表和用于IPv6的列表。 IPAM驱动程序名称是唯一的必填字段。如果未提供，则网络创建将失败。

在配置列表中，每个元素具有以下形式：

    // IpamConf contains all the ipam related configurations for a network
     type IpamConf struct {
	// The master address pool for containers and network interfaces
	PreferredPool string
	// A subset of the master pool. If specified,
	// this becomes the container pool
	SubPool string
	// Input options for IPAM Driver (optional)
	Options map[string]string
	// Preferred Network Gateway address (optional)
	Gateway string
	// Auxiliary addresses for network driver. Must be within the master pool.
	// libnetwork will reserve them if they fall into the container pool
	AuxAddresses map[string]string
    }

创建网络时，libnetwork将迭代该列表，并对IPAM驱动程序执行以下请求：

1.请求地址池，并通过RequestPool（）传递选项。

2.请求网络网关地址（如果指定）。否则，请从池中请求任何地址用作网络网关。这是通过RequestAddress（）完成的。

3.通过RequestAddress（）请求每个指定的辅助地址。

如果IPv4配置列表为空，则libnetwork将自动添加一个空的IpamConf结构。这将导致libnetwork在配置的地址空间（如果指定）上或在IPAM驱动程序默认地址空间上，向IPAM驱动程序请求驱动程序选择的IPv4地址池。如果IPAM驱动程序无法提供地址池，则网络创建将失败。如果IPv6配置列表为空，则libnetwork将不执行任何操作。在执行点1）到3）的过程中从IPAM驱动程序检索到的数据将存储在网络结构中，作为IPv4的IpamInfo结构的列表和IPv6的列表。

创建端点时，libnetwork将遍历配置列表并执行以下操作：

1.从IPv4池请求IPv4地址，并将其分配给端点接口IPv4地址。如果成功，则停止迭代。

2.从IPv6池（如果存在）中请求一个IPv6地址，并将其分配给端点接口IPv6地址。如果成功，则停止迭代。

如果上述任何操作失败，则端点创建将失败

删除端点时，libnetwork将执行以下操作：

1.释放端点接口IPv4地址

2.释放端点接口IPv6地址（如果存在）

在删除网络时，libnetwork将迭代IpamData结构的列表，并对ipam驱动程序执行以下请求：

1.通过ReleaseAddress（）释放网络网关地址

2.通过ReleaseAddress（）释放每个辅助地址

3.通过ReleasePool（）释放池

### GetDefaultAddressSpaces

GetDefaultAddressSpaces返回此IPAM的默认本地和全局地址空间名称。地址空间是与其他地址空间的池隔离的一组不重叠的地址池。换句话说，同一池可以存在于N个不同的地址空间上。地址空间自然会映射到租户名称。在libnetwork中，与本地或全局地址空间相关联的含义是，本地地址空间不需要跨集群同步，而全局地址空间则需要同步。除非在IPAM配置中另外指定，否则libnetwork将根据所创建网络的范围从默认本地或默认全局地址空间请求地址池。例如，如果未在配置中另外指定，则libnetwork将从桥接网络的默认本地地址空间请求地址池，而从覆盖网络的默认全局地址空间请求地址池。

在注册过程中，远程驱动程序将收到没有payload的POST消息到URL /IpamDriver.GetDefaultAddressSpaces。驱动的响应应具有如下结构：
    {
	    "LocalDefaultAddressSpace": string
	    "GlobalDefaultAddressSpace": string
    }

### RequestPool

该API用于通过IPAM驱动程序注册地址池。多个相同的调用必须返回相同的结果。 IPAM驱动程序负责保留池的引用计数。

    RequestPool(addressSpace, pool, subPool string, options map[string]string, v6 bool) (string, *net.IPNet, map[string]string, error)

对于此API，远程驱动程序将使用以下payload接收到URL /IpamDriver.RequestPool的POST消息：

    {
	"AddressSpace": string
	"Pool":         string 
	"SubPool":      string 
	"Options":      map[string]string 
	"V6":           bool 
    }

注释：

*`AddressSpace` IP地址空间。它表示一组非重叠池。

*`Pool` CIDR格式的IPv4或IPv6地址池

*`SubPool`地址池的可选子集，CIDR格式的ip范围

*`Options` IPAM驱动程序特定选项的映射

*`V6` IPAM自选池是否应为IPv6

AddressSpace是唯一的必填字段。如果未指定池，则IPAM驱动程序可以选择返回自行选择的地址池。在这种情况下，如果呼叫者需要IPAM选择的IPv6池，则必须设置V6标志。具有空池和非空子池的请求应被拒绝为无效。如果未指定池，则IPAM将分配默认池之一。未指定Pool时，如果网络需要分配IPv6地址，则应设置V6标志。

成功的响应形式为：

    {
	"PoolID": string
	"Pool":   string
	"Data":   map[string]string
    }

注释：

PoolID是此池的标识符。相同的池必须具有相同的池ID。

池是CIDR格式的池

数据是该池的IPAM驱动程序提供的元数据

### ReleasePool

该API用于释放先前注册的地址池。

    ReleasePool（poolID string）error

对于此API，远程驱动程序将使用以下payload接收到URL /IpamDriver.ReleasePool的POST消息：

    {
    “ PoolID”：string
    }

注释：

PoolID是池标识符

成功的响应为空：

    {}

### RequestAddress

该API用于保留IP地址。

    RequestAddress(string, net.IP, map[string]string) (*net.IPNet, map[string]string, error)

对于此API，远程驱动程序将使用以下payload接收到URL /IpamDriver.RequestAddress的POST消息：

    {
	"PoolID":  string
	"Address": string
	"Options": map[string]string
    }

注释：

PoolID是池标识符

地址是常规IP格式（A.B.C.D）的必需地址。如果无法满足该地址，则请求失败。如果为空，则IPAM驱动程序选择池中的任何可用地址

选项是IPAM驱动程序特定的选项

成功的响应形式为：

    {
	"Address": string
	"Data":    map[string]string
    }

注释：

Address 是CIDR格式（A.B.C.D / MM）分配的地址

Data 是一些IPAM驱动程序特定的元数据

### ReleaseAddress

该API用于释放IP地址。

对于此API，远程驱动程序将使用以下负载接收到URL /IpamDriver.ReleaseAddress的POST消息：

    {
	"PoolID": string
	"Address": string
    }

注释：

PoolID是池标识符

Address是要发布的IP地址

### GetCapabilities

在驱动程序注册期间，libnetwork将查询驱动程序的功能。驱动程序不一定要支持此URL端点。如果驱动程序不支持，则注册将成功，并将自动添加到内部驱动程序句柄的空功能。

在注册过程中，远程驱动程序将收到没有有效负载的POST消息，指向URL /IpamDriver.GetCapabilities。驾驶员的回应应采用以下形式：

    {
        “ RequiresMACAddress”：bool
        “ RequiresRequestReplay”：bool
    }

### Capabilities

是远程ipam驱动程序在向libnetwork注册期间可以表达的能力。截至目前，libnetwork接受以下能力集合：

#### RequiresMACAddress

它是一个布尔值，它告诉libnetwork ipam驱动程序是否需要知道网卡的MAC地址才能正确处理RequestAddress（）调用。如果为true，则在CreateEndpoint（）请求时，libnetwork将为该端点生成一个随机MAC地址（如果用户尚未提供明确的MAC地址），并在选项映射内请求IP地址时将其传递给RequestAddress（），映射的key是netlabel.MacAddress常量：“ com.docker.network.endpoint.macaddress”。

#### RequiresRequestReplay

这是一个布尔值，它告诉libnetwork ipam驱动程序是否需要在重新加载守护程序时接收RequestPool（）和RequestAddress（）请求的响应。当libnetwork controller 正在初始化时，它会从本地存储中检索当前本地作用域网络的列表，如果设置了此功能标志，则它允许IPAM驱动程序通过为每个池和每个池响应RequestPool（）请求来重建池的数据库。本地网络拥有的每个网络网关的RequestAddress（）。这对于决定不保留分配给本地作用域的池的ipam驱动程序很有用。
    
