翻译自：https://github.com/opencontainers/runtime-spec/blob/master/config.md

这个配置文件包含了在容器上实现标准操作所必要的元数据。这包括了执行集成，环境变量注入，沙箱特性的使用。

公认的的模板在这篇文档中将被定义，但是这里有一个json格式的模板[`schema/config-schema.json`](https://github.com/opencontainers/runtime-spec/blob/master/schema/config-schema.json)和go的二进制模板[`specs-go/config.go`](https://github.com/opencontainers/runtime-spec/blob/master/specs-go/config.go)。平台相关的配置模板在[`platform-specific documents`](https://github.com/opencontainers/runtime-spec/blob/master/config.md#platform-specific-configuration)进行了定义。对于某些属性只在一些平台上进行了定义，Go的属性有一个 platform 标签列举了这些协议。
下面是配置格式每一个属性的详细的描述和有效值。平台相关的配置值，定义在下面的 平台相关配置部分。

### 声明版本
**ociVersion** (string,REQUIRED) MUST 必须以 [`SemVer 2.0.0`](https://semver.org/spec/v2.0.0.html)格式声明OCI运行时bundle所符合的版本。开放容器倡议运行时规范遵循语义版本控制，并在主要版本中保留了向前和向后的兼容性。 例如，如果配置符合该规范的1.1版，则它与支持此规范的任何1.1或更高版本的所有运行时兼容，但与支持1.0而不是1.1的运行时不兼容。

`"ociVersion": "0.1.0"`

### Root

**root** (object, OPTIONAL) 声明了容器的根文件系统。在Windows上， Windows Server容器的这个属性是REQUIRED,对于[`Hyper-V 容器`](https://github.com/opencontainers/runtime-spec/blob/master/config-windows.md#hyperv)是 MUST NOT。

其他的平台，这个属性是REQUIRED
* **`path`** (string, REQUIRED) 定义容器的根文件系统的路径.
    * Windows 平台, `path` MUST 必须是 [volume GUID path][naming-a-volume].
    * POSIX 平台, `path` 要么是绝对路径，要么是相对路径.
        例如, 有一个bundle路径为 `/to/bundle`， 一个根文件系统为 `/to/bundle/rootfs`, `path` 值可以是 `/to/bundle/rootfs`，也可以是 `rootfs`.
        通常应该 SHOULD 是 `rootfs`.

    path声明的路径必须存在 MUST.

* **`readonly`** (bool, OPTIONAL) true 代表根文件系统在容器里面必须 MUST 是 read-only, 默认值是false.
    * Windows 平台, 这个值必须 MUST 忽略或者false.

### Example (POSIX platforms)

```json
"root": {
    "path": "rootfs",
    "readonly": true
}
```

### Example (Windows)

```json
"root": {
    "path": "\\\\?\\Volume{ec84d99e-3f02-11e7-ac6c-00155d7682cf}\\"
}
```

