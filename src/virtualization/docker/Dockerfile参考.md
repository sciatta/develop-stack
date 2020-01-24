# Dockerfile参考

# 概述



Docker从Dockerfile文件中读取指令，自动构建镜像。



`docker build` 从Dockerfile和上下文构建镜像。构建上下文是在指定位置 `PATH` 或 `URL` 下的文件集合。其中，PATH是本地文件系统的目录，URL是Git仓库的位置。上下文的处理是递归的。



```
docker build .
```



以当前目录作为上下文。



构建过程是运行在 Docker daemon，而并不是CLI。构建过程的第一件事是发送整个上下文（递归）到daemon。因此，最好是把Dockerfile放置在一个空目录作为上下文，仅增加对构建需要的文件。为了使用上下文下的文件，可以使用`COPY`指令。为了提高构建性能，可以在上下文目录增加一个`.dockerignore`文件来排除文件和目录。



# 注释



指令不是大小写敏感的，但按照惯例，把指令全部写成大写来与参数作区分。Docker按顺序运行Dockerfile内的指令。一个Dockerfile必须以FROM指令开始（可在解析器指令、注释、全局范围参数之后）。



Docker会将以＃开头的行视为注释，除非该行是有效的解析器指令。一行中其他任何地方的＃标记均被视为参数。注释中不支持换行符。



```
# Comment
RUN echo 'we are running some # of cool things'
```



第一个 # 为注释；第二个 # 被视为参数，正常输出 `we are running some # of cool things` 。



# 解析器指令



解析器指令是可选的，影响Dockerfile后续行的执行。解析器指令不会增加层，也不会显示为一个构件步骤。解析器指令被写成一个特殊的注释`# directive=value`。处理完注释，空行或构件指令后，Docker不再寻找解析器指令。 而是将格式化为解析器指令的任何内容都视为注释，并且不会尝试验证它是否可能是解析器指令。 因此，所有解析器指令必须位于Dockerfile的最顶部。



解析器指令不是大小写敏感的，但按照惯例，全部写成小写。约定还应在任何解析器指令之后包含一个空白行。 解析器指令不支持换行符。而非换行符的空格字符是支持的。



## syntax



```
# syntax=[remote image reference]
```



## escape



```
# escape=\ (backslash)
```



转义指令设置Dockerfile中的转义字符，如果没有指定，默认是 \ 。



# .dockerignore 文件

docker CLI 发送上下文到 docker daemon之前，会首先在上下文的root目录查找.dockerignore文件。如果存在，CLI会修改上下文，排除同模式匹配的文件和目录。这样可以避免发送不必要的大或敏感文件和目录到daemon。如果需要这些文件或目录的话，可以使用 `ADD` 或 `COPY` 指令。



CLI将.dockerignore文件解释为以换行符分隔的模式列表，类似于Unix shell的文件组。为了匹配，上下文的根被认为是工作目录和根目录。例如，模式 `/foo/bar` 和 `foo/bar` 都在PATH或位于URL的git仓库根目录的foo子目录中排除名为bar的文件或目录。



如果.dockerignore文件中第一行以 # 开始 ，则被看做是注释，在CLI解释之前被忽略。



| #    | 注释                        | # comment注释                                                |
| ---- | --------------------------- | ------------------------------------------------------------ |
| *    | 匹配任意字符                | */temp*匹配在root的直接子目录中，以temp为前缀的文件或目录  */*/temp* 匹配在root的二级子目录中，以temp为前缀的文件或目录 |
| **   | 匹配任意数量的目录，包括0个 | **/*.go排除所有目录以.go结尾的文件，包括root目录             |
| !    | 排除例外                    | !README.md包含文件README.md                                  |
| ?    | 匹配一个字符                | temp?匹配在root中以一个字符扩展temp的所有文件或目录          |



# FROM



```
FROM <image> [AS <name>]

FROM <image>[:<tag>] [AS <name>]

FROM <image>[@<digest>] [AS <name>]
```



