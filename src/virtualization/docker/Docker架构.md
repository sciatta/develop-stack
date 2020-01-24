# Docker架构

# 基础

- 什么是容器

​	作为容器，包含了开发、共享、部署的一切，包括代码，运行时，系统库，系统工具和相关的设置。因此，它可以快速运行，并可以在多台不同环境的计算机间无缝运行，避免解决各种因软件和硬件环境差异带来的各种开发和系统问题。



- 安装

​	登录DockerHub

​	下载Docker.dmg，并安装



- 卸载

​	在【应用程序】卸载docker

​	无法使用docker命令



# 架构

![1576639222645-f9f1fde9-e003-46ee-b382-58e794c4ad8a](Docker架构.assets/1576639222645-f9f1fde9-e003-46ee-b382-58e794c4ad8a.svg)



Docker使用的是client-server架构。Docker client同Docker daemon（守护进程）通信，Docker daemon负责构建、运行和分发Docker容器。Docker client和Docker daemon可以运行在同一个系统内，Docker client也可以连接到一个远程的Docker daemon。



​	The Docker daemon

​	Docker daemon（dockerd）监听Docker API请求，管理Docker对象（镜像，容器，网络和卷）。一个daemon也可以同其他daemons通信来管理Docker服务。



​	The Docker client

​	Docker client（docker）是Docker用户与Docker交互的主要方式。当您使用docker run等命令时，client会将这些命令发送给dockerd，dockerd会执行这些命令。docker命令使用docker API。Docker client可以与多个daemon通信。



​	Docker registries

​	Docker registry存储Docker镜像。Docker Hub是一个公共的注册中心，任何人均可使用，Docker默认从Docker Hub中获取镜像。



​	Docker objects

​	IMAGES 镜像

​	创建Docker容器的只读模板。为了创建自己的镜像，你需要创建一个Dockerfile，定义一些语法步骤，来创建和运行一个镜像。在Dockerfile中的每一个指令创建镜像中的一层。当你改变Dockerfile，然后重建镜像，仅仅改变的那些层被重建。

​	CONTAINERS 容器

​	一个容器是一个镜像的运行实例。一个容器与其他容器，主机隔离。

​	SERVICES 服务

​	服务允许跨多个Docker daemon来扩展容器，多个管理者和工作者作为一个集群工作。一个集群的每一个成员都是一个Docker daemon，这些daemon使用Docker API来通信。

