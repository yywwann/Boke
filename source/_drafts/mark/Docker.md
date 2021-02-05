# Docker

- 学习路径

![](https://s0.lgstatic.com/i/image/M00/4C/CA/Ciqc1F9YoBKAP5TpAAHqwwYYWWc486.png)





### 卸载已有 Docker

```shell
$ sudo yum remove docker \

                  docker-client \

                  docker-client-latest \

                  docker-common \

                  docker-latest \

                  docker-latest-logrotate \

                  docker-logrotate \

                  docker-engine

```

## 安装 Docker

- 首次安装 Docker 之前，需要添加 Docker 安装源。添加之后，我们就可以从已经配置好的源，安装和更新 Docker。添加 Docker 安装源的命令如下：

  ```shell
  $ sudo yum-config-manager \
  
      --add-repo \
  
      https://download.docker.com/linux/centos/docker-ce.repo
  
  ```

  

- 正常情况下，直接安装最新版本的 Docker 即可，因为最新版本的 Docker 有着更好的稳定性和安全性。你可以使用以下命令安装最新版本的 Docker。

```shell
$ sudo yum install docker-ce docker-ce-cli containerd.io

```



- 安装完成后，使用以下命令启动 Docker。

```shell
$ sudo systemctl start docker
```



- 这里有一个国际惯例，安装完成后，我们需要使用以下命令启动一个 hello world 的容器。

```shell
$ sudo docker run hello-world

Unable to find image 'hello-world:latest' locally

latest: Pulling from library/hello-world

0e03bdcc26d7: Pull complete

Digest: sha256:7f0a9f93b4aa3022c3a4c147a449bf11e0941a1fd0bf4a8e6c9408b2600777c5

Status: Downloaded newer image for hello-world:latest

Hello from Docker!

```



1. Docker 首先会检查本地是否有`hello-world`这个镜像

   1. 如果发现本地没有这个镜像，Docker 就会去 Docker Hub 官方仓库下载此镜像
2. 有了，运行它



# 容器技术原理

- chroot 就是可以改变某进程的根目录，使这个程序不能访问目录之外的其他目录

- Docker 是利用 Linux 的 Namespace 、Cgroups 和联合文件系统三大机制来保证实现的， 所以它的原理是使用 Namespace 做主机名、网络、PID 等资源的隔离，使用 Cgroups 对进程或者进程组做资源（例如：CPU、内存等）的限制，联合文件系统用于镜像构建和容器运行环境。
- **Namespace** 是 Linux 内核的一项功能，该功能**对内核资源进行隔离**，使得容器中的进程都可以在单独的命名空间中运行，并且只可以访问当前容器命名空间的资源。Namespace 可以隔离进程 ID、主机名、用户 ID、文件名、网络访问和进程间通信等相关资源。
- **Cgroups** 是一种 Linux 内核功能，可以**限制和隔离进程的资源使用情况**（CPU、内存、磁盘 I/O、网络等）。在容器的实现中，Cgroups 通常用来限制容器的 CPU 和内存等资源的使用。
- **联合文件系统**，又叫 UnionFS，是一种通过创建文件层进程操作的文件系统，因此，联合文件系统非常轻快。Docker 使用联合文件系统为容器提供构建层，使得容器可以**实现写时复制以及镜像的分层构建和存储**。常用的联合文件系统有 AUFS、Overlay 和 Devicemapper 等。



# Docker 的操作围绕镜像、容器、仓库三大核心概念。

镜像是容器启动的基础，容器是镜像的运行实体，容器运行着真正的应用进程。容器有初建、运行、停止、暂停和删除五种状态。Docker 的镜像仓库类似于代码仓库，用来存储和分发 Docker 镜像。



镜像是容器的基石，容器是由镜像创建的。一个镜像可以创建多个容器，容器是镜像运行的实体。仓库就非常好理解了，就是用来存放和分发镜像的。





# Docker 架构

`OCI`全称为开放容器标准（Open Container Initiative），它是一个轻量级、开放的治理结构。

![](https://s0.lgstatic.com/i/image/M00/49/93/Ciqc1F9PYtCAC1GSAADIK4E6wrc368.png)

Docker 整体架构采用 C/S（客户端 / 服务器）模式，主要由客户端和服务端两大部分组成。客户端负责发送操作指令，服务端负责接收和处理指令。



Docker 的两个至关重要的组件：`runC`和`containerd`。

- `runC`是 Docker 官方按照 OCI 容器运行时标准的一个实现。通俗地讲，runC 是一个用来运行容器的轻量级工具，是真正用来运行容器的。
- `containerd`是 Docker 服务端的一个核心组件，它是从`dockerd`中剥离出来的 ，它的诞生完全遵循 OCI 标准，是容器标准化后的产物。`containerd`通过 containerd-shim 启动并管理 runC，可以说`containerd`真正管理了容器的生命周期。



containerd-shim 的意思是垫片，类似于拧螺丝时夹在螺丝和螺母之间的垫片。containerd-shim 的主要作用是将 containerd 和真正的容器进程解耦，使用 containerd-shim 作为容器进程的父进程，从而实现重启 dockerd 不影响已经启动的容器进程。



dockerd 启动的时候， containerd 就随之启动了，dockerd 与 containerd 一直存在。当执行 docker run 命令（通过 busybox 镜像创建并启动容器）时，containerd 会创建 containerd-shim 充当 “垫片”进程，然后启动容器的真正进程 sleep 3600 。





# 拉取镜像 docker pull

docker pull

docker pull [Registry]/[Repository]/[Image]:[Tag]



```
$ docker pull busybox
```



# docker 换源

```
$ vim /etc/docker/daemon.json
{
    "registry-mirrors":[
         "http://docker.mirrors.ustc.edu.cn",
         "http://hub-mirror.c.163.com",
         "http://registry.docker-cn.com"
    ] ,
    "insecure-registries":[
       "docker.mirrors.ustc.edu.cn",
         "registry.docker-cn.com"
    ]
} 
$ service docker restart
```



## 查看镜像

```
$ docker image ls
```





## 删除镜像

```
$ docker rmi mybusybox
```







# 构建镜像

构建镜像主要有两种方式：

1. 使用docker commit命令从运行中的容器提交为镜像；

2. 使用docker build命令从 Dockerfile 构建镜像。



1. 

```
$ docker commit busybox busybox:hello
```

2. 

```
FROM centos:7 # 第一行表示我要基于 centos:7 这个镜像来构建自定义镜像。

COPY nginx.repo /etc/yum.repos.d/nginx.repo # 第二行表示拷贝本地文件 nginx.repo 文件到容器内的 /etc/yum.repos.d 目录下。

RUN yum install -y nginx # 第三行表示在容器内运行yum install -y nginx命令，安装 nginx 服务到容器内，执行完第三行命令，容器内的 nginx 已经安装完成。

EXPOSE 80 # 第四行声明容器内业务（nginx）使用 80 端口对外提供服务。

ENV HOST=mynginx # 第五行定义容器启动时的环境变量 HOST=mynginx，容器启动后可以获取到环境变量 HOST 的值为 mynginx。

CMD ["nginx","-g","daemon off;"] # 第六行定义容器的启动命令，命令格式为 json 数组。这里设置了容器的启动命令为 nginx ，并且添加了 nginx 的启动参数 -g 'daemon off;' ，使得 nginx 以前台的方式启动。

```







# 镜像的实现原理：
镜像是由一系列的镜像层（layer ）组成，每一层代表了镜像构建过程中的一次提交，当我们需要修改镜像内的某个文件时，只需要在当前镜像层的基础上新建一个镜像层，并且只存放修改过的文件内容。分层结构使得镜像间共享镜像层变得非常简单和方便。







# 容器（Container）是什么？

容器启动有两种方式：

1. 使用docker start命令基于已经创建好的容器直接启动 。

2. 使用docker run命令直接基于镜像新建一个容器并启动，相当于先执行docker create命令从镜像创建容器，然后再执行docker start命令启动容器。



当使用docker run创建并启动容器时，Docker 后台执行的流程为：

1. Docker 会检查本地是否存在 busybox 镜像，如果镜像不存在则从 Docker Hub 拉取 busybox 镜像；

2. 使用 busybox 镜像创建并启动一个容器；

3. 分配文件系统，并且在镜像只读层外创建一个读写层；

4. 从 Docker IP 池中分配一个 IP 给容器；

5. 执行用户的启动命令运行镜像。



docker attach 多个终端窗口同步显示相同内容，不够灵活，会阻塞

所以我们是用 docker exec



容器导出后会成为一个镜像











# Docker Hub

yywwann

Yywwann111



```
$ docker pull busybox
```





# Dockerfile 书写原则

## （1）单一职责

## （2）提供注释信息

## （3）保持容器最小化

## （4）合理选择基础镜像

## （5）使用 .dockerignore 文件

## （6）尽量使用构建缓存

```
Docker 构建时判断是否需要使用缓存的规则如下：

从当前构建层开始，比较所有的子镜像，检查所有的构建指令是否与当前完全一致，如果不一致，则不使用缓存；

一般情况下，只需要比较构建指令即可判断是否需要使用缓存，但是有些指令除外（例如ADD和COPY）；

对于ADD和COPY指令不仅要校验命令是否一致，还要为即将拷贝到容器的文件计算校验和（根据文件内容计算出的一个数值，如果两个文件计算的数值一致，表示两个文件内容一致 ），命令和校验和完全一致，才认为命中缓存。

因此，基于 Docker 构建时的缓存特性，我们可以把不轻易改变的指令放到 Dockerfile 前面（例如安装软件包），而可能经常发生改变的指令放在 Dockerfile 末尾（例如编译应用程序）。

FROM centos:7
# 设置环境变量指令放前面
ENV PATH /usr/local/bin:$PATH
# 安装软件指令放前面
RUN yum install -y make
# 把业务软件的配置,版本等经常变动的步骤放最后
...
```

## （7）正确设置时区

Ubuntu 和Debian 系统可以向 Dockerfile 中添加以下指令：

```
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
RUN echo "Asia/Shanghai" >> /etc/timezone
```

CentOS 系统则向 Dockerfile 中添加以下指令：

```
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

## （8）使用国内软件源加快镜像构建速度

这里我以 CentOS 7 为例，介绍一下如何使用 163 软件源（国内有很多大厂，例如阿里、腾讯、网易等公司都免费提供的软件加速源）加快镜像构建。

首先在容器构建目录创建文件 CentOS7-Base-163.repo，文件内容如下：

```python
# CentOS-Base.repo
#
# The mirror system uses the connecting IP address of the client and the
# update status of each mirror to pick mirrors that are updated to and
# geographically close to the client.  You should use this for CentOS updates
# unless you are manually picking other mirrors.
#
# If the mirrorlist= does not work for you, as a fall back you can try the 
# remarked out baseurl= line instead.
#
#
[base]
name=CentOS-$releasever - Base - 163.com
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os
baseurl=http://mirrors.163.com/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7
#released updates
[updates]
name=CentOS-$releasever - Updates - 163.com
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates
baseurl=http://mirrors.163.com/centos/$releasever/updates/$basearch/
gpgcheck=1
gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7
#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras - 163.com
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras
baseurl=http://mirrors.163.com/centos/$releasever/extras/$basearch/
gpgcheck=1
gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7
#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus - 163.com
baseurl=http://mirrors.163.com/centos/$releasever/centosplus/$basearch/
gpgcheck=1
enabled=0
gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7
```

然后在 Dockerfile 中添加如下指令：

```
COPY CentOS7-Base-163.repo /etc/yum.repos.d/CentOS7-Base.repo
```

执行完上述步骤后，再使用yum install命令安装软件时就会默认从 163 获取软件包，这样可以大大提升构建速度。



## （9）最小化镜像层数

在构建镜像时尽可能地减少 Dockerfile 指令行数。例如我们要在 CentOS 系统中安装make和net-tools两个软件包，应该在 Dockerfile 中使用以下指令：



```
RUN yum install -y make net-tools
```

而不应该写成这样：

```
RUN yum install -y make
RUN yum install -y net-tools
```



#  Dockerfile 指令书写建议

## （1）RUN

RUN指令在构建时将会生成一个新的镜像层并且执行RUN指令后面的内容。

使用RUN指令时应该尽量遵循以下原则：

当RUN指令后面跟的内容比较复杂时，建议使用反斜杠（\） 结尾并且换行；

RUN指令后面的内容尽量按照字母顺序排序，提高可读性。

例如，我想在官方的 CentOS 镜像下安装一些软件，一个建议的 Dockerfile 指令如下：

```
FROM centos:7

RUN yum install -y automake \

                   curl \

                   python \

                   vim

```

## （2）CMD 和 ENTRYPOINT

`CMD`和`ENTRYPOINT`指令都是容器运行的命令入口，这两个指令使用中有很多相似的地方，但是也有一些区别。

这两个指令的相同之处，`CMD`和`ENTRYPOINT`的基本使用格式分为两种。

- 第一种为`CMD`/`ENTRYPOINT`["command" , "param"]。这种格式是使用 Linux 的`exec`实现的， 一般称为`exec`模式，这种书写格式为`CMD`/`ENTRYPOINT`后面跟 json 数组，也是Docker 推荐的使用格式。
- 另外一种格式为`CMD`/`ENTRYPOINT`command param ，这种格式是基于 shell 实现的， 通常称为`shell`模式。当使用`shell`模式时，Docker 会以 /bin/sh -c command 的方式执行命令。

> 使用 exec 模式启动容器时，容器的 1 号进程就是 CMD/ENTRYPOINT 中指定的命令，而使用 shell 模式启动容器时相当于我们把启动命令放在了 shell 进程中执行，等效于执行 /bin/sh -c "task command" 命令。因此 shell 模式启动的进程在容器中实际上并不是 1 号进程。

这两个指令的区别：

- Dockerfile 中如果使用了`ENTRYPOINT`指令，启动 Docker 容器时需要使用 --entrypoint 参数才能覆盖 Dockerfile 中的`ENTRYPOINT`指令 ，而使用`CMD`设置的命令则可以被`docker run`后面的参数直接覆盖。
- `ENTRYPOINT`指令可以结合`CMD`指令使用，也可以单独使用，而`CMD`指令只能单独使用。

看到这里你也许会问，我什么时候应该使用`ENTRYPOINT`,什么时候使用`CMD`呢？

如果你希望你的镜像足够灵活，推荐使用`CMD`指令。如果你的镜像只执行单一的具体程序，并且不希望用户在执行`docker run`时覆盖默认程序，建议使用`ENTRYPOINT`。

最后再强调一下，无论使用`CMD`还是`ENTRYPOINT`，都尽量使用`exec`模式。

## （3）ADD 和 COPY

`ADD`和`COPY`指令功能类似，都是从外部往容器内添加文件。但是`COPY`指令只支持基本的文件和文件夹拷贝功能，`ADD`则支持更多文件来源类型，比如自动提取 tar 包，并且可以支持源文件为 URL 格式。

那么在日常应用中，我们应该使用哪个命令向容器里添加文件呢？你可能在想，既然`ADD`指令支持的功能更多，当然应该使用`ADD`指令了。然而事实恰恰相反，我更推荐你使用`COPY`指令，因为`COPY`指令更加透明，仅支持本地文件向容器拷贝，而且使用`COPY`指令可以更好地利用构建缓存，有效减小镜像体积。

当你想要使用`ADD`向容器中添加 URL 文件时，请尽量考虑使用其他方式替代。例如你想要在容器中安装 memtester（一种内存压测工具），你应该避免使用以下格式：

```
ADD http://pyropus.ca/software/memtester/old-versions/memtester-4.3.0.tar.gz /tmp/

RUN tar -xvf /tmp/memtester-4.3.0.tar.gz -C /tmp

RUN make -C /tmp/memtester-4.3.0 && make -C /tmp/memtester-4.3.0 install

```

下面是推荐写法：

```
RUN wget -O /tmp/memtester-4.3.0.tar.gz http://pyropus.ca/software/memtester/old-versions/memtester-4.3.0.tar.gz \

&& tar -xvf /tmp/memtester-4.3.0.tar.gz -C /tmp \

&& make -C /tmp/memtester-4.3.0 && make -C /tmp/memtester-4.3.0 install

```



## （4）WORKDIR

为了使构建过程更加清晰明了，推荐使用 WORKDIR 来指定容器的工作路径，应该尽量避免使用 RUN cd /work/path && do some work 这样的指令。

最后给出几个常用软件的官方 Dockerfile 示例链接，希望可以对你有所帮助。

- [Go](https://github.com/docker-library/golang/blob/4d68c4dd8b51f83ce4fdce0f62484fdc1315bfa8/1.15/buster/Dockerfile)
- [Nginx](https://github.com/nginxinc/docker-nginx/blob/9774b522d4661effea57a1fbf64c883e699ac3ec/mainline/buster/Dockerfile)
- [Hy](https://github.com/hylang/docker-hylang/blob/f9c873b7f71f466e5af5ea666ed0f8f42835c688/dockerfiles-generated/Dockerfile.python3.8-buster)





























# **开篇词 | 溯本求源，吃透 Docker！**

你好，我是你的 Docker 老师——郭少。

我是从 2015 年开始使用和推广容器技术的，算是国内首批容器践行者。当时，我还在从事 Java 业务开发，业务内部的微服务需要做容器化改造，公司首席架构师牵头成立了云平台组，很荣幸我被选入该小组，从此我认识了 Docker。

刚开始接触时，我十分惊叹于 Docker 竟同时拥有业务隔离、软件标准交付等特性，而且又十分轻量，和虚拟机相比，容器化损耗几乎可以忽略不计。

接下来 5 年多的时间，我便在容器领域深耕，帮助过多家企业实现业务容器化，其间曾经在 360 推广容器云技术，实现了单集群数万个容器的规模，同时设计和开发了 Kubernetes 多集群管理平台 Wayne（有多家公司将 Wayne 用于生产环境）。2019 年，我被 CNCF 邀约作为嘉宾分享容器化实践经验，那时容器已经成为云计算的主流，以容器为代表的云原生技术，已经成为释放云价值的最短路径。

而在平时工作中，我仍然发现很多人在学习和实践 Docker 时，并非一路坦途：

- 学习 Docker 会顾及较多，比如，我不会 Golang 怎么办？Linux 懂一点行吗？
- 对 Docker 的知识掌握零零碎碎，不系统，说自己懂吧，但好像也懂得不多，还是经常会查资料。
- 自己对 Docker 底层原理理解欠缺，核心功能掌握不全，遇到问题时无法定位，耽误时间。
- 不知道如何使用 Docker 提升从开发到部署的效率？
- 不同场景下，如何选用最适合的容器编排和调度框架？

这些境遇恰是我曾走过的路，对此我也有很多感悟和思考，因此也一直希望有机会分享出来，这个课程正好是一个契机，相信我在这个行业实践的一些方法和思路能给你带来很多启发和帮助。

### 我是如何学习 Docker 的？

当今，Docker 技术已经形成了更为成熟的生态圈，各家公司都在积极做业务容器化改造，大家对 Docker 也都已经不再陌生。但在我刚接触 Docker 时，市面上的资料还非常少，甚至官网的资料也不太齐全。为了更深入地学习和了解 Docker，我只能从最笨但也最有效的方式入手，也就是读源码。

为什么说这是最笨的方法？因为想研究 Docker 源码，就意味着我需要学习一门新的编程语言 —— Golang。

虽然我当时已经掌握了一些编程语言，比如 Java、Scala、C 等，但对 Golang 的确十分陌生。好在 Golang 属于类 C 语言，当时我一边研究 Docker 源码，一边学习 Golang 语法。虽然学习过程有些艰辛，但结果很好。我只用了一周左右，便熟悉了这门新的编程语言，并从此与 Golang 和 Docker 结下了不解之缘。这可以说是我的另一层意外收获。

然而，在学习 Docker 源码的过程中我又发现，想要彻底了解 Docker 的底层原理，必须对 Linux 相关的技术有一定了解。例如，我们不了解 Linux 内核的 Cgroups 技术，就无法知道容器是如何做资源（CPU、内存等）限制的；不了解 Linux 的 Namespace 技术，就无法知道容器是如何做主机名、网络、文件等资源隔离的。

我记得有一次在生产环境中，告警系统显示一台机器状态为 NotReady，业务无法正常运行，登录机器发现运行`docker ps`命令无响应。这是当时线上 Docker 版本信息：

复制代码

```
$ docker -v
$ Docker version 17.03.2-ce, build f5ec1e2
$ docker-containerd -v
$ containerd version 0.2.3 commit:4ab9917febca54791c5f071a9d1f404867857fcc
$ docker-runc -v
$ runc version 1.0.0-rc2
$ commit: 54296cf40ad8143b62dbcaa1d90e520a2136ddfe
$ spec: 1.0.0-rc2-dev
```

这里简单介绍下我当时的排查过程。

我首先打开 Docker 的调试模式，查看详细日志，我根据调试日志去查找对应的 Docker 代码，发现是 dockerd 请求 containerd 无响应（这里你需要知道 Docker 组件构成和调用关系），然后发送 Linux`SIGUSR1`信号量（这里你需要知道 Linux 的一些信号量），打印 Golang 堆栈信息（这里你需要了解 Golang 语言）。最后结合内核 Cgroups 相关日志（这里你需要了解 Cgroups 的工作机制），才最终定位和解决问题。

可以看到，排查一个看起来很简单的问题就需要用到非常多的知识，**首先需要理解 Docker 架构，需要阅读 Docker 源码，还得懂一些 Linux 内核问题才能完全定位并解决问题。**

相信大多数了解 Docker 的人都知道，Docker 是基于 Linux Kernel 的 Namespace 和 Cgroups 技术实现的，但究竟什么是 Namespace？什么是 Cgroups？容器是如何一步步创建的？很多人可能都难以回答。你可能在想，我不用理会这些，照样可以正常使用容器呀，但如果你要真正在生产环境中使用容器，你就会发现如果不了解容器的技术原理，生产环境中遇到的问题你很难轻松解决。所以，**仅仅掌握容器的一些皮毛是远远不够的，需要我们了解容器的底层技术实现，结合生产实践经验，才能让我们更好地向上攀登**。

当然，我知道每个人的基础都不一样，所以在一开始规划这个课程的时候，我就和拉勾教育的团队一起定义好了我们的核心目标，就是“由浅入深带你吃透 Docker”，希望让不同基础的人都能在这个课程中收获满满。

### 送你一份“学习路径”

接下来，是我们为你画出的一个学习路径，这也是我们课程设计的核心。

![11.png](https://s0.lgstatic.com/i/image/M00/4C/CA/Ciqc1F9YoBKAP5TpAAHqwwYYWWc486.png)

用一句话总结，我希望这个课程从 Docker**基础知识点**到**底层原理**，再到**编排实践**，层层递进地展开介绍，最大程度帮你吸收和掌握 Docker 知识。

- **模块一：基础概念与操作**

在模块一，我首先会带你了解 Docker 基础知识以及一些基本的操作，比如拉取镜像，创建并启动容器等基本操作。这样可以让你对 Docker 有一个整体的认识，并且掌握 Docker 的基本概念和基本操作。这些内容可以满足你日常的开发和使用。

- **模块二：底层实现原理及关键技术**

在对 Docker 有个基本了解后，我们就进入重点部分—— Docker 的实现原理和关键性技术。比如，Namespace 和 Cgroups 原理剖析，Docker 是如何使用不同覆盖文件系统的（Overlay2、AUFS、Devicemapper），Docker 的网络模型等。当然，在这里我会趁热打铁，教你动手写一个精简版的 Docker，这能进一步加深你对 Docker 原理的认知。学习这些知识可以让你在生产环境中遇到问题时快速定位并解决问题。

- **模块三：编排技术三剑客**

仅仅有单机的容器只能解决基本的资源隔离需求，真正想在生产环境中大批量使用容器技术，还是需要有对容器进行调度和编排的能力。所以在这时，我会从 Dcoker Compose 到 Docker Swarm 再到 Kubernetes，一步步带你探索容器编排技术，这些知识可以让你在不同的环境中选择最优的编排框架。

- **模块四：综合实战案例**

在对容器技术原理和容器编排有一定了解后，我会教你将这些技术应用于 DevOps 中，最后会通过一个 CI/CD 实例让你了解容器的强大之处。

我希望这样的讲解框架，既能让你巩固基础的概念和知识，又能让你对 Docker 有更深一步的认识，同时也能让你体会容器结合编排后的强大力量。最重要的是，你不用再自己去研究这么多繁杂的技术点，不用再自己去头痛地读源码，因为这些事情正好我都提前帮你做了。

### 寄语

现阶段，很多公司的业务都在使用容器技术搭建自己的云平台，使用容器云来支撑业务运行也成为一种趋势，所以公司都会比较在意业务人员对 Docker 的掌握情况。那我希望这个课程，能够像及时雨一样，帮你彻底解决 Docker 相关的难题。

如果说，我们已经错过了互联网技术大爆发的时代，也没有在以虚拟机为代表的云计算时代分得一杯羹。那么，这次以 “容器” 为代表的历史变革正呼之欲出，你又有什么理由错过呢？

好了，我说了这么多，最后我也希望听你来说一说，告诉我：你在学习 Docker 的路上踩到过哪些坑？你在Docker的使用中又有哪些成功的经验，可以分享给大家？写在留言区，我们一起交流。





# **01 | Docker 安装：入门案例带你了解容器技术原理**

咱们第一课时就先聊聊 Docker 的基础内容：Docker 能做什么，怎么安装 Docker，以及容器技术的原理。

### Docker 能做什么？

众所周知，Docker 是一个用于开发，发布和运行应用程序的开放平台。通俗地讲，Docker 类似于集装箱。在一艘大船上，各种货物要想被整齐摆放并且相互不受到影响，我们就需要把各种货物进行集装箱标准化。有了集装箱，我们就不需要专门运输水果或者化学用品的船了。我们可以把各种货品通过集装箱打包，然后统一放到一艘船上运输。Docker 要做的就是把各种软件打包成一个集装箱（镜像），然后分发，且在运行的时候可以相互隔离。

到此，相信你已经迫不及待想要体验下了，下面就让我们来安装一个 Docker。

### CentOS 下安装 Docker

Docker 是跨平台的解决方案，它支持在当前主流的各大平台安装，包括 Ubuntu、RHEL、CentOS、Debian 等 Linux 发行版，同时也可以在 OSX 、Microsoft Windows 等非 Linux 平台下安装使用。

因为 Linux 是 Docker 的原生支持平台，所以推荐你在 Linux 上使用 Docker。由于生产环境中我们使用 CentOS 较多，下面主要针对在 CentOS 平台下安装和使用 Docker 展开介绍。

#### 操作系统要求

要安装 Docker，我们需要 CentOS 7 及以上的发行版本。建议使用`overlay2`存储驱动程序。

#### 卸载已有 Docker

如果你已经安装过旧版的 Docker，可以先执行以下命令卸载旧版 Docker。

复制代码

```
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

#### 安装 Docker

首次安装 Docker 之前，需要添加 Docker 安装源。添加之后，我们就可以从已经配置好的源，安装和更新 Docker。添加 Docker 安装源的命令如下：

复制代码

```
$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

正常情况下，直接安装最新版本的 Docker 即可，因为最新版本的 Docker 有着更好的稳定性和安全性。你可以使用以下命令安装最新版本的 Docker。

复制代码

```
$ sudo yum install docker-ce docker-ce-cli containerd.io
```

如果你想要安装指定版本的 Docker，可以使用以下命令：

复制代码

```
$ sudo yum list docker-ce --showduplicates | sort -r
docker-ce.x86_64            18.06.1.ce-3.el7                   docker-ce-stable
docker-ce.x86_64            18.06.0.ce-3.el7                   docker-ce-stable
docker-ce.x86_64            18.03.1.ce-1.el7.centos            docker-ce-stable
docker-ce.x86_64            18.03.0.ce-1.el7.centos            docker-ce-stable
docker-ce.x86_64            17.12.1.ce-1.el7.centos            docker-ce-stable
docker-ce.x86_64            17.12.0.ce-1.el7.centos            docker-ce-stable
docker-ce.x86_64            17.09.1.ce-1.el7.centos            docker-ce-stable
```

然后选取想要的版本执行以下命令：

复制代码

```
$ sudo yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
```

安装完成后，使用以下命令启动 Docker。

复制代码

```
$ sudo systemctl start docker
```

这里有一个国际惯例，安装完成后，我们需要使用以下命令启动一个 hello world 的容器。

复制代码

```
$ sudo docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
0e03bdcc26d7: Pull complete
Digest: sha256:7f0a9f93b4aa3022c3a4c147a449bf11e0941a1fd0bf4a8e6c9408b2600777c5
Status: Downloaded newer image for hello-world:latest
Hello from Docker!
```

运行上述命令，Docker 首先会检查本地是否有`hello-world`这个镜像，如果发现本地没有这个镜像，Docker 就会去 Docker Hub 官方仓库下载此镜像，然后运行它。最后我们看到该镜像输出 "Hello from Docker!" 并退出。

> 安装完成后默认 docker 命令只能以 root 用户执行，如果想允许普通用户执行 docker 命令，需要执行以下命令 sudo groupadd docker && sudo gpasswd -a ${USER} docker && sudo systemctl restart docker ，执行完命令后，退出当前命令行窗口并打开新的窗口即可。

安装完 Docker，先不着急使用，先来了解下容器的技术原理，这样才能知其所以然。

### 容器技术原理

提起容器就不得不说 chroot，因为 chroot 是最早的容器雏形。chroot 意味着切换根目录，有了 chroot 就意味着我们可以把任何目录更改为当前进程的根目录，这与容器非常相似，下面我们通过一个实例了解下 chroot。

#### chroot

什么是 chroot 呢？下面是 chroot 维基百科定义：

> chroot 是在 Unix 和 Linux 系统的一个操作，针对正在运作的软件行程和它的子进程，改变它外显的根目录。一个运行在这个环境下，经由 chroot 设置根目录的程序，它不能够对这个指定根目录之外的文件进行访问动作，不能读取，也不能更改它的内容。

通俗地说 ，chroot 就是可以改变某进程的根目录，使这个程序不能访问目录之外的其他目录，这个跟我们在一个容器中是很相似的。下面我们通过一个实例来演示下 chroot。

首先我们在当前目录下创建一个 rootfs 目录：

复制代码

```
$ mkdir rootfs
```

这里为了方便演示，我使用现成的 busybox 镜像来创建一个系统，镜像的概念和组成后面我会详细讲解，如果你没有 Docker 基础可以把下面的操作命令理解成在 rootfs 下创建了一些目录和放置了一些二进制文件。

复制代码

```
$ cd rootfs 
$ docker export $(docker create busybox) -o busybox.tar
$ tar -xf busybox.tar
```

执行完上面的命令后，在 rootfs 目录下，我们会得到一些目录和文件。下面我们使用 ls 命令查看一下 rootfs 目录下的内容。

复制代码

```
$ ls
bin  busybox.tar  dev  etc  home  proc  root  sys  tmp  usr  var
```

可以看到我们在 rootfs 目录下初始化了一些目录，下面让我们通过一条命令来见证 chroot 的神奇之处。使用以下命令，可以启动一个 sh 进程，并且把 /home/centos/rootfs 作为 sh 进程的根目录。

复制代码

```
$ chroot /home/centos/rootfs /bin/sh
```

此时，我们的命令行窗口已经处于上述命令启动的 sh 进程中。在当前 sh 命令行窗口下，我们使用 ls 命令查看一下当前进程，看是否真的与主机上的其他目录隔离开了。

复制代码

```
/ # /bin/ls /
bin  busybox.tar  dev  etc  home  proc  root  sys  tmp  usr  var
```

这里可以看到当前进程的根目录已经变成了主机上的 /home/centos/rootfs 目录。这样就实现了当前进程与主机的隔离。到此为止，一个目录隔离的容器就完成了。
但是，此时还不能称之为一个容器，为什么呢？你可以在上一步（使用 chroot 启动命令行窗口）执行以下命令，查看如下路由信息：

复制代码

```
/etc # /bin/ip route
default via 172.20.1.1 dev eth0
172.17.0.0/16 dev docker0 scope link  src 172.17.0.1
172.20.1.0/24 dev eth0 scope link  src 172.20.1.3
```

执行 ip route 命令后，你可以看到网络信息并没有隔离，实际上进程等信息此时也并未隔离。要想实现一个完整的容器，我们还需要 Linux 的其他三项技术： Namespace、Cgroups 和联合文件系统。

Docker 是利用 Linux 的 Namespace 、Cgroups 和联合文件系统三大机制来保证实现的， 所以它的原理是使用 Namespace 做主机名、网络、PID 等资源的隔离，使用 Cgroups 对进程或者进程组做资源（例如：CPU、内存等）的限制，联合文件系统用于镜像构建和容器运行环境。

后面我会对这些技术进行详细讲解，这里我就简单解释下它们的作用。

#### Namespace

Namespace 是 Linux 内核的一项功能，该功能对内核资源进行隔离，使得容器中的进程都可以在单独的命名空间中运行，并且只可以访问当前容器命名空间的资源。Namespace 可以隔离进程 ID、主机名、用户 ID、文件名、网络访问和进程间通信等相关资源。

Docker 主要用到以下五种命名空间。

- pid namespace：用于隔离进程 ID。
- net namespace：隔离网络接口，在虚拟的 net namespace 内用户可以拥有自己独立的 IP、路由、端口等。
- mnt namespace：文件系统挂载点隔离。
- ipc namespace：信号量,消息队列和共享内存的隔离。
- uts namespace：主机名和域名的隔离。

#### Cgroups

Cgroups 是一种 Linux 内核功能，可以限制和隔离进程的资源使用情况（CPU、内存、磁盘 I/O、网络等）。在容器的实现中，Cgroups 通常用来限制容器的 CPU 和内存等资源的使用。

#### 联合文件系统

联合文件系统，又叫 UnionFS，是一种通过创建文件层进程操作的文件系统，因此，联合文件系统非常轻快。Docker 使用联合文件系统为容器提供构建层，使得容器可以实现写时复制以及镜像的分层构建和存储。常用的联合文件系统有 AUFS、Overlay 和 Devicemapper 等。

### 结语

容器技术从 1979 年 chroot 的首次问世便已崭露头角，但是到了 2013 年，Dokcer 的横空出世才使得容器技术迅速发展，可见 Docker 对于容器技术的推动力和影响力。

> 另外， Docker 还提供了工具和平台来管理容器的生命周期：
>
> 1. 使用容器开发应用程序及其支持组件。
> 2. 容器成为分发和测试你的应用程序的单元。
> 3. 可以将应用程序作为容器或协调服务部署到生产环境中。无论您的生产环境是本地数据中心，云提供商还是两者的混合，其工作原理都相同。

到此，相信你已经了解了实现容器的基本技术原理，并且对 Docker 的作用有了一定认知。那么你知道为什么容器技术在 Docker 出现之前一直没有爆发的根本原因吗？思考后，可以把你的想法写在留言区。

下一课时，我将讲解 Docker 的架构设计以及 Docker 的三大核心概念。





# **02 | 核心概念：镜像、容器、仓库，彻底掌握 Docker 架构核心设计理念**

Docker 的操作围绕镜像、容器、仓库三大核心概念。在学架构设计之前，我们需要先了解 Docker 的三个核心概念。

### Docker 核心概念

#### 镜像

镜像是什么呢？通俗地讲，它是一个只读的文件和文件夹组合。它包含了容器运行时所需要的所有基础文件和配置信息，是容器启动的基础。所以你想启动一个容器，那首先必须要有一个镜像。**镜像是 Docker 容器启动的先决条件。**

如果你想要使用一个镜像，你可以用这两种方式：

1. 自己创建镜像。通常情况下，一个镜像是基于一个基础镜像构建的，你可以在基础镜像上添加一些用户自定义的内容。例如你可以基于`centos`镜像制作你自己的业务镜像，首先安装`nginx`服务，然后部署你的应用程序，最后做一些自定义配置，这样一个业务镜像就做好了。
2. 从功能镜像仓库拉取别人制作好的镜像。一些常用的软件或者系统都会有官方已经制作好的镜像，例如`nginx`、`ubuntu`、`centos`、`mysql`等，你可以到 [Docker Hub](https://hub.docker.com/) 搜索并下载它们。

#### 容器

容器是什么呢？容器是 Docker 的另一个核心概念。通俗地讲，容器是镜像的运行实体。镜像是静态的只读文件，而容器带有运行时需要的可写文件层，并且容器中的进程属于运行状态。即**容器运行着真正的应用进程。容器有初建、运行、停止、暂停和删除五种状态。**

虽然容器的本质是主机上运行的一个进程，但是容器有自己独立的命名空间隔离和资源限制。也就是说，在容器内部，无法看到主机上的进程、环境变量、网络等信息，这是容器与直接运行在主机上进程的本质区别。

#### 仓库

Docker 的镜像仓库类似于代码仓库，用来存储和分发 Docker 镜像。镜像仓库分为公共镜像仓库和私有镜像仓库。

目前，[Docker Hub](https://hub.docker.com/) 是 Docker 官方的公开镜像仓库，它不仅有很多应用或者操作系统的官方镜像，还有很多组织或者个人开发的镜像供我们免费存放、下载、研究和使用。除了公开镜像仓库，你也可以构建自己的私有镜像仓库，在第 5 课时，我会带你搭建一个私有的镜像仓库。

#### 镜像、容器、仓库，三者之间的联系

![Drawing 1.png](https://s0.lgstatic.com/i/image/M00/49/93/Ciqc1F9PYryALHVmAABihjRzo4c527.png)

从图 1 可以看到，镜像是容器的基石，容器是由镜像创建的。一个镜像可以创建多个容器，容器是镜像运行的实体。仓库就非常好理解了，就是用来存放和分发镜像的。

了解了 Docker 的三大核心概念，接下来认识下 Docker 的核心架构和一些重要的组件。

### Docker 架构

在了解 Docker 架构前，我先说下相关的背景知识——容器的发展史。

容器技术随着 Docker 的出现变得炙手可热，所有公司都在积极拥抱容器技术。此时市场上除了有 Docker 容器，还有很多其他的容器技术，比如 CoreOS 的 rkt、lxc 等。容器技术百花齐放是好事，但也出现了很多问题。比如容器技术的标准到底是什么？容器标准应该由谁来制定？

也许你可能会说， Docker 已经成为了事实标准，把 Docker 作为容器技术的标准不就好了？事实并没有想象的那么简单。因为那时候不仅有容器标准之争，编排技术之争也十分激烈。当时的编排技术有三大主力，分别是 Docker Swarm、Kubernetes 和 Mesos 。Swarm 毋庸置疑，肯定愿意把 Docker 作为唯一的容器运行时，但是 Kubernetes 和 Mesos 就不同意了，因为它们不希望调度的形式过度单一。

在这样的背景下，最终爆发了容器大战，`OCI`也正是在这样的背景下应运而生。

`OCI`全称为开放容器标准（Open Container Initiative），它是一个轻量级、开放的治理结构。`OCI`组织在 Linux 基金会的大力支持下，于 2015 年 6 月份正式注册成立。基金会旨在为用户围绕工业化容器的格式和镜像运行时，制定一个开放的容器标准。目前主要有两个标准文档：**容器运行时标准 （runtime spec）\**和\**容器镜像标准（image spec）**。

正是由于容器的战争，才导致 Docker 不得不在战争中改变一些技术架构。最终形成了下图所示的技术架构。

![Drawing 2.png](https://s0.lgstatic.com/i/image/M00/49/93/Ciqc1F9PYtCAC1GSAADIK4E6wrc368.png)

图2 Docker 架构图

我们可以看到，Docker 整体架构采用 C/S（客户端 / 服务器）模式，主要由客户端和服务端两大部分组成。客户端负责发送操作指令，服务端负责接收和处理指令。客户端和服务端通信有多种方式，既可以在同一台机器上通过`UNIX`套接字通信，也可以通过网络连接远程通信。

下面我逐一介绍客户端和服务端。

#### Docker 客户端

Docker 客户端其实是一种泛称。其中 docker 命令是 Docker 用户与 Docker 服务端交互的主要方式。除了使用 docker 命令的方式，还可以使用直接请求 REST API 的方式与 Docker 服务端交互，甚至还可以使用各种语言的 SDK 与 Docker 服务端交互。目前社区维护着 Go、Java、Python、PHP 等数十种语言的 SDK，足以满足你的日常需求。

#### Docker 服务端

Docker 服务端是 Docker 所有后台服务的统称。其中 dockerd 是一个非常重要的后台管理进程，它负责响应和处理来自 Docker 客户端的请求，然后将客户端的请求转化为 Docker 的具体操作。例如镜像、容器、网络和挂载卷等具体对象的操作和管理。

Docker 从诞生到现在，服务端经历了多次架构重构。起初，服务端的组件是全部集成在 docker 二进制里。但是从 1.11 版本开始， dockerd 已经成了独立的二进制，此时的容器也不是直接由 dockerd 来启动了，而是集成了 containerd、runC 等多个组件。

虽然 Docker 的架构在不停重构，但是各个模块的基本功能和定位并没有变化。它和一般的 C/S 架构系统一样，Docker 服务端模块负责和 Docker 客户端交互，并管理 Docker 的容器、镜像、网络等资源。

#### Docker 重要组件

下面，我以 Docker 的 18.09.2 版本为例，看下 Docker 都有哪些工具和组件。在 Docker 安装路径下执行 ls 命令可以看到以下与 docker 有关的二进制文件。

复制代码

```
-rwxr-xr-x 1 root root 27941976 Dec 12  2019 containerd
-rwxr-xr-x 1 root root  4964704 Dec 12  2019 containerd-shim
-rwxr-xr-x 1 root root 15678392 Dec 12  2019 ctr
-rwxr-xr-x 1 root root 50683148 Dec 12  2019 docker
-rwxr-xr-x 1 root root   764144 Dec 12  2019 docker-init
-rwxr-xr-x 1 root root  2837280 Dec 12  2019 docker-proxy
-rwxr-xr-x 1 root root 54320560 Dec 12  2019 dockerd
-rwxr-xr-x 1 root root  7522464 Dec 12  2019 runc
```

可以看到，Docker 目前已经有了非常多的组件和工具。这里我不对它们逐一介绍，因为在第 11 课时，我会带你深入剖析每一个组件和工具。
这里我先介绍一下 Docker 的两个至关重要的组件：`runC`和`containerd`。

- `runC`是 Docker 官方按照 OCI 容器运行时标准的一个实现。通俗地讲，runC 是一个用来运行容器的轻量级工具，是真正用来运行容器的。
- `containerd`是 Docker 服务端的一个核心组件，它是从`dockerd`中剥离出来的 ，它的诞生完全遵循 OCI 标准，是容器标准化后的产物。`containerd`通过 containerd-shim 启动并管理 runC，可以说`containerd`真正管理了容器的生命周期。

![Drawing 3.png](https://s0.lgstatic.com/i/image/M00/49/93/Ciqc1F9PYuuAQINxAAA236heaL0459.png)

图3 Docker 服务端组件调用关系图

通过上图，可以看到，`dockerd`通过 gRPC 与`containerd`通信，由于`dockerd`与真正的容器运行时，`runC`中间有了`containerd`这一 OCI 标准层，使得`dockerd`可以确保接口向下兼容。

> [gRPC](https://grpc.io/) 是一种远程服务调用。想了解更多信息可以参考[https://grpc.io](https://grpc.io/)
> containerd-shim 的意思是垫片，类似于拧螺丝时夹在螺丝和螺母之间的垫片。containerd-shim 的主要作用是将 containerd 和真正的容器进程解耦，使用 containerd-shim 作为容器进程的父进程，从而实现重启 dockerd 不影响已经启动的容器进程。

了解了 dockerd、containerd 和 runC 之间的关系，下面可以通过启动一个 Docker 容器，来验证它们进程之间的关系。

#### Docker 各组件之间的关系

首先通过以下命令来启动一个 busybox 容器：

复制代码

```
$ docker run -d busybox sleep 3600
```

容器启动后，通过以下命令查看一下 dockerd 的 PID：

复制代码

```
$ sudo ps aux |grep dockerd
root      4147  0.3  0.2 1447892 83236 ?       Ssl  Jul09 245:59 /usr/bin/dockerd
```

通过上面的输出结果可以得知 dockerd 的 PID 为 4147。为了验证图 3 中 Docker 各组件之间的调用关系，下面使用 pstree 命令查看一下进程父子关系：

复制代码

```
$ sudo pstree -l -a -A 4147
dockerd
  |-containerd --config /var/run/docker/containerd/containerd.toml --log-level info
  |   |-containerd-shim -namespace moby -workdir /var/lib/docker/containerd/daemon/io.containerd.runtime.v1.linux/moby/d14d20507073e5743e607efd616571c834f1a914f903db6279b8de4b5ba3a45a -address /var/run/docker/containerd/containerd.sock -containerd-binary /usr/bin/containerd -runtime-root /var/run/docker/runtime-runc
  |   |   |-sleep 3600
```

事实上，dockerd 启动的时候， containerd 就随之启动了，dockerd 与 containerd 一直存在。当执行 docker run 命令（通过 busybox 镜像创建并启动容器）时，containerd 会创建 containerd-shim 充当 “垫片”进程，然后启动容器的真正进程 sleep 3600 。这个过程和架构图是完全一致的。

#### 结语

本课时有基础、有架构，是一篇为后续打基础的文章。如果你有什么知识点没理解到位，有疑问，可写在留言处，我回复置顶，给他人参考。

如果你理解到位，相信你对 Docker 的三大核心概念镜像、容器、仓库有了一个清楚的认识，并对 Dokcer 的架构有了一定的了解。那么你知道为什么 Docker 公司要把`containerd`拆分并捐献给社区吗？思考后，也可以把你的想法写在留言区。





# **03 | 镜像使用：Docker 环境下如何配置你的镜像？**

今天我将围绕 Docker 核心概念镜像展开，首先重点讲解一下镜像的基本操作，然后介绍一下镜像的实现原理。首先说明，咱们本课时的镜像均指 Docker 镜像。

你是否还记得镜像是什么？我们先回顾一下。

镜像是一个只读的 Docker 容器模板，包含启动容器所需要的所有文件系统结构和内容。简单来讲，镜像是一个特殊的文件系统，它提供了容器运行时所需的程序、软件库、资源、配置等静态数据。即**镜像不包含任何动态数据，镜像内容在构建后不会被改变**。

然后我们来看下如何操作镜像。

### 镜像操作

![Lark20200904-175130.png](https://s0.lgstatic.com/i/image/M00/4A/AD/CgqCHl9SDkWAaxh7AAFaMgWI7cI029.png)

图 1 镜像操作

从图中可知，镜像的操作可分为：

- 拉取镜像，使用`docker pull`命令拉取远程仓库的镜像到本地 ；
- 重命名镜像，使用`docker tag`命令“重命名”镜像 ；
- 查看镜像，使用`docker image ls`或`docker images`命令查看本地已经存在的镜像 ；
- 删除镜像，使用`docker rmi`命令删除无用镜像 ；
- 构建镜像，构建镜像有两种方式。第一种方式是使用`docker build`命令基于 Dockerfile 构建镜像，也是我比较推荐的镜像构建方式；第二种方式是使用`docker commit`命令基于已经运行的容器提交为镜像。

下面，我们逐一详细介绍。

#### 拉取镜像

Docker 镜像的拉取使用`docker pull`命令， 命令格式一般为 docker pull [Registry]/[Repository]/[Image]:[Tag]。

- Registry 为注册服务器，Docker 默认会从 docker.io 拉取镜像，如果你有自己的镜像仓库，可以把 Registry 替换为自己的注册服务器。
- Repository 为镜像仓库，通常把一组相关联的镜像归为一个镜像仓库，`library`为 Docker 默认的镜像仓库。
- Image 为镜像名称。
- Tag 为镜像的标签，如果你不指定拉取镜像的标签，默认为`latest`。

例如，我们需要获取一个 busybox 镜像，可以执行以下命令：

> busybox 是一个集成了数百个 Linux 命令（例如 curl、grep、mount、telnet 等）的精简工具箱，只有几兆大小，被誉为 Linux 系统的瑞士军刀。我经常会使用 busybox 做调试来查找生产环境中遇到的问题。

复制代码

```
$ docker pull busybox
Using default tag: latest
latest: Pulling from library/busybox
61c5ed1cbdf8: Pull complete
Digest: sha256:4f47c01fa91355af2865ac10fef5bf6ec9c7f42ad2321377c21e844427972977
Status: Downloaded newer image for busybox:latest
docker.io/library/busybox:latest
```

实际上执行`docker pull busybox`命令，都是先从本地搜索，如果本地搜索不到`busybox`镜像则从 Docker Hub 下载镜像。

拉取完镜像，如果你想查看镜像，应该怎么操作呢？

#### 查看镜像

Docker 镜像查看使用`docker images`或者`docker image ls`命令。

下面我们使用`docker images`命令列出本地所有的镜像。

复制代码

```
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              4bb46517cac3        9 days ago          133MB
nginx               1.15                53f3fd8007f7        15 months ago       109MB
busybox             latest              018c9d7b792b        3 weeks ago         1.22MB
```

如果我们想要查询指定的镜像，可以使用`docker image ls`命令来查询。

复制代码

```
$ docker image ls busybox
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
busybox             latest              018c9d7b792b        3 weeks ago         1.22MB
```

当然你也可以使用`docker images`命令列出所有镜像，然后使用`grep`命令进行过滤。使用方法如下：

复制代码

```
$ docker images |grep busybox
busybox             latest              018c9d7b792b        3 weeks ago         1.22MB
```

#### “重命名”镜像

如果你想要自定义镜像名称或者推送镜像到其他镜像仓库，你可以使用`docker tag`命令将镜像重命名。`docker tag`的命令格式为 docker tag [SOURCE_IMAGE][:TAG] [TARGET_IMAGE][:TAG]。

下面我们通过实例演示一下：

复制代码

```
$ docker tag busybox:latest mybusybox:latest
```

执行完`docker tag`命令后，可以使用查询镜像命令查看一下镜像列表：

复制代码

```
docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
busybox             latest              018c9d7b792b        3 weeks ago         1.22MB
mybusybox           latest              018c9d7b792b        3 weeks ago         1.22MB
```

可以看到，镜像列表中多了一个`mybusybox`的镜像。但细心的同学可能已经发现，`busybox`和`mybusybox`这两个镜像的 IMAGE ID 是完全一样的。为什么呢？实际上它们指向了同一个镜像文件，只是别名不同而已。
如果我不需要`mybusybox`镜像了，想删除它，应该怎么操作呢？

#### 删除镜像

你可以使用`docker rmi`或者`docker image rm`命令删除镜像。

举例：你可以使用以下命令删除`mybusybox`镜像。

复制代码

```
$ docker rmi mybusybox
Untagged: mybusybox:latest
```

此时，再次使用`docker images`命令查看一下我们机器上的镜像列表。

复制代码

```
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
busybox             latest              018c9d7b792b        3 weeks ago         1.22MB
```

通过上面的输出，我们可以看到，`mybusybox`镜像已经被删除。
如果你想构建属于自己的镜像，应该怎么做呢？

#### 构建镜像

构建镜像主要有两种方式：

1. 使用`docker commit`命令从运行中的容器提交为镜像；
2. 使用`docker build`命令从 Dockerfile 构建镜像。

首先介绍下如何从运行中的容器提交为镜像。我依旧使用 busybox 镜像举例，使用以下命令创建一个名为 busybox 的容器并进入 busybox 容器。

复制代码

```
$ docker run --rm --name=busybox -it busybox sh
/ #
```

执行完上面的命令后，当前窗口会启动一个 busybox 容器并且进入容器中。在容器中，执行以下命令创建一个文件并写入内容：

复制代码

```
/ # touch hello.txt && echo "I love Docker. " > hello.txt
/ #
```

此时在容器的根目录下，已经创建了一个 hello.txt 文件，并写入了 "I love Docker. "。下面，我们新打开另一个命令行窗口，运行以下命令提交镜像：

复制代码

```
$ docker commit busybox busybox:hello
sha256:cbc6406aaef080d1dd3087d4ea1e6c6c9915ee0ee0f5dd9e0a90b03e2215e81c
```

然后使用上面讲到的`docker image ls`命令查看镜像：

复制代码

```
$ docker image ls busybox
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
busybox             hello               cbc6406aaef0        2 minutes ago       1.22MB
busybox             latest              018c9d7b792b        4 weeks ago         1.22MB
```

此时我们可以看到主机上新生成了 busybox:hello 这个镜像。

第二种方式是最重要也是最常用的镜像构建方式：Dockerfile。Dockerfile 是一个包含了用户所有构建命令的文本。通过`docker build`命令可以从 Dockerfile 生成镜像。

使用 Dockerfile 构建镜像具有以下特性：

- Dockerfile 的每一行命令都会生成一个独立的镜像层，并且拥有唯一的 ID；
- Dockerfile 的命令是完全透明的，通过查看 Dockerfile 的内容，就可以知道镜像是如何一步步构建的；
- Dockerfile 是纯文本的，方便跟随代码一起存放在代码仓库并做版本管理。

看到使用 Dockerfile 的方式构建镜像有这么多好的特性，你是不是已经迫不及待想知道如何使用了。别着急，我们先学习下 Dockerfile 常用的指令。

| Dockerfile 指令 | 指令简介                                                     |
| --------------- | ------------------------------------------------------------ |
| FROM            | Dockerfile 除了注释第一行必须是 FROM ，FROM 后面跟镜像名称，代表我们要基于哪个基础镜像构建我们的容器。 |
| RUN             | RUN 后面跟一个具体的命令，类似于 Linux 命令行执行命令。      |
| ADD             | 拷贝本机文件或者远程文件到镜像内                             |
| COPY            | 拷贝本机文件到镜像内                                         |
| USER            | 指定容器启动的用户                                           |
| ENTRYPOINT      | 容器的启动命令                                               |
| CMD             | CMD 为 ENTRYPOINT 指令提供默认参数，也可以单独使用 CMD 指定容器启动参数 |
| ENV             | 指定容器运行时的环境变量，格式为 key=value                   |
| ARG             | 定义外部变量，构建镜像时可以使用 build-arg = 的格式传递参数用于构建 |
| EXPOSE          | 指定容器监听的端口，格式为 [port]/tcp 或者 [port]/udp        |
| WORKDIR         | 为 Dockerfile 中跟在其后的所有 RUN、CMD、ENTRYPOINT、COPY 和 ADD 命令设置工作目录。 |

看了这么多指令，感觉有点懵？别担心，我通过一个实例让你来熟悉它们。这是一个 Dockerfile：

复制代码

```
FROM centos:7
COPY nginx.repo /etc/yum.repos.d/nginx.repo
RUN yum install -y nginx
EXPOSE 80
ENV HOST=mynginx
CMD ["nginx","-g","daemon off;"]
```

好，我来逐行分析一下上述的 Dockerfile。

- 第一行表示我要基于 centos:7 这个镜像来构建自定义镜像。这里需要注意，每个 Dockerfile 的第一行除了注释都必须以 FROM 开头。
- 第二行表示拷贝本地文件 nginx.repo 文件到容器内的 /etc/yum.repos.d 目录下。这里拷贝 nginx.repo 文件是为了添加 nginx 的安装源。
- 第三行表示在容器内运行`yum install -y nginx`命令，安装 nginx 服务到容器内，执行完第三行命令，容器内的 nginx 已经安装完成。
- 第四行声明容器内业务（nginx）使用 80 端口对外提供服务。
- 第五行定义容器启动时的环境变量 HOST=mynginx，容器启动后可以获取到环境变量 HOST 的值为 mynginx。
- 第六行定义容器的启动命令，命令格式为 json 数组。这里设置了容器的启动命令为 nginx ，并且添加了 nginx 的启动参数 -g 'daemon off;' ，使得 nginx 以前台的方式启动。

上面这个 Dockerfile 的例子基本涵盖了常用的镜像构建指令，代码我已经放在 [GitHub](https://github.com/wilhelmguo/docker-demo/tree/master/dockerfiles)上，如果你感兴趣可以到 [GitHub 下载源码](https://github.com/wilhelmguo/docker-demo/tree/master/dockerfiles)并尝试构建这个镜像。

学习了镜像的各种操作，下面我们深入了解一下镜像的实现原理。

### 镜像的实现原理

其实 Docker 镜像是由一系列镜像层（layer）组成的，每一层代表了镜像构建过程中的一次提交。下面以一个镜像构建的 Dockerfile 来说明镜像是如何分层的。

复制代码

```
FROM busybox
COPY test /tmp/test
RUN mkdir /tmp/testdir
```

上面的 Dockerfile 由三步组成：

第一行基于 busybox 创建一个镜像层；

第二行拷贝本机 test 文件到镜像内；

第三行在 /tmp 文件夹下创建一个目录 testdir。

为了验证镜像的存储结构，我们使用`docker build`命令在上面 Dockerfile 所在目录构建一个镜像：

复制代码

```
$ docker build -t mybusybox .
```

这里我的 Docker 使用的是 overlay2 文件驱动，进入到`/var/lib/docker/overlay2`目录下使用`tree .`命令查看产生的镜像文件：

复制代码

```
$ tree .
# 以下为 tree . 命令输出内容
|-- 3e89b959f921227acab94f5ab4524252ae0a829ff8a3687178e3aca56d605679
|   |-- diff  # 这一层为基础层，对应上述 Dockerfile 第一行，包含 busybox 镜像所有文件内容，例如 /etc,/bin,/var 等目录
... 此次省略部分原始镜像文件内容
|   `-- link 
|-- 6591d4e47eb2488e6297a0a07a2439f550cdb22845b6d2ddb1be2466ae7a9391
|   |-- diff   # 这一层对应上述 Dockerfile 第二行，拷贝 test 文件到 /tmp 文件夹下，因此 diff 文件夹下有了 /tmp/test 文件
|   |   `-- tmp
|   |       `-- test
|   |-- link
|   |-- lower
|   `-- work
|-- backingFsBlockDev
|-- bec6a018080f7b808565728dee8447b9e86b3093b16ad5e6a1ac3976528a8bb1
|   |-- diff  # 这一层对应上述 Dockerfile 第三行，在 /tmp 文件夹下创建 testdir 文件夹，因此 diff 文件夹下有了 /tmp/testdir 文件夹
|   |   `-- tmp
|   |       `-- testdir
|   |-- link
|   |-- lower
|   `-- work
...
```

通过上面的目录结构可以看到，Dockerfile 的每一行命令，都生成了一个镜像层，每一层的 diff 夹下只存放了增量数据，如图 2 所示。

![Lark20200904-175137.png](https://s0.lgstatic.com/i/image/M00/4A/AD/CgqCHl9SDmGACBEjAABkgtnn_hE625.png)

图 2 镜像文件系统

分层的结构使得 Docker 镜像非常轻量，每一层根据镜像的内容都有一个唯一的 ID 值，当不同的镜像之间有相同的镜像层时，便可以实现不同的镜像之间共享镜像层的效果。

总结一下， Docker 镜像是静态的分层管理的文件组合，镜像底层的实现依赖于联合文件系统（UnionFS）。充分掌握镜像的原理，可以帮助我们在生产实践中构建出最优的镜像，同时也可以帮助我们更好地理解容器和镜像的关系。

#### 总结

到此，相信你已经对 Docker 镜像这一核心概念有了较深的了解，并熟悉了 Docker 镜像的常用操作（拉取、查看、“重命名”、删除和构建自定义镜像）及底层实现原理。

本课时内容精华，我帮你总结如下：

> 镜像操作命令：
>
> 1. 拉取镜像，使用 docker pull 命令拉取远程仓库的镜像到本地 ；
> 2. 重命名镜像，使用 docker tag 命令“重命名”镜像 ；
> 3. 查看镜像，使用 docker image ls 或 docker images 命令查看本地已经存在的镜像；
> 4. 删除镜像，使用 docker rmi 命令删除无用镜像 ；
> 5. 构建镜像，构建镜像有两种方式。第一种方式是使用 docker build 命令基于 Dockerfile 构建镜像，也是我比较推荐的镜像构建方式；第二种方式是使用 docker commit 命令基于已经运行的容器提交为镜像。
>
> 镜像的实现原理：
> 镜像是由一系列的镜像层（layer ）组成，每一层代表了镜像构建过程中的一次提交，当我们需要修改镜像内的某个文件时，只需要在当前镜像层的基础上新建一个镜像层，并且只存放修改过的文件内容。分层结构使得镜像间共享镜像层变得非常简单和方便。

最后试想下，如果有一天我们机器存储空间不足，那你知道使用什么命令可以清理本地无用的镜像和容器文件吗？思考后，可以把你的想法写在留言区。

[点击即可查看本课时相关源码](https://github.com/wilhelmguo/docker-demo/tree/master/dockerfiles)





# **04 | 容器操作：得心应手掌握 Docker 容器基本操作**

前几天在咱们的社群里看到有同学在讨论，说面试的时候被问到容器和镜像的区别，有同学回答说没什么区别，也许是在开玩笑，不过这两者的区别很大。今天，我们就来看看容器的相关知识，比如什么是容器？容器的生命周期，以及容器常用的操作命令。学完之后你可以对比下与镜像的区别。

### 容器（Container）是什么？

容器是基于镜像创建的可运行实例，并且单独存在，一个镜像可以创建出多个容器。运行容器化环境时，实际上是在容器内部创建该文件系统的读写副本。 这将添加一个容器层，该层允许修改镜像的整个副本。如图 1 所示。

![image.png](https://s0.lgstatic.com/i/image/M00/4C/D1/CgqCHl9YmlSAGgF0AABXUH--rM4624.png)

图1 容器组成

了解完容器是什么，接下来我们聊一聊容器的生命周期。

### 容器的生命周期

容器的生命周期是容器可能处于的状态，容器的生命周期分为 5 种。

1. created：初建状态
2. running：运行状态
3. stopped：停止状态
4. paused： 暂停状态
5. deleted：删除状态

各生命周期之前的转换关系如图所示：

![Lark20200923-114857.png](https://s0.lgstatic.com/i/image/M00/55/BF/CgqCHl9qxcuANmQGAADHS_nfwJE810.png)

图2 容器的生命周期

通过`docker create`命令生成的容器状态为初建状态，初建状态通过`docker start`命令可以转化为运行状态，运行状态的容器可以通过`docker stop`命令转化为停止状态，处于停止状态的容器可以通过`docker start`转化为运行状态，运行状态的容器也可以通过`docker pause`命令转化为暂停状态，处于暂停状态的容器可以通过`docker unpause`转化为运行状态 。处于初建状态、运行状态、停止状态、暂停状态的容器都可以直接删除。

下面我通过实际操作和命令来讲解容器各生命周期间的转换关系。

### 容器的操作

容器的操作可以分为五个步骤：创建并启动容器、终止容器、进入容器、删除容器、导入和导出容器。下面我们逐一来看。

#### （1）创建并启动容器

容器十分轻量，用户可以随时创建和删除它。我们可以使用`docker create`命令来创建容器，例如：

复制代码

```
$ docker create -it --name=busybox busybox
Unable to find image 'busybox:latest' locally
latest: Pulling from library/busybox
61c5ed1cbdf8: Pull complete
Digest: sha256:4f47c01fa91355af2865ac10fef5bf6ec9c7f42ad2321377c21e844427972977
Status: Downloaded newer image for busybox:latest
2c2e919c2d6dad1f1712c65b3b8425ea656050bd5a0b4722f8b01526d5959ec6
$ docker ps -a| grep busybox
2c2e919c2d6d        busybox             "sh"                     34 seconds ago      Created                                         busybox
```

如果使用`docker create`命令创建的容器处于停止状态，我们可以使用`docker start`命令来启动它，如下所示。

复制代码

```
$ docker start busybox
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
d6f3d364fad3        busybox             "sh"                16 seconds ago      Up 8 seconds                            busybox
```

这时候我们可以看到容器已经处于启动状态了。
容器启动有两种方式：

1. 使用`docker start`命令基于已经创建好的容器直接启动 。
2. 使用`docker run`命令直接基于镜像新建一个容器并启动，相当于先执行`docker create`命令从镜像创建容器，然后再执行`docker start`命令启动容器。

使用`docker run`的命令如下:

复制代码

```
$ docker run -it --name=busybox busybox
```

当使用`docker run`创建并启动容器时，Docker 后台执行的流程为：

- Docker 会检查本地是否存在 busybox 镜像，如果镜像不存在则从 Docker Hub 拉取 busybox 镜像；
- 使用 busybox 镜像创建并启动一个容器；
- 分配文件系统，并且在镜像只读层外创建一个读写层；
- 从 Docker IP 池中分配一个 IP 给容器；
- 执行用户的启动命令运行镜像。

上述命令中， -t 参数的作用是分配一个伪终端，-i 参数则可以终端的 STDIN 打开，同时使用 -it 参数可以让我们进入交互模式。 在交互模式下，用户可以通过所创建的终端来输入命令，例如：

复制代码

```
$ ps aux
PID   USER     TIME  COMMAND
    1 root      0:00 sh
    6 root      0:00 ps aux
```

我们可以看到容器的 1 号进程为 sh 命令，在容器内部并不能看到主机上的进程信息，因为容器内部和主机是完全隔离的。同时由于 sh 是 1 号进程，意味着如果通过 exit 退出 sh，那么容器也会退出。所以对于容器来说，**杀死容器中的主进程，则容器也会被杀死。**

#### （2）终止容器

容器启动后，如果我们想停止运行中的容器，可以使用`docker stop`命令。命令格式为 docker stop [-t|--time[=10]]。该命令首先会向运行中的容器发送 SIGTERM 信号，如果容器内 1 号进程接受并能够处理 SIGTERM，则等待 1 号进程处理完毕后退出，如果等待一段时间后，容器仍然没有退出，则会发送 SIGKILL 强制终止容器。

复制代码

```
$ docker stop busybox
busybox
```

如果你想查看停止状态的容器信息，你可以使用 docker ps -a 命令。

复制代码

```
$ docker ps -a
CONTAINERID       IMAGE      COMMAND            CREATED             STATUS     PORTS         NAMES
28d477d3737a        busybox             "sh"                26 minutes ago      Exited (137) About a minute ago                       busybox
```

处于终止状态的容器也可以通过`docker start`命令来重新启动。

复制代码

```
$ docker start busybox
busybox
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
28d477d3737a        busybox             "sh"                30 minutes ago      Up 25 seconds                           busybox
```

此外，`docker restart`命令会将一个运行中的容器终止，并且重新启动它。

复制代码

```
$ docker restart busybox
busybox
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
28d477d3737a        busybox             "sh"                32 minutes ago      Up 3 seconds                            busybox
```

#### （3）进入容器

处于运行状态的容器可以通过`docker attach`、`docker exec`、`nsenter`等多种方式进入容器。

- **使用**`docker attach`命令**进入容器**

使用 docker attach ，进入我们上一步创建好的容器，如下所示。

复制代码

```
$ docker attach busybox
/ # ps aux
PID   USER     TIME  COMMAND
    1 root      0:00 sh
    7 root      0:00 ps aux
/ #
```

注意：当我们同时使用`docker attach`命令同时在多个终端运行时，所有的终端窗口将同步显示相同内容，当某个命令行窗口的命令阻塞时，其他命令行窗口同样也无法操作。
由于`docker attach`命令不够灵活，因此我们一般不会使用`docker attach`进入容器。下面我介绍一个更加灵活的进入容器的方式`docker exec`

- **使用 docker exec 命令进入容器**

Docker 从 1.3 版本开始，提供了一个更加方便地进入容器的命令`docker exec`，我们可以通过`docker exec -it CONTAINER`的方式进入到一个已经运行中的容器，如下所示。

复制代码

```
$ docker exec -it busybox sh
/ # ps aux
PID   USER     TIME  COMMAND
    1 root      0:00 sh
    7 root      0:00 sh
   12 root      0:00 ps aux
```

我们进入容器后，可以看到容器内有两个`sh`进程，这是因为以`exec`的方式进入容器，会单独启动一个 sh 进程，每个窗口都是独立且互不干扰的，也是使用最多的一种方式。

#### （4）删除容器

我们已经掌握了用 Docker 命令创建、启动和终止容器。那如何删除处于终止状态或者运行中的容器呢？删除容器命令的使用方式如下：`docker rm [OPTIONS] CONTAINER [CONTAINER...]`。

如果要删除一个停止状态的容器，可以使用`docker rm`命令删除。

复制代码

```
docker rm busybox
```

如果要删除正在运行中的容器，必须添加 -f (或 --force) 参数， Docker 会发送 SIGKILL 信号强制终止正在运行的容器。

复制代码

```
docker rm -f busybox
```

#### （5）导出导入容器

- **导出容器**

我们可以使用`docker export CONTAINER`命令导出一个容器到文件，不管此时该容器是否处于运行中的状态。导出容器前我们先进入容器，创建一个文件，过程如下。

首先进入容器创建文件

复制代码

```
docker exec -it busybox sh
cd /tmp && touch test
```

然后执行导出命令

复制代码

```
docker export busybox > busybox.tar
```

执行以上命令后会在当前文件夹下生成 busybox.tar 文件，我们可以将该文件拷贝到其他机器上，通过导入命令实现容器的迁移。

- **导入容器**

通过`docker export`命令导出的文件，可以使用`docker import`命令导入，执行完`docker import`后会变为本地镜像，最后再使用`docker run`命令启动该镜像，这样我们就实现了容器的迁移。

导入容器的命令格式为 docker import [OPTIONS] file|URL [REPOSITORY[:TAG]]。接下来我们一步步将上一步导出的镜像文件导入到其他机器的 Docker 中并启动它。

首先，使用`docker import`命令导入上一步导出的容器

复制代码

```
docker import busybox.tar busybox:test
```

此时，busybox.tar 被导入成为新的镜像，镜像名称为 busybox:test 。下面，我们使用`docker run`命令启动并进入容器，查看上一步创建的临时文件

复制代码

```
docker run -it busybox:test sh
/ # ls /tmp/
test
```

可以看到我们之前在 /tmp 目录下创建的 test 文件也被迁移过来了。这样我们就通过`docker export`和`docker import`命令配合实现了容器的迁移。

### 结语

到此，我相信你已经了解了容器的基本概念和组成，并已经熟练掌握了容器各个生命周期操作和管理。那容器与镜像的区别，你应该也很清楚了。镜像包含了容器运行所需要的文件系统结构和内容，是静态的只读文件，而容器则是在镜像的只读层上创建了可写层，并且容器中的进程属于运行状态，容器是真正的应用载体。

那你知道为什么容器的文件系统要设计成写时复制(如图 1 所示)，而不是每一个容器都单独拷贝一份镜像文件吗？思考后，可以把你的想法写在留言区。



# **05 | 仓库访问：怎样搭建属于你的私有仓库？**

在第三课时“镜像使用：Docker 环境下如何配置你的镜像？”里，我介绍了镜像的基本操作和镜像的原理，那么有了镜像，我们应该如何更好地存储和分发镜像呢？答案就是今天的主角——Docker 的镜像仓库。其实我们不仅可以使用公共镜像仓库存储和分发镜像，也可以自己搭建私有的镜像仓库，那在搭建之前，我们先回顾下仓库的基础知识。

### 仓库是什么？

仓库（Repository）是存储和分发 Docker 镜像的地方。镜像仓库类似于代码仓库，Docker Hub 的命名来自 GitHub，Github 是我们常用的代码存储和分发的地方。同样 Docker Hub 是用来提供 Docker 镜像存储和分发的地方。

有的同学可能经常分不清注册服务器（Registry）和仓库（Repository）的概念。在这里我可以解释下这两个概念的区别：注册服务器是存放仓库的实际服务器，而仓库则可以被理解为一个具体的项目或者目录；注册服务器可以包含很多个仓库，每个仓库又可以包含多个镜像。例如我的镜像地址为 docker.io/centos，docker.io 是注册服务器，centos 是仓库名。 它们之间的关系如图 1 所示。

![Lark20200911-162223.png](https://s0.lgstatic.com/i/image/M00/4D/C7/Ciqc1F9bM-uAI6MDAADk1noY7ic639.png)

按照类型，我们将镜像仓库分为公共镜像仓库和私有镜像仓库。

### 公共镜像仓库

公共镜像仓库一般是 Docker 官方或者其他第三方组织（阿里云，腾讯云，网易云等）提供的，允许所有人注册和使用的镜像仓库。

Docker Hub 是全球最大的镜像市场，目前已经有超过 10w 个容器镜像，这些容器镜像主要来自软件供应商、开源组织和社区。大部分的操作系统镜像和软件镜像都可以直接在 Docker Hub 下载并使用。

![Drawing 1.png](https://s0.lgstatic.com/i/image/M00/4D/C3/Ciqc1F9bL9yAYd_LAAJW9Q4Ue2w855.png)

图 2 Docker Hub 镜像

下面我以 Docker Hub 为例，教你如何使用公共镜像仓库分发和存储镜像。

#### 注册 Docker Hub 账号

我们首先访问[Docker Hub](https://hub.docker.com/)官网，点击注册按钮进入注册账号界面。

![Drawing 2.png](https://s0.lgstatic.com/i/image/M00/4D/CE/CgqCHl9bL-aABPLiAABcwVxClDY261.png)

图 3 注册 Docker Hub 账号

注册完成后，我们可以点击创建仓库，新建一个仓库用于推送镜像。

![Drawing 3.png](https://s0.lgstatic.com/i/image/M00/4D/C3/Ciqc1F9bL--AYVIKAADWoafHnho359.png)

图 4 创建仓库

这里我的账号为 lagoudocker，创建了一个名称为 busybox 的仓库，创建好仓库后我们就可以推送本地镜像到这个仓库里了。下面我通过一个实例来演示一下如何推送镜像到自己的仓库中。

首先我们使用以下命令拉取 busybox 镜像：

复制代码

```
$ docker pull busybox
Using default tag: latest
latest: Pulling from library/busybox
Digest: sha256:4f47c01fa91355af2865ac10fef5bf6ec9c7f42ad2321377c21e844427972977
Status: Image is up to date for busybox:latest
docker.io/library/busybox:latest
```

在推送镜像仓库前，我们需要使用`docker login`命令先登录一下镜像服务器，因为只有已经登录的用户才可以推送镜像到仓库。

复制代码

```
$ docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: lagoudocker
Password:
Login Succeeded
```

使用`docker login`命令登录镜像服务器，这时 Docker 会要求我们输入用户名和密码，输入我们刚才注册的账号和密码，看到`Login Succeeded`表示登录成功。登录成功后就可以推送镜像到自己创建的仓库了。

> `docker login`命令默认会请求 Docker Hub，如果你想登录第三方镜像仓库或者自建的镜像仓库，在`docker login`后面加上注册服务器即可。例如我们想登录访问阿里云镜像服务器，则使用`docker login registry.cn-beijing.aliyuncs.com`，输入阿里云镜像服务的用户名密码即可。

在本地镜像推送到自定义仓库前，我们需要先把镜像“重命名”一下，才能正确推送到自己创建的镜像仓库中，使用`docker tag`命令将镜像“重命名”：

复制代码

```
$ docker tag busybox lagoudocker/busybox
```

镜像“重命名”后使用`docker push`命令就可以推送镜像到自己创建的仓库中了。

复制代码

```
$ docker push lagoudocker/busybox
The push refers to repository [docker.io/lagoudocker/busybox]
514c3a3e64d4: Mounted from library/busybox
latest: digest: sha256:400ee2ed939df769d4681023810d2e4fb9479b8401d97003c710d0e20f7c49c6 size: 527
```

此时，`busybox`这个镜像就被推送到自定义的镜像仓库了。这里我们也可以新建其他的镜像仓库，然后把自己构建的镜像推送到仓库中。
有时候，出于安全或保密的需求，你可能想要搭建一个自己的镜像仓库，下面我带你一步一步构建一个私有的镜像仓库。

### 搭建私有仓库

#### 启动本地仓库

Docker 官方提供了开源的镜像仓库 [Distribution](https://github.com/docker/distribution)，并且镜像存放在 Docker Hub 的 [Registry](https://hub.docker.com/_/registry) 仓库下供我们下载。

我们可以使用以下命令启动一个本地镜像仓库：

复制代码

```
$ docker run -d -p 5000:5000 --name registry registry:2.7
Unable to find image 'registry:2.7' locally
2.7: Pulling from library/registry
cbdbe7a5bc2a: Pull complete
47112e65547d: Pull complete
46bcb632e506: Pull complete
c1cc712bcecd: Pull complete
3db6272dcbfa: Pull complete
Digest: sha256:8be26f81ffea54106bae012c6f349df70f4d5e7e2ec01b143c46e2c03b9e551d
Status: Downloaded newer image for registry:2.7
d7e449a8a93e71c9a7d99c67470bd7e7a723eee5ae97b3f7a2a8a1cf25982cc3
```

使用`docker ps`命令查看一下刚才启动的容器：

复制代码

```
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
d7e449a8a93e        registry:2.7        "/entrypoint.sh /etc…"   50 seconds ago      Up 49 seconds       0.0.0.0:5000->5000/tcp   registry
```

此时我们就拥有了一个私有镜像仓库，访问地址为`localhost`，端口号为 5000。

#### 推送镜像到本地仓库

我们依旧使用 busybox 镜像举例。首先我们使用`docker tag`命令把 busybox 镜像"重命名"为`localhost:5000/busybox`

复制代码

```
$ docker tag busybox localhost:5000/busybox
```

此时 Docker 为`busybox`镜像创建了一个别名`localhost:5000/busybox`，`localhost:5000`为主机名和端口，Docker 将会把镜像推送到这个地址。
使用`docker push`推送镜像到本地仓库：

复制代码

```
$ docker push localhost:5000/busybox
The push refers to repository [localhost:5000/busybox]
514c3a3e64d4: Layer already exists
latest: digest: sha256:400ee2ed939df769d4681023810d2e4fb9479b8401d97003c710d0e20f7c49c6 size: 527
```

这里可以看到，我们已经可以把`busybox`推送到了本地镜像仓库。

此时，我们验证一下从本地镜像仓库拉取镜像。首先，我们删除本地的`busybox`和`localhost:5000/busybox`镜像。

复制代码

```
$ docker rmi busybox localhost:5000/busybox
Untagged: busybox:latest
Untagged: busybox@sha256:4f47c01fa91355af2865ac10fef5bf6ec9c7f42ad2321377c21e844427972977
Untagged: localhost:5000/busybox:latest
Untagged: localhost:5000/busybox@sha256:400ee2ed939df769d4681023810d2e4fb9479b8401d97003c710d0e20f7c49c6
```

查看一下本地`busybox`镜像：

复制代码

```
$ docker image ls busybox
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
```

可以看到此时本地已经没有`busybox`这个镜像了。下面，我们从本地镜像仓库拉取`busybox`镜像：

复制代码

```
$ docker pull localhost:5000/busybox
Using default tag: latest
latest: Pulling from busybox
Digest: sha256:400ee2ed939df769d4681023810d2e4fb9479b8401d97003c710d0e20f7c49c6
Status: Downloaded newer image for localhost:5000/busybox:latest
localhost:5000/busybox:latest
```

然后再使用`docker image ls busybox`命令，这时可以看到我们已经成功从私有镜像仓库拉取`busybox`镜像到本地了

#### 持久化镜像存储

我们知道，容器是无状态的。上面私有仓库的启动方式可能会导致镜像丢失，因为我们并没有把仓库的数据信息持久化到主机磁盘上，这在生产环境中是无法接受的。下面我们使用以下命令将镜像持久化到主机目录：

复制代码

```
$ docker run -v /var/lib/registry/data:/var/lib/registry -d -p 5000:5000 --name registry registry:2.7
```

我们在上面启动`registry`的命令中加入了`-v /var/lib/registry/data:/var/lib/registry`，`-v`的含义是把 Docker 容器的某个目录或文件挂载到主机上，保证容器被重建后数据不丢失。`-v`参数冒号前面为主机目录，冒号后面为容器内目录。

> 事实上，registry 的持久化存储除了支持本地文件系统还支持很多种类型，例如 S3、Google Cloud Platform、Microsoft Azure Blob Storage Service 等多种存储类型。

到这里我们的镜像仓库虽然可以本地访问和拉取，但是如果你在另外一台机器上是无法通过 Docker 访问到这个镜像仓库的，因为 Docker 要求非`localhost`访问的镜像仓库必须使用 HTTPS，这时候就需要构建外部可访问的镜像仓库。

#### 构建外部可访问的镜像仓库

要构建一个支持 HTTPS 访问的安全镜像仓库，需要满足以下两个条件：

- 拥有一个合法的域名，并且可以正确解析到镜像服务器；
- 从证书颁发机构（CA）获取一个证书。

在准备好域名和证书后，就可以部署我们的镜像服务器了。这里我以`regisry.lagoudocker.io`这个域名为例。首先准备存放证书的目录`/var/lib/registry/certs`，然后把申请到的证书私钥和公钥分别放到该目录下。 假设我们申请到的证书文件分别为`regisry.lagoudocker.io.crt`和`regisry.lagoudocker.io.key`。

如果上一步启动的仓库容器还在运行，我们需要先停止并删除它。

复制代码

```
$ docker stop registry && docker rm registry
```

然后使用以下命令启动新的镜像仓库：

复制代码

```
$ docker run -d \
  --name registry \
  -v "/var/lib/registry/data:/var/lib/registry \
  -v "/var/lib/registry/certs:/certs \
  -e REGISTRY_HTTP_ADDR=0.0.0.0:443 \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/regisry.lagoudocker.io.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/regisry.lagoudocker.io.key \
  -p 443:443 \
  registry:2.7
```

这里，我们使用 -v 参数把镜像数据持久化在`/var/lib/registry/data`目录中，同时把主机上的证书文件挂载到了容器的 /certs 目录下，同时通过 -e 参数设置 HTTPS 相关的环境变量参数，最后让仓库在主机上监听 443 端口。

仓库启动后，我们就可以远程推送镜像了。

复制代码

```
$ docker tag busybox regisry.lagoudocker.io/busybox
$ docker push regisry.lagoudocker.io/busybox
```

#### 私有仓库进阶

Docker 官方开源的镜像仓库`Distribution`仅满足了镜像存储和管理的功能，用户权限管理相对较弱，并且没有管理界面。

如果你想要构建一个企业的镜像仓库，[Harbor](https://goharbor.io/) 是一个非常不错的解决方案。Harbor 是一个基于`Distribution`项目开发的一款企业级镜像管理软件，拥有 RBAC （基于角色的访问控制）、管理用户界面以及审计等非常完善的功能。目前已经从 CNCF 毕业，这代表它已经有了非常高的软件成熟度。

![Drawing 4.png](https://s0.lgstatic.com/i/image/M00/4D/CF/CgqCHl9bMHCAFgcMAABNmNOujV4312.png)

图 5 Harbor 官网

Harbor 的使命是成为 Kubernetes 信任的云原生镜像仓库。 Harbor 需要结合 Kubernetes 才能发挥其最大价值，因此，在这里我就不展开介绍 Harbor 了。如果你对 Harbor 构建企业级镜像仓库感兴趣，可以到它的[官网](https://goharbor.io/)了解更多。







# **06 | 最佳实践：如何在生产中编写最优 Dockerfile？**

在介绍 Dockerfile 最佳实践前，这里再强调一下，**生产实践中一定优先使用 Dockerfile 的方式构建镜像。** 因为使用 Dockerfile 构建镜像可以带来很多好处：

- 易于版本化管理，Dockerfile 本身是一个文本文件，方便存放在代码仓库做版本管理，可以很方便地找到各个版本之间的变更历史；
- 过程可追溯，Dockerfile 的每一行指令代表一个镜像层，根据 Dockerfile 的内容即可很明确地查看镜像的完整构建过程；
- 屏蔽构建环境异构，使用 Dockerfile 构建镜像无须考虑构建环境，基于相同 Dockerfile 无论在哪里运行，构建结果都一致。

虽然有这么多好处，但是如果你 Dockerfile 使用不当也会引发很多问题。比如镜像构建时间过长，甚至镜像构建失败；镜像层数过多，导致镜像文件过大。所以，这一课时我就教你如何在生产环境中编写最优的 Dockerfile。

在介绍 Dockerfile 最佳实践前，我们再聊一下我们平时书写 Dockerfile 应该尽量遵循的原则。

### Dockerfile 书写原则

遵循以下 Dockerfile 书写原则，不仅可以使得我们的 Dockerfile 简洁明了，让协作者清楚地了解镜像的完整构建流程，还可以帮助我们减少镜像的体积，加快镜像构建的速度和分发速度。

#### （1）单一职责

由于容器的本质是进程，一个容器代表一个进程，因此不同功能的应用应该尽量拆分为不同的容器，每个容器只负责单一业务进程。

#### （2）提供注释信息

Dockerfile 也是一种代码，我们应该保持良好的代码编写习惯，晦涩难懂的代码尽量添加注释，让协作者可以一目了然地知道每一行代码的作用，并且方便扩展和使用。

#### （3）保持容器最小化

应该避免安装无用的软件包，比如在一个 nginx 镜像中，我并不需要安装 vim 、gcc 等开发编译工具。这样不仅可以加快容器构建速度，而且可以避免镜像体积过大。

#### （4）合理选择基础镜像

容器的核心是应用，因此只要基础镜像能够满足应用的运行环境即可。例如一个`Java`类型的应用运行时只需要`JRE`，并不需要`JDK`，因此我们的基础镜像只需要安装`JRE`环境即可。

#### （5）使用 .dockerignore 文件

在使用`git`时，我们可以使用`.gitignore`文件忽略一些不需要做版本管理的文件。同理，使用`.dockerignore`文件允许我们在构建时，忽略一些不需要参与构建的文件，从而提升构建效率。`.dockerignore`的定义类似于`.gitignore`。

`.dockerignore`的本质是文本文件，Docker 构建时可以使用换行符来解析文件定义，每一行可以忽略一些文件或者文件夹。具体使用方式如下：

| 规则       | 含义                                                         |
| ---------- | ------------------------------------------------------------ |
| #          | # 开头的表示注释，# 后面所有内容将会被忽略                   |
| */tmp*     | 匹配当前目录下任何以 tmp 开头的文件或者文件夹                |
| *.md       | 匹配以 .md 为后缀的任意文件                                  |
| tem?       | 匹配以 tem 开头并且以任意字符结尾的文件，？代表任意一个字符  |
| !README.md | ! 表示排除忽略。 例如 .dockerignore 定义如下：  *.md !README.md  表示除了 README.md 文件外所有以 .md 结尾的文件。 |

#### （6）尽量使用构建缓存

Docker 构建过程中，每一条 Dockerfile 指令都会提交为一个镜像层，下一条指令都是基于上一条指令构建的。如果构建时发现要构建的镜像层的父镜像层已经存在，并且下一条命令使用了相同的指令，即可命中构建缓存。

Docker 构建时判断是否需要使用缓存的规则如下：

- 从当前构建层开始，比较所有的子镜像，检查所有的构建指令是否与当前完全一致，如果不一致，则不使用缓存；
- 一般情况下，只需要比较构建指令即可判断是否需要使用缓存，但是有些指令除外（例如`ADD`和`COPY`）；
- 对于`ADD`和`COPY`指令不仅要校验命令是否一致，还要为即将拷贝到容器的文件计算校验和（根据文件内容计算出的一个数值，如果两个文件计算的数值一致，表示两个文件内容一致 ），命令和校验和完全一致，才认为命中缓存。

因此，基于 Docker 构建时的缓存特性，我们可以把不轻易改变的指令放到 Dockerfile 前面（例如安装软件包），而可能经常发生改变的指令放在 Dockerfile 末尾（例如编译应用程序）。

例如，我们想要定义一些环境变量并且安装一些软件包，可以按照如下顺序编写 Dockerfile：

复制代码

```
FROM centos:7
# 设置环境变量指令放前面
ENV PATH /usr/local/bin:$PATH
# 安装软件指令放前面
RUN yum install -y make
# 把业务软件的配置,版本等经常变动的步骤放最后
...
```

按照上面原则编写的 Dockerfile 在构建镜像时，前面步骤命中缓存的概率会增加，可以大大缩短镜像构建时间。

#### （7）正确设置时区

我们从 Docker Hub 拉取的官方操作系统镜像大多数都是 UTC 时间（世界标准时间）。如果你想要在容器中使用中国区标准时间（东八区），请根据使用的操作系统修改相应的时区信息，下面我介绍几种常用操作系统的修改方式：

- **Ubuntu 和Debian 系统**

Ubuntu 和Debian 系统可以向 Dockerfile 中添加以下指令：

复制代码

```
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
RUN echo "Asia/Shanghai" >> /etc/timezone
```

- **CentOS系统**

CentOS 系统则向 Dockerfile 中添加以下指令：

复制代码

```
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

#### （8）使用国内软件源加快镜像构建速度

由于我们常用的官方操作系统镜像基本都是国外的，软件服务器大部分也在国外，所以我们构建镜像的时候想要安装一些软件包可能会非常慢。

这里我以 CentOS 7 为例，介绍一下如何使用 163 软件源（国内有很多大厂，例如阿里、腾讯、网易等公司都免费提供的软件加速源）加快镜像构建。

首先在容器构建目录创建文件 CentOS7-Base-163.repo，文件内容如下：

复制代码

```
# CentOS-Base.repo
#
# The mirror system uses the connecting IP address of the client and the
# update status of each mirror to pick mirrors that are updated to and
# geographically close to the client.  You should use this for CentOS updates
# unless you are manually picking other mirrors.
#
# If the mirrorlist= does not work for you, as a fall back you can try the 
# remarked out baseurl= line instead.
#
#
[base]
name=CentOS-$releasever - Base - 163.com
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os
baseurl=http://mirrors.163.com/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7
#released updates
[updates]
name=CentOS-$releasever - Updates - 163.com
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates
baseurl=http://mirrors.163.com/centos/$releasever/updates/$basearch/
gpgcheck=1
gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7
#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras - 163.com
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras
baseurl=http://mirrors.163.com/centos/$releasever/extras/$basearch/
gpgcheck=1
gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7
#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus - 163.com
baseurl=http://mirrors.163.com/centos/$releasever/centosplus/$basearch/
gpgcheck=1
enabled=0
gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7
```

然后在 Dockerfile 中添加如下指令：

复制代码

```
COPY CentOS7-Base-163.repo /etc/yum.repos.d/CentOS7-Base.repo
```

执行完上述步骤后，再使用`yum install`命令安装软件时就会默认从 163 获取软件包，这样可以大大提升构建速度。

#### （9）最小化镜像层数

在构建镜像时尽可能地减少 Dockerfile 指令行数。例如我们要在 CentOS 系统中安装`make`和`net-tools`两个软件包，应该在 Dockerfile 中使用以下指令：

复制代码

```
RUN yum install -y make net-tools
```

而不应该写成这样：

复制代码

```
RUN yum install -y make
RUN yum install -y net-tools
```

了解完 Dockerfile 的书写原则后，我们再来具体了解下这些原则落实到具体的 Dockerfile 指令应该如何书写。

### Dockerfile 指令书写建议

下面是我们常用的一些指令，这些指令对于刚接触 Docker 的人来说会非常容易出错，下面我对这些指令的书写建议详细讲解一下。

#### （1）RUN

`RUN`指令在构建时将会生成一个新的镜像层并且执行`RUN`指令后面的内容。

使用`RUN`指令时应该尽量遵循以下原则：

- 当`RUN`指令后面跟的内容比较复杂时，建议使用反斜杠（\） 结尾并且换行；
- `RUN`指令后面的内容尽量按照字母顺序排序，提高可读性。

例如，我想在官方的 CentOS 镜像下安装一些软件，一个建议的 Dockerfile 指令如下：

复制代码

```
FROM centos:7
RUN yum install -y automake \
                   curl \
                   python \
                   vim
```

#### （2）CMD 和 ENTRYPOINT

`CMD`和`ENTRYPOINT`指令都是容器运行的命令入口，这两个指令使用中有很多相似的地方，但是也有一些区别。

这两个指令的相同之处，`CMD`和`ENTRYPOINT`的基本使用格式分为两种。

- 第一种为`CMD`/`ENTRYPOINT`["command" , "param"]。这种格式是使用 Linux 的`exec`实现的， 一般称为`exec`模式，这种书写格式为`CMD`/`ENTRYPOINT`后面跟 json 数组，也是Docker 推荐的使用格式。
- 另外一种格式为`CMD`/`ENTRYPOINT`command param ，这种格式是基于 shell 实现的， 通常称为`shell`模式。当使用`shell`模式时，Docker 会以 /bin/sh -c command 的方式执行命令。

> 使用 exec 模式启动容器时，容器的 1 号进程就是 CMD/ENTRYPOINT 中指定的命令，而使用 shell 模式启动容器时相当于我们把启动命令放在了 shell 进程中执行，等效于执行 /bin/sh -c "task command" 命令。因此 shell 模式启动的进程在容器中实际上并不是 1 号进程。

这两个指令的区别：

- Dockerfile 中如果使用了`ENTRYPOINT`指令，启动 Docker 容器时需要使用 --entrypoint 参数才能覆盖 Dockerfile 中的`ENTRYPOINT`指令 ，而使用`CMD`设置的命令则可以被`docker run`后面的参数直接覆盖。
- `ENTRYPOINT`指令可以结合`CMD`指令使用，也可以单独使用，而`CMD`指令只能单独使用。

看到这里你也许会问，我什么时候应该使用`ENTRYPOINT`,什么时候使用`CMD`呢？

如果你希望你的镜像足够灵活，推荐使用`CMD`指令。如果你的镜像只执行单一的具体程序，并且不希望用户在执行`docker run`时覆盖默认程序，建议使用`ENTRYPOINT`。

最后再强调一下，无论使用`CMD`还是`ENTRYPOINT`，都尽量使用`exec`模式。

#### （3）ADD 和 COPY

`ADD`和`COPY`指令功能类似，都是从外部往容器内添加文件。但是`COPY`指令只支持基本的文件和文件夹拷贝功能，`ADD`则支持更多文件来源类型，比如自动提取 tar 包，并且可以支持源文件为 URL 格式。

那么在日常应用中，我们应该使用哪个命令向容器里添加文件呢？你可能在想，既然`ADD`指令支持的功能更多，当然应该使用`ADD`指令了。然而事实恰恰相反，我更推荐你使用`COPY`指令，因为`COPY`指令更加透明，仅支持本地文件向容器拷贝，而且使用`COPY`指令可以更好地利用构建缓存，有效减小镜像体积。

当你想要使用`ADD`向容器中添加 URL 文件时，请尽量考虑使用其他方式替代。例如你想要在容器中安装 memtester（一种内存压测工具），你应该避免使用以下格式：

复制代码

```
ADD http://pyropus.ca/software/memtester/old-versions/memtester-4.3.0.tar.gz /tmp/
RUN tar -xvf /tmp/memtester-4.3.0.tar.gz -C /tmp
RUN make -C /tmp/memtester-4.3.0 && make -C /tmp/memtester-4.3.0 install
```

下面是推荐写法：

复制代码

```
RUN wget -O /tmp/memtester-4.3.0.tar.gz http://pyropus.ca/software/memtester/old-versions/memtester-4.3.0.tar.gz \
&& tar -xvf /tmp/memtester-4.3.0.tar.gz -C /tmp \
&& make -C /tmp/memtester-4.3.0 && make -C /tmp/memtester-4.3.0 install
```

#### （4）WORKDIR

为了使构建过程更加清晰明了，推荐使用 WORKDIR 来指定容器的工作路径，应该尽量避免使用 RUN cd /work/path && do some work 这样的指令。

最后给出几个常用软件的官方 Dockerfile 示例链接，希望可以对你有所帮助。

- [Go](https://github.com/docker-library/golang/blob/4d68c4dd8b51f83ce4fdce0f62484fdc1315bfa8/1.15/buster/Dockerfile)
- [Nginx](https://github.com/nginxinc/docker-nginx/blob/9774b522d4661effea57a1fbf64c883e699ac3ec/mainline/buster/Dockerfile)
- [Hy](https://github.com/hylang/docker-hylang/blob/f9c873b7f71f466e5af5ea666ed0f8f42835c688/dockerfiles-generated/Dockerfile.python3.8-buster)

### 结语

好了，到此为止，相信你已经对 Dockerfile 的书写原则和一些重要指令有了较深的认识。

当你需要编写编译型语言（例如 Golang、Java）的 Dockerfile 时，如何分离编译环境和运行环境，使得镜像体积尽可能小呢？思考后，可以把你的想法写在留言区。

# **07 | Docker 安全：基于内核的弱隔离系统如何保障安全性？**



在第 01 课时“Docker 安装：入门案例带你了解容器技术原理”中，我有介绍到 Docker 是基于 Linux 内核的 Namespace 技术实现资源隔离的，所有的容器都共享主机的内核。其实这与以虚拟机为代表的云计算时代还是有很多区别的，比如虚拟机有着更好的隔离性和安全性，而容器的隔离性和安全性则相对较弱。

在讨论容器的安全性之前，我们先了解下容器与虚拟机的区别，这样可以帮助我们更好地了解容器的安全隐患以及如何加固容器安全。

### Docker 与虚拟机区别

![WechatIMG1632.jpeg](https://s0.lgstatic.com/i/image/M00/56/B7/Ciqc1F9sDDSAQhNcAAD8rL1NLXc02.jpeg)

从图 1 可以看出，虚拟机是通过管理系统(Hypervisor)模拟出 CPU、内存、网络等硬件，然后在这些模拟的硬件上创建客户内核和操作系统。这样做的好处就是虚拟机有自己的内核和操作系统，并且硬件都是通过虚拟机管理系统模拟出来的，用户程序无法直接使用到主机的操作系统和硬件资源，因此虚拟机也对隔离性和安全性有着更好的保证。

而 Docker 容器则是通过 Linux 内核的 Namespace 技术实现了文件系统、进程、设备以及网络的隔离，然后再通过 Cgroups 对 CPU、 内存等资源进行限制，最终实现了容器之间相互不受影响，由于容器的隔离性仅仅依靠内核来提供，因此容器的隔离性也远弱于虚拟机。

你可能会问，既然虚拟机安全性这么好，为什么我们还要用容器呢？这是因为容器与虚拟机相比，容器的性能损耗非常小，并且镜像也非常小，而且在业务快速开发和迭代的今天，容器秒级的启动等特性也非常匹配业务快速迭代的业务场景。

既然我们要利用容器的优点，那有没有什么办法可以尽量弥补容器弱隔离的安全性缺点呢？要了解如何解决容器的安全问题，我们首先需要了解下容器目前存在的安全问题。

### Docker 容器的安全问题

#### (1) Docker 自身安全

Docker 作为一款容器引擎，本身也会存在一些安全漏洞，CVE 目前已经记录了多项与 Docker 相关的安全漏洞，主要有权限提升、信息泄露等几类安全问题。具体 Docker 官方记录的安全问题可以参考[这里](https://docs.docker.com/engine/security/non-events/)。

> CVE 的维基百科定义：CVE 是公共漏洞和暴露（英语：CVE, Common Vulnerabilities and Exposures）又称常见漏洞与披露，是一个与信息安全有关的数据库，收集各种信息安全弱点及漏洞并给予编号以便于公众查阅。此数据库现由美国非营利组织 MITRE 所属的 National Cybersecurity FFRDC 所营运维护 。

#### (2) 镜像安全

由于 Docker 容器是基于镜像创建并启动，因此镜像的安全直接影响到容器的安全。具体影响镜像安全的总结如下。

- 镜像软件存在安全漏洞：由于容器需要安装基础的软件包，如果软件包存在漏洞，则可能会被不法分子利用并且侵入容器，影响其他容器或主机安全。
- 仓库漏洞：无论是 Docker 官方的镜像仓库还是我们私有的镜像仓库，都有可能被攻击，然后篡改镜像，当我们使用镜像时，就可能成为攻击者的目标对象。
- 用户程序漏洞：用户自己构建的软件包可能存在漏洞或者被植入恶意脚本，这样会导致运行时提权影响其他容器或主机安全。

#### (3) Linux 内核隔离性不够

尽管目前 Namespace 已经提供了非常多的资源隔离类型，但是仍有部分关键内容没有被完全隔离，其中包括一些系统的关键性目录（如 /sys、/proc 等），这些关键性的目录可能会泄露主机上一些关键性的信息，让攻击者利用这些信息对整个主机甚至云计算中心发起攻击。

而且仅仅依靠 Namespace 的隔离是远远不够的，因为一旦内核的 Namespace 被突破，使用者就有可能直接提权获取到主机的超级权限，从而影响主机安全。

#### (4) 所有容器共享主机内核

由于同一宿主机上所有容器共享主机内核，所以攻击者可以利用一些特殊手段导致内核崩溃，进而导致主机宕机影响主机上其他服务。

既然容器有这么多安全上的问题，那么我们应该如何做才能够既享受到容器的便利性同时也可以保障容器安全呢？下面我带你来逐步了解下如何解决容器的安全问题。

### 如何解决容器的安全问题？

#### (1) Docker 自身安全性改进

事实上，Docker 从 2013 年诞生到现在，在安全性上面已经做了非常多的努力。目前 Docker 在默认配置和默认行为下是足够安全的。

Docker 自身是基于 Linux 的多种 Namespace 实现的，其中有一个很重要的 Namespace 叫作 User Namespace，User Namespace 主要是用来做容器内用户和主机的用户隔离的。在过去容器里的 root 用户就是主机上的 root 用户，如果容器受到攻击，或者容器本身含有恶意程序，在容器内就可以直接获取到主机 root 权限。Docker 从 1.10 版本开始，使用 User Namespace 做用户隔离，实现了容器中的 root 用户映射到主机上的非 root 用户，从而大大减轻了容器被突破的风险。

因此，我们尽可能地使用 Docker 最新版本就可以得到更好的安全保障。

#### (2) 保障镜像安全

为保障镜像安全，我们可以在私有镜像仓库安装镜像安全扫描组件，对上传的镜像进行检查，通过与 CVE 数据库对比，一旦发现有漏洞的镜像及时通知用户或阻止非安全镜像继续构建和分发。同时为了确保我们使用的镜像足够安全，在拉取镜像时，要确保只从受信任的镜像仓库拉取，并且与镜像仓库通信一定要使用 HTTPS 协议。

#### (3) 加强内核安全和管理

由于仅仅依赖内核的隔离可能会引发安全问题，因此我们对于内核的安全应该更加重视。可以从以下几个方面进行加强。

**宿主机及时升级内核漏洞**

宿主机内核应该尽量安装最新补丁，因为更新的内核补丁往往有着更好的安全性和稳定性。

**使用 Capabilities 划分权限**

Capabilities 是 Linux 内核的概念，Linux 将系统权限分为了多个 Capabilities，它们都可以单独地开启或关闭，Capabilities 实现了系统更细粒度的访问控制。

容器和虚拟机在权限控制上还是有一些区别的，在虚拟机内我们可以赋予用户所有的权限，例如设置 cron 定时任务、操作内核模块、配置网络等权限。而容器则需要针对每一项 Capabilities 更细粒度的去控制权限，例如：

- cron 定时任务可以在容器内运行，设置定时任务的权限也仅限于容器内部；
- 由于容器是共享主机内核的，因此在容器内部一般不允许直接操作主机内核；
- 容器的网络管理在容器外部，这就意味着一般情况下，我们在容器内部是不需要执行`ifconfig`、`route`等命令的 。

由于容器可以按照需求逐项添加 Capabilities 权限，因此在大多数情况下，容器并不需要主机的 root 权限，Docker 默认情况下也是不开启额外特权的。

最后，在执行`docker run`命令启动容器时，如非特殊可控情况，--privileged 参数不允许设置为 true，其他特殊权限可以使用 --cap-add 参数，根据使用场景适当添加相应的权限。

**使用安全加固组件**

Linux 的 SELinux、AppArmor、GRSecurity 组件都是 Docker 官方推荐的安全加固组件。下面我对这三个组件做简单介绍。

- SELinux (Secure Enhanced Linux): 是 Linux 的一个内核安全模块，提供了安全访问的策略机制，通过设置 SELinux 策略可以实现某些进程允许访问某些文件。
- AppArmor: 类似于 SELinux，也是一个 Linux 的内核安全模块，普通的访问控制仅能控制到用户的访问权限，而 AppArmor 可以控制到用户程序的访问权限。
- GRSecurity: 是一个对内核的安全扩展，可通过智能访问控制，提供内存破坏防御，文件系统增强等多种防御形式。

这三个组件可以限制一个容器对主机的内核或其他资源的访问控制。目前，容器报告的一些安全漏洞中，很多都是通过对内核进行加强访问和隔离来实现的。

**资源限制**

在生产环境中，建议每个容器都添加相应的资源限制。下面给出一些执行`docker run`命令启动容器时可以传递的资源限制参数：

复制代码

```
  --cpus                          限制 CPU 配额
  -m, --memory                    限制内存配额
  --pids-limit                    限制容器的 PID 个数
```

例如我想要启动一个 1 核 2G 的容器，并且限制在容器内最多只能创建 1000 个 PID，启动命令如下：

复制代码

```
$ docker run -it --cpus=1 -m=2048m --pids-limit=1000 busybox sh
```

推荐在生产环境中限制 CPU、内存、PID 等资源，这样即便应用程序有漏洞，也不会导致主机的资源完全耗尽，最大限度降低安全风险。

#### (4) 使用安全容器

容器有着轻便快速启动的优点，虚拟机有着安全隔离的优点，有没有一种技术可以兼顾两者的优点，做到既轻量又安全呢？

答案是有，那就是安全容器。安全容器是相较于普通容器的，安全容器与普通容器的主要区别在于，安全容器中的每个容器都运行在一个单独的微型虚拟机中，拥有独立的操作系统和内核，并且有虚拟化层的安全隔离。

安全容器目前推荐的技术方案是 [Kata Containers](https://github.com/kata-containers)，Kata Container 并不包含一个完整的操作系统，只有一个精简版的 Guest Kernel 运行着容器本身的应用，并且通过减少不必要的内存，尽量共享可以共享的内存来进一步减少内存的开销。另外，Kata Container 实现了 OCI 规范，可以直接使用 Docker 的镜像启动 Kata 容器，具有开销更小、秒级启动、安全隔离等许多优点。

### 结语

容器技术带来的技术革新是空前的，但是随之而来的容器安全问题也是我们必须要足够重视的。本课时解决 Docker 安全问题的精华我帮你总结如下：

![Lark20200918-170906.png](https://s0.lgstatic.com/i/image/M00/51/28/Ciqc1F9keVSAHuDTAADaB11MKbU710.png)

到此，相信你已经了解了 Docker 与虚拟机的本质区别，也知道了容器目前存在的一些安全隐患以及如何在生产环境中尽量避免这些安全隐患。

目前除了 Kata Container 外，你还知道其他的安全容器解决方案吗？知道的同学，可以把你的想法写在留言区。







# **08 | 容器监控：容器监控原理及 cAdvisor 的安装与使用**

生产环境中监控容器的运行状况十分重要，通过监控我们可以随时掌握容器的运行状态，做到线上隐患和问题早发现，早解决。所以今天我就和你分享关于容器监控的知识（原理及工具 cAdvisor）。

虽然传统的物理机和虚拟机监控已经有了比较成熟的监控方案，但是容器的监控面临着更大的挑战，因为容器的行为和本质与传统的虚拟机是不一样的，总的来说，容器具有以下特性：

- 容器是短期存活的，并且可以动态调度；
- 容器的本质是进程，而不是一个完整操作系统；
- 由于容器非常轻量，容器的创建和销毁也会比传统虚拟机更加频繁。

Docker 容器的监控方案有很多，除了 Docker 自带的`docker stats`命令，还有很多开源的解决方案，例如 sysdig、cAdvisor、Prometheus 等，都是非常优秀的监控工具。

下面我们首先来看下，不借助任何外部工具，如何用 Docker 自带的`docker stats`命令实现容器的监控。

### 使用 docker stats 命令

使用 Docker 自带的`docker stats`命令可以很方便地看到主机上所有容器的 CPU、内存、网络 IO、磁盘 IO、PID 等资源的使用情况。下面我们可以具体操作看看。

首先在主机上使用以下命令启动一个资源限制为 1 核 2G 的 nginx 容器：

复制代码

```
$ docker run --cpus=1 -m=2g --name=nginx  -d nginx
```

容器启动后，可以使用`docker stats`命令查看容器的资源使用状态:

复制代码

```
$ docker stats nginx
```

通过`docker stats`命令可以看到容器的运行状态如下：

复制代码

```
CONTAINER           CPU %               MEM USAGE / LIMIT   MEM %               NET I/O             BLOCK I/O           PIDS
f742a467b6d8        0.00%               1.387 MiB / 2 GiB   0.07%               656 B / 656 B       0 B / 9.22 kB       2
```

从容器的运行状态可以看出，`docker stats`命令确实可以获取并显示 Docker 容器运行状态。但是它的缺点也很明显，因为它只能获取本机数据，无法查看历史监控数据，没有可视化展示面板。

因此，生产环境中我们通常使用另一种容器监控解决方案 cAdvisor。

### cAdvisor

cAdvisor 是谷歌开源的一款通用的容器监控解决方案。cAdvisor 不仅可以采集机器上所有运行的容器信息，还提供了基础的查询界面和 HTTP 接口，更方便与外部系统结合。所以，cAdvisor很快成了容器指标监控最常用组件，并且 Kubernetes 也集成了 cAdvisor 作为容器监控指标的默认工具。

#### cAdvisor 的安装与使用

下面我们以 cAdvisor 0.37.0 版本为例，演示一下 cAdvisor 的安装与使用。

cAdvisor 官方提供了 Docker 镜像，我们只需要拉取镜像并且启动镜像即可。

> 由于 cAdvisor 镜像存放在谷歌的 gcr.io 镜像仓库中，国内无法访问到。这里我把打好的镜像放在了 Docker Hub。你可以使用 docker pull lagoudocker/cadvisor:v0.37.0 命令从 Docker Hub 拉取。

首先使用以下命令启动 cAdvisor：

复制代码

```
$ docker run \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:ro \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --volume=/dev/disk/:/dev/disk:ro \
  --publish=8080:8080 \
  --detach=true \
  --name=cadvisor \
  --privileged \
  --device=/dev/kmsg \
  lagoudocker/cadvisor:v0.37.0
```

此时，cAdvisor 已经成功启动，我们可以通过访问 [http://localhost:8080](http://localhost:8080/) 访问到 cAdvisor 的 Web 界面。

![Drawing 0.png](https://s0.lgstatic.com/i/image/M00/56/18/Ciqc1F9rCXSAQEwLAADKlh0at8o307.png)

图1 cAdvisor 首页

cAdvisor 不仅可以监控容器的资源使用情况，还可以监控主机的资源使用情况。下面我们就先看下它是如何查看主机资源使用情况的。

#### 使用 cAdvisor 查看主机监控

访问 http://localhost:8080/containers/ 地址，在首页可以看到主机的资源使用情况，包含 CPU、内存、文件系统、网络等资源，如下图所示。

![Drawing 1.png](https://s0.lgstatic.com/i/image/M00/56/23/CgqCHl9rCX2ANrtaAADIGkeKKPc100.png)

图2 主机 CPU 使用情况

#### 使用 cAdvisor 查看容器监控

如果你想要查看主机上运行的容器资源使用情况，可以访问 http://localhost:8080/docker/，这个页面会列出 Docker 的基本信息和运行的容器情况，如下图所示。

![Drawing 2.png](https://s0.lgstatic.com/i/image/M00/56/18/Ciqc1F9rCZyAN8hYAAGAOL1FGcg401.png)

图3 Docker 容器

在上图中的 Subcontainers 下会列出当前主机上运行的所有容器，点击其中一个容器即可查看该容器的详细运行状态，如下图所示。

![Drawing 3.png](https://s0.lgstatic.com/i/image/M00/56/23/CgqCHl9rCaWAVSLVAAGGy2lTMqY130.png)

图4 容器监控状态

总体来说，使用 cAdvisor 监控容器具有以下特点：

- 可以同时采集物理机和容器的状态；
- 可以展示监控历史数据。

了解 Docker 的监控工具，你是否想问，这些监控数据是怎么来的呢？下面我就带你了解一下容器监控的原理。

### 监控原理

我们知道 Docker 是基于 Namespace、Cgroups 和联合文件系统实现的。其中 Cgroups 不仅可以用于容器资源的限制，还可以提供容器的资源使用率。无论何种监控方案的实现，底层数据都来源于 Cgroups。

Cgroups 的工作目录为`/sys/fs/cgroup`，`/sys/fs/cgroup`目录下包含了 Cgroups 的所有内容。Cgroups包含很多子系统，可以用来对不同的资源进行限制。例如对CPU、内存、PID、磁盘 IO等资源进行限制和监控。

为了更详细的了解 Cgroups 的子系统，我们通过 ls -l 命令查看`/sys/fs/cgroup`文件夹，可以看到很多目录：

复制代码

```
$ sudo ls -l /sys/fs/cgroup/
total 0
dr-xr-xr-x 5 root root  0 Jul  9 19:32 blkio
lrwxrwxrwx 1 root root 11 Jul  9 19:32 cpu -> cpu,cpuacct
dr-xr-xr-x 5 root root  0 Jul  9 19:32 cpu,cpuacct
lrwxrwxrwx 1 root root 11 Jul  9 19:32 cpuacct -> cpu,cpuacct
dr-xr-xr-x 3 root root  0 Jul  9 19:32 cpuset
dr-xr-xr-x 5 root root  0 Jul  9 19:32 devices
dr-xr-xr-x 3 root root  0 Jul  9 19:32 freezer
dr-xr-xr-x 3 root root  0 Jul  9 19:32 hugetlb
dr-xr-xr-x 5 root root  0 Jul  9 19:32 memory
lrwxrwxrwx 1 root root 16 Jul  9 19:32 net_cls -> net_cls,net_prio
dr-xr-xr-x 3 root root  0 Jul  9 19:32 net_cls,net_prio
lrwxrwxrwx 1 root root 16 Jul  9 19:32 net_prio -> net_cls,net_prio
dr-xr-xr-x 3 root root  0 Jul  9 19:32 perf_event
dr-xr-xr-x 5 root root  0 Jul  9 19:32 pids
dr-xr-xr-x 5 root root  0 Jul  9 19:32 systemd
```

这些目录代表了 Cgroups 的子系统，Docker 会在每一个 Cgroups 子系统下创建 docker 文件夹。这里如果你对 Cgroups 子系统不了解的话，不要着急，后续我会在第 10 课时对 Cgroups 子系统做详细讲解，这里你只需要明白容器监控数据来源于 Cgroups 即可。

#### 监控系统是如何获取容器的内存限制的？

下面我们以 memory 子系统（memory 子系统是Cgroups 众多子系统的一个，主要用来限制内存使用，Cgroups 会在第十课时详细讲解）为例，讲解一下监控组件是如何获取到容器的资源限制和使用状态的（即容器的内存限制）。

我们首先在主机上使用以下命令启动一个资源限制为 1 核 2G 的 nginx 容器：

复制代码

```
$ docker run --name=nginx --cpus=1 -m=2g --name=nginx  -d nginx
## 这里输出的是容器 ID
51041a74070e9260e82876974762b8c61c5ed0a51832d74fba6711175f89ede1
```

> 注意：如果你已经创建过名称为 nginx 的容器，请先使用 docker rm -f nginx 命令删除已经存在的 nginx 容器。

容器启动后，我们通过命令行的输出可以得到容器的 ID，同时 Docker 会在`/sys/fs/cgroup/memory/docker`目录下以容器 ID 为名称创建对应的文件夹。

下面我们查看一下`/sys/fs/cgroup/memory/docker`目录下的文件：

复制代码

```
$ sudo ls -l /sys/fs/cgroup/memory/docker
total 0
drwxr-xr-x 2 root root 0 Sep  2 15:12 51041a74070e9260e82876974762b8c61c5ed0a51832d74fba6711175f89ede1
-rw-r--r-- 1 root root 0 Sep  2 14:57 cgroup.clone_children
--w--w--w- 1 root root 0 Sep  2 14:57 cgroup.event_control
-rw-r--r-- 1 root root 0 Sep  2 14:57 cgroup.procs
-rw-r--r-- 1 root root 0 Sep  2 14:57 memory.failcnt
--w------- 1 root root 0 Sep  2 14:57 memory.force_empty
-rw-r--r-- 1 root root 0 Sep  2 14:57 memory.kmem.failcnt
-rw-r--r-- 1 root root 0 Sep  2 14:57 memory.kmem.limit_in_bytes
-rw-r--r-- 1 root root 0 Sep  2 14:57 memory.kmem.max_usage_in_bytes
-r--r--r-- 1 root root 0 Sep  2 14:57 memory.kmem.slabinfo
-rw-r--r-- 1 root root 0 Sep  2 14:57 memory.kmem.tcp.failcnt
-rw-r--r-- 1 root root 0 Sep  2 14:57 memory.kmem.tcp.limit_in_bytes
-rw-r--r-- 1 root root 0 Sep  2 14:57 memory.kmem.tcp.max_usage_in_bytes
-r--r--r-- 1 root root 0 Sep  2 14:57 memory.kmem.tcp.usage_in_bytes
-r--r--r-- 1 root root 0 Sep  2 14:57 memory.kmem.usage_in_bytes
-rw-r--r-- 1 root root 0 Sep  2 14:57 memory.limit_in_bytes
-rw-r--r-- 1 root root 0 Sep  2 14:57 memory.max_usage_in_bytes
-rw-r--r-- 1 root root 0 Sep  2 14:57 memory.memsw.failcnt
-rw-r--r-- 1 root root 0 Sep  2 14:57 memory.memsw.limit_in_bytes
-rw-r--r-- 1 root root 0 Sep  2 14:57 memory.memsw.max_usage_in_bytes
-r--r--r-- 1 root root 0 Sep  2 14:57 memory.memsw.usage_in_bytes
-rw-r--r-- 1 root root 0 Sep  2 14:57 memory.move_charge_at_immigrate
-r--r--r-- 1 root root 0 Sep  2 14:57 memory.numa_stat
-rw-r--r-- 1 root root 0 Sep  2 14:57 memory.oom_control
---------- 1 root root 0 Sep  2 14:57 memory.pressure_level
-rw-r--r-- 1 root root 0 Sep  2 14:57 memory.soft_limit_in_bytes
-r--r--r-- 1 root root 0 Sep  2 14:57 memory.stat
-rw-r--r-- 1 root root 0 Sep  2 14:57 memory.swappiness
-r--r--r-- 1 root root 0 Sep  2 14:57 memory.usage_in_bytes
-rw-r--r-- 1 root root 0 Sep  2 14:57 memory.use_hierarchy
-rw-r--r-- 1 root root 0 Sep  2 14:57 notify_on_release
-rw-r--r-- 1 root root 0 Sep  2 14:57 tasks
```

可以看到 Docker 已经创建了以容器 ID 为名称的目录，我们再使用 ls 命令查看一下该目录的内容：

复制代码

```
$ sudo ls -l /sys/fs/cgroup/memory/docker/51041a74070e9260e82876974762b8c61c5ed0a51832d74fba6711175f89ede1
total 0
-rw-r--r-- 1 root root 0 Sep  2 15:21 cgroup.clone_children
--w--w--w- 1 root root 0 Sep  2 15:13 cgroup.event_control
-rw-r--r-- 1 root root 0 Sep  2 15:12 cgroup.procs
-rw-r--r-- 1 root root 0 Sep  2 15:12 memory.failcnt
--w------- 1 root root 0 Sep  2 15:21 memory.force_empty
-rw-r--r-- 1 root root 0 Sep  2 15:21 memory.kmem.failcnt
-rw-r--r-- 1 root root 0 Sep  2 15:12 memory.kmem.limit_in_bytes
-rw-r--r-- 1 root root 0 Sep  2 15:21 memory.kmem.max_usage_in_bytes
-r--r--r-- 1 root root 0 Sep  2 15:21 memory.kmem.slabinfo
-rw-r--r-- 1 root root 0 Sep  2 15:21 memory.kmem.tcp.failcnt
-rw-r--r-- 1 root root 0 Sep  2 15:21 memory.kmem.tcp.limit_in_bytes
-rw-r--r-- 1 root root 0 Sep  2 15:21 memory.kmem.tcp.max_usage_in_bytes
-r--r--r-- 1 root root 0 Sep  2 15:21 memory.kmem.tcp.usage_in_bytes
-r--r--r-- 1 root root 0 Sep  2 15:21 memory.kmem.usage_in_bytes
-rw-r--r-- 1 root root 0 Sep  2 15:12 memory.limit_in_bytes
-rw-r--r-- 1 root root 0 Sep  2 15:12 memory.max_usage_in_bytes
-rw-r--r-- 1 root root 0 Sep  2 15:21 memory.memsw.failcnt
-rw-r--r-- 1 root root 0 Sep  2 15:12 memory.memsw.limit_in_bytes
-rw-r--r-- 1 root root 0 Sep  2 15:21 memory.memsw.max_usage_in_bytes
-r--r--r-- 1 root root 0 Sep  2 15:21 memory.memsw.usage_in_bytes
-rw-r--r-- 1 root root 0 Sep  2 15:21 memory.move_charge_at_immigrate
-r--r--r-- 1 root root 0 Sep  2 15:21 memory.numa_stat
-rw-r--r-- 1 root root 0 Sep  2 15:13 memory.oom_control
---------- 1 root root 0 Sep  2 15:21 memory.pressure_level
-rw-r--r-- 1 root root 0 Sep  2 15:21 memory.soft_limit_in_bytes
-r--r--r-- 1 root root 0 Sep  2 15:21 memory.stat
-rw-r--r-- 1 root root 0 Sep  2 15:21 memory.swappiness
-r--r--r-- 1 root root 0 Sep  2 15:12 memory.usage_in_bytes
-rw-r--r-- 1 root root 0 Sep  2 15:21 memory.use_hierarchy
-rw-r--r-- 1 root root 0 Sep  2 15:21 notify_on_release
-rw-r--r-- 1 root root 0 Sep  2 15:21 tasks
```

由上可以看到，容器 ID 的目录下有很多文件，其中 memory.limit_in_bytes 文件代表该容器内存限制大小，单位为 byte，我们使用 cat 命令（cat 命令可以查看文件内容）查看一下文件内容：

复制代码

```
$ sudo cat /sys/fs/cgroup/memory/docker/51041a74070e9260e82876974762b8c61c5ed0a51832d74fba6711175f89ede1/memory.limit_in_bytes
2147483648
```

这里可以看到memory.limit_in_bytes 的值为2147483648，转换单位后正好为 2G，符合我们启动容器时的内存限制 2G。

通过 memory 子系统的例子，我们可以知道**监控组件通过读取 memory.limit_in_bytes 文件即可获取到容器内存的限制值**。了解完容器的内存限制我们来了解一下容器的内存使用情况。

#### 监控系统是如何获取容器的内存使用状态的？

内存使用情况存放在 memory.usage_in_bytes 文件里，同样我们也使用 cat 命令查看一下文件内容:

复制代码

```
$ sudo /sys/fs/cgroup/memory/docker/51041a74070e9260e82876974762b8c61c5ed0a51832d74fba6711175f89ede1/memory.usage_in_bytes
4259840
```

可以看到当前内存的使用大小为 4259840 byte，约为 4 M。了解了内存的监控，下面我们来了解下网络的监控数据来源。

网络的监控数据来源是从 /proc/{PID}/net/dev 目录下读取的，其中 PID 为容器在主机上的进程 ID。下面我们首先使用 docker inspect 命令查看一下上面启动的 nginx 容器的 PID，命令如下：

复制代码

```
$ docker inspect nginx |grep Pid
            "Pid": 27348,
            "PidMode": "",
            "PidsLimit": 0,
```

可以看到容器的 PID 为 27348，使用 cat 命令查看一下 /proc/27348/net/dev 的内容：

复制代码

```
$ sudo cat /proc/27348/net/dev
Inter-|   Receive                                                |  Transmit
 face |bytes    packets errs drop fifo frame compressed multicast|bytes    packets errs drop fifo colls carrier compressed
    lo:       0       0    0    0    0     0          0         0        0       0    0    0    0     0       0          0
  eth0:       0       0    0    0    0     0          0         0        0       0    0    0    0     0       0          0
```

/proc/27348/net/dev 文件记录了该容器里每一个网卡的流量接收和发送情况，以及错误数、丢包数等信息。可见容器的网络监控数据都是定时从这里读取并展示的。

总结一下，**容器的监控原理其实就是定时读取 Linux 主机上相关的文件并展示给用户。**

### 结语

到此，相信你已经可以使用 docker stats 和 cAdvisor 监控并查看容器的状态了；也可以自己启动一个 cAdvisor 容器来监控主机和主机上的容器，并对监控系统的原理有了较深的了解。

试想下，cAdvisor 虽然可以临时存储一段历史监控数据，并且提供了一个简版的监控面板，在大规模的容器集群中，cAdvisor 有什么明显的不足吗？思考后，把你的想法写在留言区。





# 未完待续







































































































































































































































































