一个有效的Dockerfile文件必须包含一个 `FROM` 指令。

- `ARG` 是唯一一个可以先于`FROM`的指令。
- `FROM`可以在单个Dockerfile中多次出现，以创建多个映像或将一个构建阶段用作对另一个构建阶段的依赖。 只需在每个新的`FROM`指令之前记录一次提交输出的最后一个镜像ID。 每个`FROM`指令清除由先前指令创建的任何状态。
- 可以增加 `AS` 到 `FROM` 指令，为一个新的构建阶段提供一个可选的名称。名称可以用于后续 `FROM` 和 `COPY --from=<name|index>` 指令，来在这个阶段引用镜像。
- `tag` 和 `digest` 值是可选的。如果省略，构建器默认latest tag。如果没有找到tag，则返回一个错误。



```
ARG VERSION=latest
FROM busybox:$VERSION
ARG VERSION
RUN echo $VERSION
```



VERSION定义先于`FROM`指令，因此在`FROM`指令之后，`ARG`不可以用于其他指令。在构建阶段如果需要使用`ARG`的默认值，则使用`ARG`指令不赋予值即可。



# RUN



```
RUN <command> 
以shell方式运行，默认 /bin/sh -c；
如：RUN /bin/bash -c 'source $HOME/.bashrc; echo $HOME'

RUN ["executable", "param1", "param2"]
以exec方式。注意双引号，JSON格式;
如：RUN ["/bin/bash", "-c", "echo hello"]
```



- 在当前镜像的顶层创建一个新层来执行命令，然后提交结果。
- 不像shell方式，以exec方式方式并不会调用命令shell。也就是说shell处理将不会发生。例如 `RUN [ "echo", "$HOME" ] `在 `$HOME `上将不会进行变量替换。如果需要进行shell处理，则以shell形式，或直接执行shell，如：`RUN [ "sh", "-c", "echo $HOME" ]`。会进行环境变量扩展，而不是docker。



# CMD



```
CMD ["executable","param1","param2"]
exec方式

CMD ["param1","param2"]
ENTRYPOINT的默认参数

CMD command param1 param2
shell方式
```



- 在Dockerfile中仅可以有一条 `CMD` 指令。如果有多条，仅有最后一条有效。
- 主要目的为正在执行容器提供默认执行方式。当运行镜像时，`CMD`指令被执行。
- 运行 `docker run` 指定参数，将覆盖 `CMD` 中的默认相同参数。
- 在构建阶段，`RUN` 执行一个命令，然后提交结果；而 `CMD` 并不在构建阶段执行，针对镜像的预期指定命令，即在容器执行时执行。



# LABEL



```
LABEL <key>=<value> <key>=<value> <key>=<value> ...
```



- 为镜像增加元数据。一个LABEL是一个 key-value 对。为了在LABEL的value中包括空字符，需要使用引号和反斜杠。
- 一个镜像可以有多个LABEL；可以指定多个LABEL在一行。
- LABEL可以包括在父镜像中，被用于继承。如果LABEL重复，后者覆盖前者。使用 `docker inspect` 查看LABEL。



```
LABEL "com.example.vendor"="ACME Incorporated"
LABEL com.example.label-with-value="foo"
LABEL version="1.0"
LABEL description="This text illustrates \
that label-values can span multiple lines."
```



# MAINTAINER (deprecated)



```
MAINTAINER <name>
```



生成镜像的作者。LABEL比MAINTAINER更灵活，建议使用LABEL替换。

例如 `LABEL maintainer="SvenDowideit@home.org.au"`



# EXPOSE



```
EXPOSE <port> [<port>/<protocol>...]
```



