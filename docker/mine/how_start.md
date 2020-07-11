## 容器是怎么运行起来的

runc， linux
runc中容器的启动是状态的控制和对象的行为组成的，对象的行为中包含了进程的交互，调用系统调用对系统进行设置。

init.go加载nsexec, 在go runtime执行之前加载（C实现）。

go的runtime启动，执行各种init方法，其中包括用于初始化用于GOMAXPROCS变量和init命令执行后日志设置的init方法，注册runc-init命令响应的initCommand对象Action熟悉。

命令行对象响应run命令:
runc run busybox

构建逻辑容器阶段

启动逻辑容器

构建逻辑Parent进程

启动runc-init子进程

execv启动用户定义的进程
