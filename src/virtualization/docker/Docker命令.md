# Docker命令

# 管理命令

## docker builder

## docker checkpoint

## docker config



## docker container

### docker container run



```
docker container run --publish 8000:8080 --detach --name tc ti:1.0

--detach: 在后台运行容器，打印容器ID
--name: 给容器指定一个名称
--publish: 发布容器的端口到主机，要求Docker将主机8000端口的流量转发到容器的8080端口
```



1. 在一个新容器中运行命令。其中，容器名称必须唯一。



### docker container ls



```
docker container ls -a

-a: 显示所有容器，默认仅显示正在运行的容器
```



1. 列举所有容器



### docker container rm



```
docker container rm --force tc

--force: 强制删除正在运行的容器
```



1. 删除一个或多个容器。



## docker context

## docker image

### docker image build



```
docker image build -t ti:1.0 .

-t: 镜像的名字及标签，通常 name:tag 或者 name 格式；可以在一次构建中为一个镜像设置多个标签
```



1. 执行Dockerfile文件命令，构建镜像。成功后显示 Successfully tagged testubuntu:1.0。



### docker image ls



```
docker image ls
```



1. 列举所有镜像



### docker image rm



```
docker image rm ti
```



1. 删除本地镜像文件，rm后可以是镜像名称，也可以是镜像 ID。



### docker image tag



```
docker image tag ti:1.0 sciatta/ti:1.0
```



1. 为镜像创建标签，创建的新标签指向的是原镜像。
2. 在Docker Hub共享镜像，必须命名为`<Docker Hub ID>/<Repository Name>:<tag>`的样式。



### docker image pull

### docker image push



```
docker image push sciatta/ti:1.0
```



1. 上传镜像到仓库。



## docker network

## docker node

## docker plugin

## docker secret

## docker service

## docker stack

## docker swarm

## docker system

## docker trust

## docker volume



# 命令

## docker attach

## docker build

## docker commit

## docker cp

## docker create

## docker deploy

## docker diff

## docker events

## docker exec



```
docker exec -it tc ps aux
```



在一个正在运行的容器内执行命令。



## docker export

## docker history

## docker images

## docker import

## docker info

## docker inspect

## docker kill

## docker load

## docker login



```
docker login
```



1. 输入用户名和密码，登录到docker仓库。默认是Docker Hub。如果没有Docker ID（用户名），需要在<https://hub.docker.com>注册。
2. 登录成功后显示Login Succeeded。



## docker logout

## docker logs

## docker pause

## docker port

## docker ps

## docker pull

## docker push

## docker rename

## docker restart

## docker rm

## docker rmi

## docker run



```
docker run -i -t sciatta/ti /bin/bash

-i: 以交互模式运行容器
-t: 为容器重新分配一个伪输入终端
```



1. 如果本机没有sciatta/ti镜像，从配置仓库pull镜像，同运行命令`docker pull`。
2. 创建一个新的容器，同运行命令`docker container create`。
3. Docker为容器创建一个可读写的文件系统作为最后一层。允许一个正在运行的容器，创建和修改本机文件系统的文件、目录。
4. 在没有指定网络选项时，Docker创建一个网络接口，连接容器到默认的网络，包括为容器分配一个IP地址。默认，容器可以使用主机的网络连接来连接到外部网络。
5. Docker启动容器，执行`/bin/bash`。因为容器以交互式运行，并且已经附加到终端 ，所以当日志输出到你的终端时，可以使用键盘输入。
6. 当输入`exit`退出`/bin/bash`命令后，容器停止运行，但容器没有被删除。可以重新启动或者删除容器。



## docker save

## docker search

## docker start

## docker stats

## docker stop

## docker tag

## docker top

## docker unpause

## docker update

## docker version

## docker wait



#  