- 在运行时，容器监听的指定网络端口。可以指定端口监听 TCP 或是 UDP，如果协议没有指定，则TCP是默认协议。
- `EXPOSE` 指令实际并没有发布端口，它的主要功能是在构建镜像者和运行容器者之间作为一种规约，也就是预期要发布的端口。当运行容器时，实际要发布端口的话，使用 `-p` 标志发布并映射一个或多个端口，或 `-P` 发布所有 `EXPOSE` 的端口并映射到高阶端口。



```
EXPOSE 80/udp
```



默认TCP协议，可以指定协议为UDP。



- 不管 `EXPOSE` 如何设置，都可以在运行时使用 `-p` 标志覆盖。



# ENV



```
ENV <key> <value>
ENV <key>=<value> ...
```



- 在构建阶段，环境变量可以像使用变量一样用于后续的指令中，被Dockerfile所解析。环境变量可以在Dockerfile中以`$variable_name` 或 `${variable_name}` 方式来引用。
- 如果在变量前面加上转义字符 \ ，则按字面字符解释。如 \$foo 或者 \${foo}，被分别解释为 $foo 和 ${foo} 。
- 环境变量在运行时持续存在。可以使用`docker inspect` 查看。
- 可以在一个command中设置一个环境变量，使用 `RUN <key>=<value> <command>` 。



# ADD



```
ADD [--chown=<user>:<group>] <src>... <dest>

ADD [--chown=<user>:<group>] ["<src>",... "<dest>"]
1、路径包括空字符
2、--chown 只支持UNIX系统
```



- 从<src>复制文件，目录或者远程 URL，到镜像文件系统<dest>。
- 可以指定多个<src>资源，如果它们是文件或目录，则将其路径解释为相对于构建上下文的路径。



```
ADD hom* /mydir/
ADD hom?.txt /mydir/
```



- <dest> 可以是一个绝对路径，也可以是一个相对 `WORKDIR` 的相对路径。



```
ADD test relativeDir/          # adds "test" to `WORKDIR`/relativeDir/
ADD test /absoluteDir/         # adds "test" to /absoluteDir/
```



- <src>路径必须在构建上下文内部；不可使用../something /something，因为docker构建的第一步（FROM）已经把上下文目录发送到docker daemon。
- 如果<src>是一个URL，并且<dest>没有以斜杠结束，则文件从URL下载并复制到<dest>。
- 如果<src>是一个URL，并且<dest>以斜杠结束，则文件名按照URL进行推导，文件下载为<dest>/<filename>。如 `ADD http://example.com/foobar /` 将创建文件 `/foobar` 。
- 如果<src>是一个目录，则目录的所有内容被复制，包括文件系统的元数据。注意，目录本身不会被复制，仅是内容。
- 如果<src>是一个本地tar压缩格式，则被解压缩为一个目录。来自远程URL的资源不被解压缩。
- 如果多个<src>资源被指定，或者是目录，或者是通配符形式，则<dest>必须是一个目录，必须以 反斜杠结尾。
- 如果<dest>不存在，路径所有缺失的目录被创建。



# COPY



```
COPY [--chown=<user>:<group>] <src>... <dest>

COPY [--chown=<user>:<group>] ["<src>",... "<dest>"]
```



- 从<src>复制文件，目录，到容器文件系统<dest>。



# ENTRYPOINT



```
ENTRYPOINT ["executable", "param1", "param2"] (exec form, preferred)

ENTRYPOINT command param1 param2 (shell form)
```



- 配置容器，作为可执行方式运行。
- 以exec方式运行，命令行参数会追加到 `docker run <image>` ENTRYPOINT之后，并覆盖`CMD`指定的元素。可以搭配 `CMD` 命令使用，一般是变参会使用 `CMD` ，这里的 `CMD` 等于是在给 `ENTRYPOINT` 传参。可以使用ENTRYPOINT的exec方式，作为稳定默认命令和参数。然后使用两种形式的`CMD`设置更可能更改的其他默认值。



```
FROM nginx

ENTRYPOINT ["nginx", "-c"] # 定参
CMD ["/etc/nginx/nginx.conf"] # 变参 
```



