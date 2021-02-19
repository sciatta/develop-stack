# 概述

![architecture](Docker实践.assets/architecture.svg)

Docker使用的是client-server架构。Docker client同Docker daemon（守护进程）通信，Docker daemon负责构建、运行和分发Docker容器。Docker client和Docker daemon可以运行在同一个系统内，Docker client也可以连接到一个远程的Docker daemon。



## Docker daemon

Docker daemon（**dockerd**）监听Docker API请求，管理Docker对象（镜像，容器，网络和卷）。一个daemon也可以同其他daemons通信来管理Docker服务。



## Docker client

Docker client（**docker**）是Docker用户与Docker交互的主要方式。当使用 `docker run` 命令时，client会将这些命令发送给dockerd，dockerd会执行这些命令。docker命令使用docker API。Docker client可以与多个daemon通信。

- REST API 和后台运行进程交互
- CLI（command line interface）通过 REST API 和后台运行进程交互（docker命令）



## Docker registries

Docker registry存储Docker镜像。Docker Hub是一个公共的注册中心，任何人均可使用，Docker默认从Docker Hub中获取镜像。当使用 `docer pull` 或者 `docker run` 命令时，会从配置的registry拉取image。当使用 `docker push` 命令时，镜像会上传到配置的registry。



## Docker objects

### IMAGES 镜像

镜像是创建Docker容器的包含一系列指令的只读模板。为了创建自己的镜像，需要创建一个带有简单语法的Dockerfile，来定义如何创建和运行一个镜像。**在Dockerfile中的每一个指令创建镜像中的一层。当你改变Dockerfile，然后重建镜像，仅仅改变的那些层被重建。**



### CONTAINERS 容器

一个容器是一个镜像的运行实例。一个容器与其他容器，主机隔离。



### SERVICES 服务

允许跨多个Docker daemon来**扩展容器**，多个managers和workers作为一个**集群**工作。一个集群的每一个成员都是一个Docker daemon，这些daemon使用Docker API来通信。



# 命令

## docker attach

连接本地标准输入，输出和错误流到<font color=red>正在运行</font>的container。

- 以 `-i -t` 运行，<font color=red>使用 `CTRL+p+q` 与container分离</font>。
- `CTRL-c` 停止container

```shell
# -d Run container in background and print container ID (--detach)
# --name Assign a name to the container
# -i Keep STDIN open even if not attached
# -t Allocate a pseudo-TTY
docker run --name test -d -it debian
docker attach test
```



## docker build

从Dockerfile和上下文构建image。

上下文是指定 `PATH or URL` 参数的文件集合。构建进程可以引用上下文中的文件。可以指定一个Git仓库作为其**URL**，在本机首先拉取仓库到一个临时目录，成功后发送到Docker daemon作为其上下文。也可以指定本地文件系统的一个目录作为**PATH**。

- <font color=red>构建命令默认会在构建上下文的root下寻找Dockerfile，可以使用 `-f or --file` 指定代替Dockerfile，适用于一个目录下存在多个Dockerfile进行不同的构建</font>

```shell
# git repository # tag or branch（指定分支或tag，默认master） : /docker（指定上下文，默认/）
docker build https://github.com/docker/rootfs.git#container:docker

# PATH是.作为构建上下文
docker build .

# 下载tar.gz，ctx/Dockerfile是其内部Dockerfile的位置
# -f 指定Dockerfile，后面参数指定上下文
docker build -f ctx/Dockerfile http://server/ctx.tar.gz

# 从标准输入读入一个Dockerfile，没有上下文
docker build - < Dockerfile

# 从标准输入读入一个tar.gz
docker build - < context.tar.gz

# 2.0 为image打tag
# -t: 镜像的名字及标签，通常 name:tag 或者 name 格式；可以在一次构建中为一个镜像设置多个标签
docker build -t vieux/apache:2.0 .
```



## docker commit

以改变后的容器为基，创建一个新的image。

```shell
# c3f279d17e0a 容器id
# svendowideit/testimage REPOSITORY
# version3 TAG
docker commit c3f279d17e0a  svendowideit/testimage:version3
```



## docker container

管理容器。

### docker container attach

连接本地标准输入，输出和错误流到<font color=red>正在运行</font>的container。

