翻译自：https://github.com/opencontainers/runtime-spec/blob/master/spec.md#notational-conventions

## OCI开发操作系统进程和应用容器的标准

### 摘要

目标：规范化容器的配置，执行环境和生命周期

config.json是支持了容器的平台的配置文件，细化了使能创建容器的属性。规范执行环境是为了确保在容器中运行的应用在各个runtime之间具有一致的环境。同时定义容器生命周期下有共同的动作。

平台

linux: runtime，config，config-linux, runtime-linux

solaris: runtime,config, config-solaris

windows: runtime, config, config-windows

vm: runtime, config config-vm

### 目录

概述

 × 现状
 
 × 容器原则
 
文件系统包

运行时和生命周期

 × 标准的linux运行时和生命周期
 
配置

 × linux配置标准
 
 × solaris配置标准
 
 × windows配置标准
 
 × 虚拟机配置标准
 
术语

### 现状

关键词 "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" 解释为 RFC 2119

关键词"unspecified", "undefined", and "implementation-defined" 的解释和 C99标准一致

如果一个CPU的架构实现不满足标准规定的一个或者多个MUST, REQUIRED, or SHALL要求，则不符合从标准。

反之，满足所有的MUST, REQUIRED, or SHALL的要求，则符合OCI标准。