```
docker run nginx:test
不传参运行，启动主进程 nginx -c /etc/nginx/nginx.conf

docker run nginx:test -c /etc/nginx/new.conf
传参运行，容器内会默认运行 nginx -c /etc/nginx/new.conf
```



- 允许向ENTRYPOINT传参。如 `docker run <image> -d` ，将 `-d` 传入到ENTRYPOINT。可以使用 `docker run --entrypoint` 覆盖 ENTRYPOINT指令。
- 以Shell方式运行，可防止使用任何 `CMD` 或 `run` 命令行参数，但缺点是ENTRYPOINT将作为`/bin/sh -c` 的子命令启动，该子命令不传递信号。这意味着executable将不是容器的`PID 1`，并且不会接收Unix信号，因此executable将不会从 `docker stop <container>` 接收到信号。
- 多个ENTRYPOINT指令，只有最后一个有效。
- ENTRYPOINT和CMD交互



|                            | No ENTRYPOINT              | ENTRYPOINT exec_entry p1_entry | ENTRYPOINT [“exec_entry”, “p1_entry”]          |
| -------------------------- | -------------------------- | ------------------------------ | ---------------------------------------------- |
| No CMD                     | error, not allowed         | /bin/sh -c exec_entry p1_entry | exec_entry p1_entry                            |
| CMD [“exec_cmd”, “p1_cmd”] | exec_cmd p1_cmd            | /bin/sh -c exec_entry p1_entry | exec_entry p1_entry exec_cmd p1_cmd            |
| CMD [“p1_cmd”, “p2_cmd”]   | p1_cmd p2_cmd              | /bin/sh -c exec_entry p1_entry | exec_entry p1_entry p1_cmd p2_cmd              |
| CMD exec_cmd p1_cmd        | /bin/sh -c exec_cmd p1_cmd | /bin/sh -c exec_entry p1_entry | exec_entry p1_entry /bin/sh -c exec_cmd p1_cmd |



# VOLUME



```
VOLUME ["/var/log/"]

VOLUME /var/log or VOLUME /var/log /var/db
```



- 创建一个指定名称的挂载点，可以将主机或其他容器的目录挂载到容器中。当容器删除时，数据被持久化而不会丢失。
- 主机目录是在容器运行时声明的：主机目录（挂载点）从本质上说是依赖于主机的。 这是为了保证镜像的可移植性，因为不能保证给定的主机目录在所有主机上都可用。 因此，您无法从Dockerfile中挂载主机目录。`VOLUME` 指令不支持指定`host-dir`参数。 当创建或运行容器时，必须指定挂载点。



# USER



```
USER <user>[:<group>]

USER <UID>[:<GID>]
```



- `USER`指令设置运行镜像时要使用的用户名（或UID）以及可选的用户组（或GID），以及Dockerfile中跟随该镜像的所有`RUN`，`CMD`和`ENTRYPOINT`指令。
- 如果用户没有组，则该镜像（或接下来指令）将用root组运行。



# WORKDIR



```
WORKDIR /path/to/workdir
```



- `WORKDIR`指令为Dockerfile中跟在其后的所有`RUN`，`CMD`，`ENTRYPOINT`，`COPY`和`ADD`指令设置工作目录。 如果`WORKDIR`不存在，即使后续的Dockerfile指令中未使用，`WORKDIR`也将被创建。
- `WORKDIR`指令可在Dockerfile中多次使用。 如果提供了相对路径，则它将相对于上一个`WORKDIR`指令的路径。



```
WORKDIR /a
WORKDIR b
WORKDIR c
RUN pwd
```



最后输出的`WORKDIR`是 `/a/b/c` 。



# ARG



```
ARG <name>[=<default value>]
```



- `ARG`指令定义的变量，用户可以在构建阶段，通过 `docker build` 指令使用 `--build-arg <varname>=<value>` 覆盖已定义的变量。