### docker container commit

以改变后的容器为基，创建一个新的image。

### docker container cp

在容器和本地文件系统之间拷贝文件或文件夹。

### docker container create

创建一个新的容器。

### docker container exec

在一个运行的container内执行一个命令。

### docker container inspect

显示容器的详细信息。

### docker container kill

杀死一个或多个正在运行的container。

### docker container ls

列举container。

### docker container pause

暂停一个或多个container的所有进程。

### docker container port

列举容器的所有端口映射或一个指定映射。

### docker container rename

重命名一个容器。

### docker container restart

重启一个或多个容器。

### docker container rm

移除一个或多个容器。

### docker container run

在一个新的容器运行一个命令。

### docker container start

启动一个或多个停止的容器。

### docker container stats

显示容器实时资源使用情况统计信息。

### docker container stop

停止一个或多个正在运行的容器。

### docker container top

显示一个容器正在运行的进程。

### docker container unpause

取消暂停一个或多个container的所有进程。



## docker cp

在容器和本地文件系统之间拷贝文件或文件夹。

```shell
# 本地目录 -> 容器的/www/目录下
docker cp /www/runoob 96f7f14e99ab:/www/
# 本地目录 -> 容器的/目录下，改名为www
docker cp /www/runoob 96f7f14e99ab:/www
# 容器的/www目录 -> 本地目录
docker cp  96f7f14e99ab:/www /tmp/
```



## docker create

创建一个新的容器。

```shell
# 创建未运行容器
docker create -t -i fedora bash
# 启动
# -a attach
docker start -a -i 6d8af538ec5

# -v mount a volume
# --name Assign a name to the container
docker create -v /data --name data ubuntu
# --rm Automatically remove the container when it exits
# --volumes-from Mount volumes from the specified container(s)
docker run --rm --volumes-from data ubuntu ls -la /data

# -v 本地目录:容器卷
docker create -v /home/docker:/docker --name docker ubuntu
docker run --rm --volumes-from docker ubuntu ls -la /docker
```



## docker exec

在一个运行的container内执行一个命令。在容器内的默认目录，也可以通过Dockerfile的 `WORKDIR` 指令指定。

```shell
# 创建一个名称为ubuntu_bash的容器，并启动一个Bash会话
docker run --name ubuntu_bash --rm -i -t ubuntu bash
# 创建一个/tmp/execWorks文件在运行的ubuntu_bash容器
docker exec -d ubuntu_bash touch /tmp/execWorks
```



## docker image

管理镜像。

### docker image build

从Dockerfile和上下文构建image。

### docker image inspect

显示镜像的详细信息。

### docker image ls

列举image。

### docker image pull

从一个注册中心拉取一个image或一个repository 。

### docker image push

向一个注册中心推送一个image或一个repository 。

### docker image rm

删除一个或多个image。



## docker info

显示docker系统范围的信息。

```shell
docker info
```



## docker inspect

显示docker底层信息。

```shell
# 显示image详细配置信息
docker inspect imageid
```



## docker kill

杀死一个或多个正在运行的container。

```shell
docker kill my_container
```



## docker network

管理网络。

### docker network connect

将**正在运行的容器**连接到网络。一旦连接，在同一个网络中容器可以同其他容器通讯。

```shell
# 正在运行的容器连接到网络
docker network connect multi-host-network container1

# --network 启动容器并立即连接到网络
docker run -itd --network=multi-host-network busybox

# --ip 指定ip
docker network connect --ip 10.10.36.122 multi-host-network container2

# --link 连接到其他容器，c1指定的别名
docker network connect --link container1:c1 multi-host-network container2
```

### docker network create

创建一个网络。内建的网络驱动是 `bridge or overlay` ，如果不指定 `--driver` ，命令自动创建一个 `bridge` 网络。当安装Docker Engine后，系统自动创建一个 `docker0` bridge网络。当使用命令 `docker run` 启动一个新的容器，自动连接到bridge网络。此默认bridge网络不可以删除。

- `bridge` 网络在单个Engine上隔离网络。如果建立一个跨越多个Engine，必须创建 `overlay` 网络。

