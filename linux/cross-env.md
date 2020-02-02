## 搭建实验环境

实验环境使用qemu模拟器，qemu功能非常强大可以支持大多数的处理器架构。（可利用qemu-kvm进行加速）

qemu两种运行模式：

（1）user mode模拟模式：启动不同arch的linux程序。

（2）system mode模拟模式：模拟整个电脑系统，包括中央处理器和外围设备。这使得跨平台编写的程序的测试和调试工作变得很容易。

### 源码安装qemu

1.下载qemu源代码 

  wget https://download.qemu.org/qemu-2.11.0.tar.xz
  
2.解压源代码 

  tar xvJf qemu-2.11.0.tar.xz
  
3.配置 

  ./configure
  
4.编译安装 

  make -j4 && make install 
  
 ### 第一个实验
 
 