```shell
# -d driver
docker network create -d bridge my-bridge-network

# 172.28.0.0/16 前16位固定 172.28.0.0~172.28.255.255
# 172.28.5.0/24 前24位固定 172.28.5.0~172.28.5.255
docker network create \
  --driver=bridge \
  --subnet=172.28.0.0/16 \
  --ip-range=172.28.5.0/24 \
  --gateway=172.28.5.254 \
  br0
```

### docker network disconnect

断开容器与网络的连接。

```shell
docker network disconnect multi-host-network container1
```

### docker network inspect

显示网络的详细信息。

```shell
# 默认三个driver，name分别是bridge, host, none

# -o "com.docker.network.bridge.enable_icc"=true 启用或禁用容器间连接
# -o "com.docker.network.bridge.name"=docker0 虚拟网卡docker0
docker network inspect bridge
```

### docker network ls

列举network

```shell
docker network ls
```

### docker network rm

移除一个或多个network

```shell
docker network rm my-network
```



## docker pause

暂停一个或多个container的所有进程。

```shell
docker pause my_container
```



## docker port

列举容器的所有端口映射或一个指定映射。

```shell
# 7890/tcp -> 0.0.0.0:4321 容器端口 -> 本机端口
docker port test
```



## docker ps

列举容器。

```shell
# -a 显示所有容器。默认只显示正在运行的容器
docker ps -a
```



## docker pull

从一个注册中心拉取一个image或一个repository 。

```shell
# 默认拉取 latest
docker pull mysql:5.7.32
```



## docker push

向一个注册中心推送一个image或一个repository 。

```shell
# 创建image
docker container commit c16378f943fe rhel-httpd:latest
# 创建tag
# docker image tag source target
# 1. 为镜像创建标签，创建的新标签指向的是原镜像。
# 2. 在Docker Hub共享镜像，必须命名为<Docker Hub ID>/<Repository Name>:<tag>的样式。
docker image tag rhel-httpd:latest registry-host:5000/myadmin/rhel-httpd:latest
# 上传注册中心
docker image push registry-host:5000/myadmin/rhel-httpd:latest
```



## docker rename

重命名一个容器。

```shell
docker rename my_container my_new_container
```



## docker restart

重启一个或多个容器。

```shell
docker restart my_container
```



## docker rm

移除一个或多个容器。

```shell
docker rm /redis
```



## docker rmi

删除一个或多个image。

```shell
# 删除与id匹配的所有image
docker rmi -f fd484f19954f
```



## docker run

在一个新的容器运行命令。

- 如果本地没有ubuntu镜像，则从配置中心下载，相当于 `docker pull ubuntu`
- 创建一个新的容器，相当于 `docker container create`
- 分配一个可读写的文件系统给容器作为最后一层
- 创建一个网络接口，将容器连接到默认网络，为容器分配IP地址
- 启动容器，执行 `/bin/bash`
- 输入exit命令，容器stop。可以重新start或者remove容器

`-p -P --expose` 区别

- -p 端口映射，自动暴露
- --expose 暴露端口，但未做映射
- -P 会自动映射Dockerfile中 EXPOSE xxx 声明的端口到主机的任意端口

```shell
docker run --name test -it debian

# -w 设置工作目录
docker run -w /path/to/dir/ -i -t  ubuntu pwd

# -v 映射本地目录->容器目录
docker run -v `pwd`:`pwd` -w `pwd` -i -t ubuntu pwd

# 绑定本机的80端口到容器的8080端口
# ip:hostPort:containerPort，ip和hostPort可省略
# 发布的端口默认暴露
docker run -p 127.0.0.1:80:8080/tcp ubuntu bash

# --expose 暴露端口，但未做映射
docker run --expose 80 ubuntu bash

# 后台运行，返回容器id、
docker run -itd --name mysqlnode mysql:5.7.32
```



## docker search

在Docker Hub中查找镜像。

```shell
docker search busybox
```



## docker start

启动一个或多个停止的容器。

```shell
# -a Attach STDOUT/STDERR and forward signals
# -i Attach container’s STDIN
docker start my_container
```



## docker stop

停止一个或多个正在运行的容器。

```shell
docker stop my_container
```



## docker tag

为源镜像创建目标镜像的tag。

```shell
# SOURCE_IMAGE[:TAG] -> TARGET_IMAGE[:TAG]
docker tag 0e5574283393 fedora/httpd:version1.0
```



## docker top

显示一个容器正在运行的进程。



## docker unpause

取消暂停一个或多个container的所有进程。

```shell
docker unpause my_container
```



## docker version

显示docker的版本信息。

```shell
docker version
```



## docker volume

### docker volume create

创建卷。

```shell
docker volume create hello
# -v 挂载hello卷
# -d --driver 默认local
docker run -d -v hello:/world busybox ls /world
```

### docker volume inspect

显示卷的详细信息。

```shell
docker volume inspect myvolume
```

### docker volume ls

列举卷。

```shell
docker volume ls
```

### docker volume prune

删除所有未使用的local卷。

```shell
docker volume prune
```

### docker volume rm

删除一个或多个卷。

```shell
docker volume rm hello
```



# Dockerfile

- Docker daemon运行Dockerfile的每一条指令
- 每一条指令的结果提交到一个新的镜像。每一条指令独立运行，并且都会创建一个新的镜像
- Docker 在构建过程中会重用intermediate镜像，在构建过程中会显示`Using cache`
- 指令大小写不明感，但建议大写用以区分参数



## .dockerignore

Docker CLI 发送上下文到 docker daemon 之前，会在上下文根目录查找 `.dockerignore` 。如果存在，则会排除此文件匹配样式的文件和目录。如果确实需要，则使用 `ADD` or `COPY` 命令来增加到镜像。

- `# comment` 注释，被忽略
- `*/temp*` 以temp开头的文件或目录，在根目录的直接子目录内
- `*/*/temp*` 以temp开头的文件或目录，在根目录的二级子目录内
- `temp?` 根目录内以temp开头，后跟一个字符
- `**/*.go` 任何目录下以.go作为后缀
- `!README.md` 排除例外



## 指令

### Parser directives

- 可选，影响后续指令
- 构建时不会增加层
- 一个指令只允许被使用一次
- 一旦 `comment` ，空行， 构建指令被处理，docker不会再寻找Parser directives
- 大小写不明感，建议小写
- 在Parser directives之后，包含一个空行
- 不支持行连续字符 `\`
- `# directive=value`
  - `# escape=\` （默认）



### ENV

设置环境变量。

```shell
ENV FOO=/bar
WORKDIR ${FOO}   # WORKDIR /bar
```



### ARG

- `ARG` 处于构建之外，因此不可在 `FROM` 之后声明。但可以 `ARG VERSION` （不赋值）使用其默认值
- `ARG` 指令定义的变量，用户可以在构建阶段，通过 `docker build` 指令使用 `--build-arg <varname>=<value>` 覆盖已定义的变量



### FROM

- 必须以 `FROM` 开头
- `Parser directives` , `ARG` 和 `comment` 可以先于FROM指令

```shell
ARG  CODE_VERSION=latest
FROM base:${CODE_VERSION}
```



### RUN

构建期间当前层<font color=red>执行命令</font>，<font color=red>提交结果</font>被用于Dockerfile的下一步。

- `RUN <command>` (shell form)
  - 底层会调用/bin/sh -c来执行命令，可以解析变量
- `RUN ["executable", "param1", "param2"]` (exec form)
  - 不会调用shell



### CMD

在Dockerfile中只能有一个 `CMD` 指令。如果有多个，则最后一个起作用。 `CMD` 指令的主要目的是<font color=red>为运行容器提供默认行为，构建期间不会执行</font>。`docker run` 指定的参数会覆盖 `CMD` 指令的默认行为。

- `CMD ["executable","param1","param2"]` (exec form, this is the preferred form)
  - 不需要调用/bin/sh执行命令
- `CMD ["param1","param2"]` (as default parameters to ENTRYPOINT; omit the executable, in which case you must specify an `ENTRYPOINT` instruction)
  - 结合 `ENTRYPOINT` 使用，`ENTRYPOINT` 为默认指令，`CMD` 为其后拼接的参数；`CMD` 的参数可以被 `docker run` 代替
- `CMD command param1 param2` (shell form)
  - PID为1的进程并不是在Dockerfile里面定义CMD命令，而是/bin/sh命令。如果从外部发送任何POSIX信号到docker容器, 由于/bin/sh命令不会转发消息给实际运行的CMD命令，则不能安全得关闭docker容器。



### LABEL

为镜像添加元数据。

- 可被继承，可被覆盖

```shell
LABEL "com.example.vendor"="ACME Incorporated"
LABEL com.example.label-with-value="foo"
LABEL version="1.0"
LABEL description="This text illustrates \
that label-values can span multiple lines."
```



### EXPOSE

通知docker，容器在运行时监听的网络端口。只是向使用者声称打算发布的端口，但实际上并没有发布端口。`docker run` 通过 `-p` 暴露端口映射；通过 `-P` 会自动映射主机的临时高阶端口。

```shell
EXPOSE 80/tcp
EXPOSE 80/udp
```



### ADD

向镜像的文件系统添加文件，目录和远程URL文件。源目录比必须存在于构建上下文中。

- `ADD [--chown=<user>:<group>] <src>... <dest>`
- `ADD [--chown=<user>:<group>] ["<src>",... "<dest>"]` 可包含空格

```shell
ADD hom* /mydir/
```



### COPY

简版 `ADD` 指令，不支持URL。

- `COPY [--chown=<user>:<group>] <src>... <dest>`
- `COPY [--chown=<user>:<group>] ["<src>",... "<dest>"]`

```shell
COPY hom* /mydir/
```



### ENTRYPOINT

<font color=red>容器启动后执行命令的入口</font>。和CMD类似，默认的ENTRYPOINT也在docker run时，也可以被覆盖。在运行时用--entrypoint覆盖默认的ENTRYPOINT。

- `ENTRYPOINT ["executable", "param1", "param2"]` (exec form)
- `ENTRYPOINT command param1 param2` (shell form)



### VOLUME

创建挂载点，无法指定主机上对应的目录，由docker自动生成。

```shell
VOLUME /myvol
```



### USER

指定用户运行镜像中的命令。

- `USER`指令设置运行镜像时要使用的用户名（或UID）以及可选的用户组（或GID），以及Dockerfile中跟随该镜像的所有`RUN`，`CMD`和`ENTRYPOINT`指令。
- 如果用户没有组，则该镜像（或接下来指令）将用root组运行。

```shell
USER patrick
```



### WORKDIR

设置指令的工作目录。

```shell
WORKDIR /path/to/workdir
```



# 问题

## 解决macOS无法访问docker容器服务的问题

- 原因

  - 对于docker网络是bridge的情况，宿主机是Linux会创建docker0虚拟网卡，通讯通过虚拟网卡，宿主机和容器通讯完全支持；而宿主机是Mac，bridge桥接网络是运行在docker创建的一个Linux虚拟机中，因此，宿主机和容器无法通讯
  - Mac不支持host主机网络驱动程序，其只适用于Linux

- 解决

  解决问题的方案是 github 上的 docker-for-mac `https://github.com/wojas/docker-mac-network` 项目，主要方法是使用OpenVpn 来访问 docker。

  - 安装 `brew install tunnelblick`

  - 克隆 `git clone https://github.com/wojas/docker-mac-network.git`

  - **修改配置**

    - `cd /Users/yangxiaoyu/work/develop/star/docker-mac-network/helpers`

    - `vi run.sh`

      ```shell
      # 修改为实际的容器IP段和子网掩码
      route 172.82.0.0 255.255.255.0
      ```

  - 启动tunnelblick

    - `cd /Users/yangxiaoyu/work/develop/star/docker-mac-network`

    - <font color=red>必须保持启动</font> `docker-compose up -d` 

    - 修改新生成的文件 `vi docker-for-mac.ovpn`

      ```shell
      </tls-auth>
      comp-lzo yes	# 添加一行
      
      route 172.82.0.0 255.255.255.0
      ```

    - 双击 `docker-for-mac.ovpn` 将配置导入到tunnelblick

    - 打开tunnelblick客户端 | 菜单 | VPN 详情 | 连接

    - 验证

      ```shell
      ping 172.82.0.100
      telnet 172.82.0.100 6379
      ```

  - 重新生成

    - 删除文件 `config/*` 和`docker-for-mac.ovpn`
    - 修改配置
    - 启动tunnelblick

