# docker

docker官网：http://www.docker.com

docker中文网站：https://www.docker-cn.com/

Docker Hub官网: https://hub.docker.com/



Docker镜像的设计，使得Docker得以打破过去「程序即应用」的观念。透过镜像(images)将作业系统核心除外，运作应用程式所需要的系统环境，由下而上打包，达到应用程式跨平台间的无缝接轨运作



Docker是基于Go语言实现的云开源项目。

Docker的主要目标是“Build，Ship and Run Any App,Anywhere”，也就是通过对应用组件的封装、分发、部署、运行等生命周期的管理，使用户的APP（可以是一个WEB应用或数据库应用等等）及其运行环境能够做到“一次封装，到处运行”。

Linux 容器技术的出现就解决了这样一个问题，而 Docker 就是在它的基础上发展过来的。将应用运行在 Docker 容器上面，而 Docker 容器在任何操作系统上都是一致的，这就实现了跨平台、跨服务器。只需要一次配置好环境，换到别的机子上就可以一键部署好，大大简化了操作



**解决了运行环境和配置问题软件容器，方便做持续集成并有助于整体发布的容器虚拟化技术。**



**虚拟机技术**

虚拟机（virtual machine）就是带环境安装的一种解决方案。
它可以在一种操作系统里面运行另一种操作系统，比如在Windows 系统里面运行Linux 系统。应用程序对此毫无感知，因为虚拟机看上去跟真实系统一模一样，而对于底层系统来说，虚拟机就是一个普通文件，不需要了就删掉，对其他部分毫无影响。这类虚拟机完美的运行了另一套系统，能够使应用程序，操作系统和硬件三者之间的逻辑不变。

虚拟机的缺点：

1. 资源占用多       
2. 冗余步骤多 
3. 启动慢

**容器虚拟化技术**

由于前面虚拟机存在这些缺点，Linux 发展出了另一种虚拟化技术：Linux 容器（Linux Containers，缩写为 LXC）。
Linux 容器不是模拟一个完整的操作系统，而是对进程进行隔离。有了容器，就可以将软件运行所需的所有资源打包到一个隔离的容器中。容器与虚拟机不同，不需要捆绑一整套操作系统，只需要软件工作所需的库资源和设置。系统因此而变得高效轻量并保证部署在任何环境中的软件都能始终如一地运行。

比较了 Docker 和传统虚拟化方式的不同之处：

* 传统虚拟机技术是虚拟出一套硬件后，在其上运行一个完整操作系统，在该系统上再运行所需应用进程；
* 而容器内的应用进程直接运行于宿主的内核，容器内没有自己的内核，而且也没有进行硬件虚拟。因此容器要比传统虚拟机更为轻便。

* 每个容器之间互相隔离，每个容器有自己的文件系统 ，容器之间进程不会相互影响，能区分计算资源。



**开发/运维（DevOps）一次构建、随处运行**

- 更快速的应用交付和部署
- 更便捷的升级和扩缩容
- 更简单的系统运维
- 更高效的计算资源利用

## Docker安装

Docker支持以下的CentOS版本：
CentOS 7 (64-bit)
CentOS 6.5 (64-bit) 或更高的版本

Docker 运行在 CentOS 7 上，要求系统为64位、系统内核版本为 3.10 以上。
Docker 运行在 CentOS-6.5 或更高的版本的 CentOS 上，要求系统为64位、系统内核版本为 2.6.32-431 或者更高版本。

uname -r 命令用于打印当前系统相关信息（内核版本号、硬件架构、主机名称和操作系统类型等）。

### Docker的基本组成

1. Docker 镜像（Image）

   就是一个**只读**的模板。镜像可以用来创建 Docker 容器，**一个镜像可以创建很多容器**。

2. Docker 利用容器（Container）独立运行的一个或一组应用。**容器是用镜像创建的运行实例**。

   它可以被启动、开始、停止、删除。每个容器都是相互隔离的、保证安全的平台。

   **可以把容器看做是一个简易版的 Linux 环境**（包括root用户权限、进程空间、用户空间和网络空间等）和运行在其中的应用程序。

   容器的定义和镜像几乎一模一样，也是一堆层的统一视角，唯一区别在于容器的最上面那一层是可读可写的。

3. 仓库（Repository）是集中存放镜像文件的场所。
   仓库(Repository)和仓库注册服务器（Registry）是有区别的。仓库注册服务器上往往存放着多个仓库，每个仓库中又包含了多个镜像，每个镜像有不同的标签（tag）。

   仓库分为公开仓库（Public）和私有仓库（Private）两种形式。
   最大的公开仓库是 Docker Hub(https://hub.docker.com/)，
   存放了数量庞大的镜像供用户下载。国内的公开仓库包括阿里云 、网易云 等



### CentOS6.8安装Docker

```bash
yum install -y epel-release
#Docker使用EPEL发布，RHEL系的OS首先要确保已经持有EPEL仓库，否则先检查OS的版本，然后安装相应的EPEL包。
yum install -y docker-io
#安装后的配置文件：
/etc/sysconfig/docker
#启动Docker后台服务：
service docker start
docker version #验证
```

### CentOS7安装Docker

https://docs.docker.com/install/linux/docker-ce/centos/

https://docs.docker-cn.com/engine/installation/linux/docker-ce/centos/#prerequisites

```bash
cat /etc/redhat-release #查看linux版本

#yum安装gcc相关
yum -y install gcc
yum -y install gcc-c++

#卸载旧版本
yum -y remove docker docker-common docker-selinux docker-engine
#2018.3官网版本
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine


#安装需要的软件包
yum install -y yum-utils device-mapper-persistent-data lvm2

#设置stable镜像仓库
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

#更新yum软件包索引
yum makecache fast

#安装DOCKER CE
yum -y install docker-ce

#启动docker
systemctl start docker
systemctl enable docker --now  #开机启动

#测试
docker version
docker run hello-world

#配置镜像加速
mkdir -p /etc/docker
vim  /etc/docker/daemon.json
 #网易云
{"registry-mirrors": ["http://hub-mirror.c.163.com"] }
#阿里云
{"registry-mirrors": ["https://｛自已的编码｝.mirror.aliyuncs.com"]}
systemctl daemon-reload
systemctl restart docker
systemctl status docker 查看状态

#卸载
systemctl stop docker 
yum -y remove docker-ce
rm -rf /var/lib/docker
```

### 阿里云镜像加速

https://dev.aliyun.com/search.html

登陆阿里云开发者平台  获取加速器地址

配置本机Docker运行镜像加速器

```bash
vim /etc/sysconfig/docker
#   将获得的自己账户下的阿里云加速地址配置进
other_args="--registry-mirror=https://你自己的账号加速信息.mirror.aliyuncs.com"

#restart
service docker restart

#检查是否生效   --registry-mirror参数
ps aux|grep docker
```

**hello world**

docker run hello-world

## 底层原理

Docker是一个Client-Server结构的系统，Docker守护进程运行在主机上， 然后通过Socket连接从客户端访问，守护进程从客户端接受命令并管理运行在主机上的容器。 容器，是一个运行时环境，就是我们前面说到的集装箱。



为什么Docker比较比VM快

1. docker有着比虚拟机更少的抽象层。由亍docker不需要Hypervisor实现硬件资源虚拟化,运行在docker容器上的程序直接使用的都是实际物理机的硬件资源。因此在CPU、内存利用率上docker将会在效率上有明显优势。

2. docker利用的是宿主机的内核,而不需要Guest OS。因此,当新建一个容器时,docker不需要和虚拟机一样重新加载一个操作系统内核。仍而避免引寻、加载操作系统内核返个比较费时费资源的过程,当新建一个虚拟机时,虚拟机软件需要加载Guest OS,返个新建过程是分钟级别的。而docker由于直接利用宿主机的操作系统,则省略了返个过程,因此新建一个docker容器只需要几秒钟。
3. ![image-20210517140255204](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210517140255204.png)

##  Docker命令

![image-20210928163151503](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210928163151503.png)

###  帮助命令

```bash
docker version
docker info
docker --help
```

```bash
attach    Attach to a running container                 # 当前 shell 下 attach 连接指定运行镜像
build     Build an image from a Dockerfile              # 通过 Dockerfile 定制镜像
commit    Create a new image from a container changes   # 提交当前容器为新的镜像
cp        Copy files/folders from the containers filesystem to the host path   #从容器中拷贝指定文件或者目录到宿主机中
create    Create a new container                        # 创建一个新的容器，同 run，但不启动容器
diff      Inspect changes on a container's filesystem   # 查看 docker 容器变化
events    Get real time events from the server          # 从 docker 服务获取容器实时事件
exec      Run a command in an existing container        # 在已存在的容器上运行命令
export    Stream the contents of a container as a tar archive   # 导出容器的内容流作为一个 tar 归档文件[对应 import ]
history   Show the history of an image                  # 展示一个镜像形成历史
images    List images                                   # 列出系统当前镜像
import    Create a new filesystem image from the contents of a tarball # 从tar包中的内容创建一个新的文件系统映像[对应export]
info      Display system-wide information               # 显示系统相关信息
inspect   Return low-level information on a container   # 查看容器详细信息
kill      Kill a running container                      # kill 指定 docker 容器
load      Load an image from a tar archive              # 从一个 tar 包中加载一个镜像[对应 save]
login     Register or Login to the docker registry server    # 注册或者登陆一个 docker 源服务器
logout    Log out from a Docker registry server          # 从当前 Docker registry 退出
logs      Fetch the logs of a container                 # 输出当前容器日志信息
port      Lookup the public-facing port which is NAT-ed to PRIVATE_PORT    # 查看映射端口对应的容器内部源端口
pause     Pause all processes within a container        # 暂停容器
ps        List containers                               # 列出容器列表
pull      Pull an image or a repository from the docker registry server   # 从docker镜像源服务器拉取指定镜像或者库镜像
push      Push an image or a repository to the docker registry server    # 推送指定镜像或者库镜像至docker源服务器
restart   Restart a running container                   # 重启运行的容器
rm        Remove one or more containers                 # 移除一个或者多个容器
rmi       Remove one or more images             # 移除一个或多个镜像[无容器使用该镜像才可删除，否则需删除相关容器才可继续或 -f 强制删除]
run       Run a command in a new container              # 创建一个新的容器并运行一个命令
save      Save an image to a tar archive                # 保存一个镜像为一个 tar 包[对应 load]
search    Search for an image on the Docker Hub         # 在 docker hub 中搜索镜像
start     Start a stopped containers                    # 启动容器
stop      Stop a running containers                     # 停止容器
tag       Tag an image into a repository                # 给源中镜像打标签
top       Lookup the running processes of a container   # 查看容器中运行的进程信息
unpause   Unpause a paused container                    # 取消暂停容器
version   Show the docker version information           # 查看 docker 版本号
wait      Block until a container stops, then print its exit code   # 截取容器停止时的退出状态值
```

### 镜像命令

```bash
docker images #列出本地主机上的镜像
#各个选项说明:
REPOSITORY：#表示镜像的仓库源
TAG：	   #镜像的标签
IMAGE ID：  #镜像ID
CREATED：   #镜像创建时间   
			#镜像都是存储在Docker宿主机的/var/lib/docker目录下
NAME #仓库名称
DESCRIPTION #镜像描述
OFFICIAL #是否官方
AUTOMATED #自动构建，表示该镜像由Docker Hub自动构建流程创建的			
SIZE：	   #镜像大小
#同一仓库源可以有多个 TAG，代表这个仓库源的不同个版本，我们使用 REPOSITORY:TAG 来定义不同的镜像。
#如果你不指定一个镜像的版本标签，例如你只使用 ubuntu，docker 将默认使用 ubuntu:latest 镜像
#OPTIONS说明：
-a #列出本地所有的镜像（含中间映像层）
-q #只显示镜像ID。
--digests #显示镜像的摘要信息
--no-trunc #显示完整的镜像信息

docker search [OPTIONS] 镜像名字
#https://hub.docker.com
#OPTIONS说明：
--no-trunc #显示完整的镜像描述
-s #列出收藏数不小于指定值的镜像。
--automated #只列出 automated build类型的镜像；

docker pull 镜像名字[:TAG]

docker rmi  -f 镜像ID #删除一个
docker rmi -f 镜像名1:TAG 镜像名2:TAG #删除多个
docker rmi -f $(docker images -qa)  #删除全部
```

### 容器命令

```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
#新建并启动容器
#OPTIONS说明（常用）：有些是一个减号，有些是两个减号
--name="容器新名字" #为容器指定一个名称；
-d: #后台运行容器，并返回容器ID，也即启动守护式容器；
-i：#以交互模式运行容器，通常与 -t 同时使用；
-t：#为容器重新分配一个伪输入终端，通常与 -i 同时使用；
-P: #随机端口映射；
-p: #指定端口映射，有以下四种格式
      ip:hostPort:containerPort
      ip::containerPort
      hostPort:containerPort
      containerPort

docker ps [OPTIONS]
#列出当前所有正在运行的容器
#OPTIONS说明（常用）： 
-a #列出当前所有正在运行的容器+历史上运行过的
-l #显示最近创建的容器。
-n #显示最近n个创建的容器。
-q #静默模式，只显示容器编号。
-v #表示目录映射关系
-f #过滤
--no-trunc #不截断输出。

#退出容器
exit #容器停止退出
ctrl+P+Q #容器不停止退出

docker start 容器ID或者容器名 #启动容器
docker restart 容器ID或者容器名  #重启容器
docker stop 容器ID或者容器名  #停止容器
docker kill 容器ID或者容器名  #强制停止容器
docker rm 容器ID             #删除已停止的容器
#一次性删除多个容器
docker rm -f $(docker ps -a -q)
docker ps -a -q | xargs docker rm
```

```bash
docker run -d 容器名  #启动守护式容器
#Docker容器后台运行,就必须有一个前台进程.
#容器运行的命令如果不是那些一直挂起的命令（比如运行top，tail），就是会自动退出的。
#所以，最佳的解决方案是,将你要运行的程序以前台进程的形式运行

docker logs -f -t --tail 容器ID  #查看容器日志
  *   -t #是加入时间戳
  *   -f #跟随最新的日志打印
  *   --tail #数字 显示最后多少条

docker top 容器ID #查看容器内运行的进程
docker inspect 容器ID #查看容器内部细节
docker inspect mysql | grep IPAddress #查看mysql、nacos、sentinel容器的ip地址

docker exec -it 容器ID bashShell #进入正在运行的容器并以命令行交互
docker attach 容器ID    #重新进入
#exec 是在容器中打开新的终端，并且可以启动新的进程
#attach 直接进入容器启动命令的终端，不会启动新的进程

docker cp  容器ID:容器内路径 目的主机路径 #从容器内拷贝文件到主机上
```

## Docker 镜像


镜像是一种轻量级、可执行的独立软件包，**用来打包软件运行环境和基于运行环境开发的软件**，它包含运行某个软件所需的所有内容，包括代码、运行时、库、环境变量和配置文件。

**UnionFS（联合文件系统）**

Union文件系统（UnionFS）是一种分层、轻量级并且高性能的文件系统，<font color='red'>它支持对文件系统的修改作为一次提交来一层层的叠加</font>，同时可以将不同目录挂载到同一个虚拟文件系统下(unite several directories into a single virtual filesystem)。Union 文件系统是 Docker 镜像的基础。镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像。

特性：一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录



**Docker镜像加载原理：**
docker的镜像实际上由一层一层的文件系统组成，这种层级的文件系统UnionFS。
bootfs(boot file system)主要包含bootloader和kernel, bootloader主要是引导加载kernel, Linux刚启动时会加载bootfs文件系统，<font color='red'>在Docker镜像的最底层是bootfs</font>。这一层与我们典型的Linux/Unix系统是一样的，包含boot加载器和内核。当boot加载完成之后整个内核就都在内存中了，此时内存的使用权已由bootfs转交给内核，此时系统也会卸载bootfs。

rootfs (root file system) ，在bootfs之上。包含的就是典型 Linux 系统中的 /dev, /proc, /bin, /etc 等标准目录和文件。rootfs就是各种不同的操作系统发行版，比如Ubuntu，Centos等等。 

平时我们安装进虚拟机的CentOS都是好几个G，为什么docker这里才200M？？

对于一个精简的OS，rootfs可以很小，只需要包括最基本的命令、工具和程序库就可以了，因为底层直接用Host的kernel，自己只需要提供 rootfs 就行了。由此可见对于不同的linux发行版, bootfs基本是一致的, rootfs会有差别, 因此不同的发行版可以公用bootfs。



**最大的一个好处就是 - 共享资源**

比如：有多个镜像都从相同的 base 镜像构建而来，那么宿主机只需在磁盘上保存一份base镜像，
同时内存中也只需加载一份 base 镜像，就可以为所有容器服务了。而且镜像的每一层都可以被共享。



特点

Docker镜像都是只读的

当容器启动时，一个新的可写层被加载到镜像的顶部。这一层通常被称作“容器层”，“容器层”之下的都叫“镜像层”。

### commit操作

```bash
#提交容器副本使之成为一个新的镜像
docker commit -m=“提交的描述信息” -a=“作者” 容器ID 要创建的目标镜像名:[标签名]
```

## Docker容器数据卷


先来看看Docker的理念：
*  将运用与运行的环境打包形成容器运行 ，运行可以伴随着容器，但是我们对数据的要求希望是持久化的
*  容器之间希望有可能共享数据


Docker容器产生的数据，如果不通过docker commit生成新的镜像，使得数据做为镜像的一部分保存下来，
那么当容器删除后，数据自然也就没有了。

为了能保存数据在docker中我们使用卷。



 卷就是目录或文件，存在于一个或多个容器中，由docker挂载到容器，但不属于联合文件系统，因此能够绕过Union File System提供一些用于持续存储或共享数据的特性：

卷的设计目的就是数据的持久化，完全独立于容器的生存周期，因此Docker不会在容器删除时删除其挂载的数据卷

特点：

1. 数据卷可在容器之间共享或重用数据
2. 卷中的更改可以直接生效
3. 数据卷中的更改不会包含在镜像的更新中
4. 数据卷的生命周期一直持续到没有容器使用它为止



**容器间继承+共享数据**

**容器的持久化**



### 添加数据卷

容器内

```bash
 docker run -it -v /宿主机绝对路径目录:/容器内目录      镜像名
 docker inspect 容器ID  #查看数据卷是否挂载成功
 
 docker run -it -v /宿主机绝对路径目录:/容器内目录:ro 镜像名  #带权限  主机只读
 
#Docker挂载主机目录Docker访问出现cannot open directory .: Permission denied
#解决办法：在挂载目录后多加一个参数即可
--privileged=true
```

DockerFile

```bash
#可在Dockerfile中使用VOLUME指令来给镜像添加一个或多个数据卷
VOLUME["/dataVolumeContainer","/dataVolumeContainer2","/dataVolumeContainer3"] 
#出于可移植和分享的考虑，用-v 主机目录:容器目录这种方法不能够直接在Dockerfile中实现。
#由于宿主机目录是依赖于特定宿主机的，并不能够保证在所有的宿主机上都存在这样的特定目录。

#生成镜像
docker build -f [dockerfile] -t [image_name]
```

### 数据卷容器

命名的容器挂载数据卷，其它容器通过挂载这个(父容器)实现数据共享，挂载数据卷的容器，称之为数据卷容器

容器间传递共享(--volumes-from)

```bash
docker run -it --name a1 --volumes-from a2 zzyy/centos   #a1继承a2
```

容器之间配置信息的传递，数据卷的生命周期一直持续到没有容器使用它为止

## Dockerfile

Dockerfile是用来构建Docker镜像的构建文件，是由一系列命令和参数构成的脚本。

1. 编写Dockerfile文件
2. docker build
3. docker run



**Dockerfile内容基础知识**

1. 每条保留字指令都必须为大写字母且后面要跟随至少一个参数
2. 指令按照从上到下，顺序执行
3. # 表示注释
4. 每条指令都会创建一个新的镜像层，并对镜像进行提交



**Docker执行Dockerfile的大致流程**

1. docker从基础镜像运行一个容器
2. 执行一条指令并对容器作出修改
3. 执行类似docker commit的操作提交一个新的镜像层
4. docker再基于刚提交的镜像运行一个新容器
5. 执行dockerfile中的下一条指令直到所有指令都执行完成



从应用软件的角度来看，Dockerfile、Docker镜像与Docker容器分别代表软件的三个不同阶段，

*  Dockerfile是软件的原材料
*  Docker镜像是软件的交付品
*  Docker容器则可以认为是软件的运行态。

Dockerfile面向开发，Docker镜像成为交付标准，Docker容器则涉及部署与运维，三者缺一不可，合力充当Docker体系的基石。

1. Dockerfile，需要定义一个Dockerfile，Dockerfile定义了进程需要的一切东西。Dockerfile涉及的内容包括执行代码或者是文件、环境变量、依赖包、运行时环境、动态链接库、操作系统的发行版、服务进程和内核进程(当应用进程需要和系统服务和内核进程打交道，这时需要考虑如何设计namespace的权限控制)等等;

2. Docker镜像，在用Dockerfile定义一个文件之后，docker build时会产生一个Docker镜像，当运行 Docker镜像时，会真正开始提供服务;

3. Docker容器，容器是直接提供服务的。



### DockerFile体系结构(保留字指令)

```dockerfile
FROM  		#基础镜像，当前新镜像是基于哪个镜像的
MAINTAINER  #镜像维护者的姓名和邮箱地址
RUN			#容器构建时需要运行的命令
EXPOSE		#当前容器对外暴露出的端口
WORKDIR		#指定在创建容器后，终端默认登陆的进来工作目录，一个落脚点
ENV			#用来在构建镜像过程中设置环境变量
#ENV MY_PATH /usr/mytest
#这个环境变量可以在后续的任何RUN指令中使用，这就如同在命令前面指定了环境变量前缀一样；
#也可以在其它指令中直接使用这些环境变量，
#比如：WORKDIR $MY_PATH
ADD			#将宿主机目录下的文件拷贝进镜像且ADD命令会自动处理URL和解压tar压缩包
COPY  		#类似ADD，拷贝文件和目录到镜像中。
			#将从构建上下文目录中 <源路径> 的文件/目录复制到新的一层的镜像内的 <目标路径> 位置
			COPY src dest
			COPY ["src", "dest"]
VOLUME		#容器数据卷，用于数据保存和持久化工作
CMD			#指定一个容器启动时要运行的命令
			#Dockerfile 中可以有多个 CMD 指令，但只有最后一个生效，CMD 会被 docker run 之后的参数替换
			#shell格式 CMD <命令>
			#exec格式  CMD ["可执行文件", "参数1", "参数2"... ]
			#参数列表格式 CMD ["参数1", "参数2"... ], 在指定 ENTRYPOINT 后 用CMD指定具体参数
ENTRYPOINT  #指定一个容器启动时要运行的命令
			#ENTRYPOINT 的目的和 CMD 一样，都是在指定容器启动程序及参数
ONBUILD     #当构建一个被继承的Dockerfile时运行命令，父镜像在被子继承后父镜像的onbuild被触发
```

```dockerfile
#BUILD
    FROM
    MAINTAINER
    COPY
    ADD
    RUN	
    ONBUILD
    .dockerignore
#Both
    WORKDIR
    USER
#RUN
    ENV
    CMD
    EXPOSE
    VOLUME
    ENTRYPOINT
```

### demo

Base镜像(scratch)

Docker Hub 中 99% 的镜像都是通过在 base 镜像中安装和配置需要的软件构建出来的



#### 自定义镜像mycentos

```dockerfile
FROM centos
MAINTAINER zzyy<zzyy167@126.com>
 
ENV MYPATH /usr/local
WORKDIR $MYPATH
 
RUN yum -y install vim
RUN yum -y install net-tools
 
EXPOSE 80
 
CMD echo $MYPATH
CMD echo "success--------------ok"
CMD /bin/bash
```

```bash
#构建
docker build -t 新镜像名字:TAG .
#运行
docker run -it 新镜像名字:TAG 
#列出镜像的变更历史
docker history 镜像名
```

#### CMD/ENTRYPOINT 

都是指定一个容器启动时要运行的命令  -- **CMD 替换 ENTRYPOINT追加**

Dockerfile 中可以有多个 CMD 指令，但只有最后一个生效，CMD 会被 docker run 之后的参数替换

docker run 之后的参数会被当做参数传递给 ENTRYPOINT，之后形成新的命令组合

#### 自定义镜像Tomcat9

```bash
mkdir -p /zzyyuse/mydockerfile/tomcat9
touch c.txt
#将jdk和tomcat安装的压缩包拷贝进上一步目录
#apache-tomcat-9.0.8.tar.gz
#jdk-8u171-linux-x64.tar.gz
新建Dockerfile文件
```

```dockerfile
FROM         centos
MAINTAINER    zzyy<zzyybs@126.com>
#把宿主机当前上下文的c.txt拷贝到容器/usr/local/路径下
COPY c.txt /usr/local/cincontainer.txt
#把java与tomcat添加到容器中
ADD jdk-8u171-linux-x64.tar.gz /usr/local/
ADD apache-tomcat-9.0.8.tar.gz /usr/local/
#安装vim编辑器
RUN yum -y install vim
#设置工作访问时候的WORKDIR路径，登录落脚点
ENV MYPATH /usr/local
WORKDIR $MYPATH
#配置java与tomcat环境变量
ENV JAVA_HOME /usr/local/jdk1.8.0_171
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.8
ENV CATALINA_BASE /usr/local/apache-tomcat-9.0.8
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin
#容器运行时监听的端口
EXPOSE  8080
#启动时运行tomcat
# ENTRYPOINT ["/usr/local/apache-tomcat-9.0.8/bin/startup.sh" ]
# CMD ["/usr/local/apache-tomcat-9.0.8/bin/catalina.sh","run"]
CMD /usr/local/apache-tomcat-9.0.8/bin/startup.sh && tail -F /usr/local/apache-tomcat-9.0.8/bin/logs/catalina.out
```

```bash
#构建
docker build -t mytomcat9
#run
docker run -d -p 9080:8080 --name myt9 
-v /zzyyuse/mydockerfile/tomcat9/test:/usr/local/apache-tomcat-9.0.8/webapps/test 
-v /zzyyuse/mydockerfile/tomcat9/tomcat9logs/:/usr/local/apache-tomcat-9.0.8/logs 
--privileged=true zzyytomcat9
```

## Docker常用安装

### 安装tomcat

```bash
docker search tomcat
docker pull tomcat
docker images
docker run -it -p 8080:8080 tomcat
```

### mysql

```bash
docker run -p 12345:3306 --name mysql 
-v /zzyyuse/mysql/conf:/etc/mysql/conf.d 
-v /zzyyuse/mysql/logs:/logs 
-v /zzyyuse/mysql/data:/var/lib/mysql 
-e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.6
 
#命令说明：
-p 12345:3306#将主机的12345端口映射到docker容器的3306端口。
--name mysql#运行服务名字
-v /zzyyuse/mysql/conf:/etc/mysql/conf.d #将主机/zzyyuse/mysql录下的conf/my.cnf 挂载到容器的 /etc/mysql/conf.d
-v /zzyyuse/mysql/logs:/logs#将主机/zzyyuse/mysql目录下的 logs 目录挂载到容器的 /logs。
-v /zzyyuse/mysql/data:/var/lib/mysql#将主机/zzyyuse/mysql目录下的data目录挂载到容器的 /var/lib/mysql 
-e MYSQL_ROOT_PASSWORD=123456#初始化 root 用户的密码。
-d mysql:5.6 #后台程序运行mysql5.6

#进入
docker exec -it MySQL运行成功后的容器ID     /bin/bash
#备份
docker exec myql服务容器ID sh -c ' exec mysqldump --all-databases -uroot -p"123456" ' > /zzyyuse/all-databases.sql
```

### redis

```bash
docker run -p 6379:6379 
-v /zzyyuse/myredis/data:/data 
-v /zzyyuse/myredis/conf/redis.conf:/usr/local/etc/redis/redis.conf  
-d redis:3.2 redis-server /usr/local/etc/redis/redis.conf --appendonly yes

#新建redis.conf文件 bind 不要绑定本机 

docker exec -it 运行着Rediis服务的容器ID redis-cli
```









### nacos
```shell
docker run --name nacos -d 
-p 8847:8848
--restart=always
--privileged=true 
--restart=always 
-e JVM_XMS=512m 
-e JVM_XMX=512m 
-e MODE=standalone 
-v /Users/jeikerxiao/nacos:/home/nacos/logs 
nacos/nacos-server
```
### portainer
```shell
#可视化 portainer  
docker volume create portainer_data 
docker run -d -p 9000:9000 
--name portainer 
--restart always 
-v /var/run/docker.sock:/var/run/docker.sock 
-v portainer_data:/data portainer/portainer
```
### RabbitMQ
```shell
docker run -d --name rabbitmq -p 5671:5671 -p 5672:5672 -p 4369:4369 -p 25672:25672 -p 15671:15671 -p 15672:15672 rabbitmq:management 

4369, 25672 (Erlang发现&集群端口) 
5672, 5671 (AMQP端口) 
15672 (web管理后台端口) 
61613, 61614 (STOMP协议端口) 
1883, 8883 (MQTT协议端口) 
# 自动启动 
docker update rabbitmq --restart=always
```




## 本地镜像发布到阿里云

```bash
#从容器创建一个新的镜像
docker commit [OPTIONS] 容器ID [REPOSITORY[:TAG]]
#OPTIONS说明：
#-a :提交的镜像作者；
#-m :提交时的说明文字；

#创建阿里云镜像仓库

#将镜像推送到Registry
$ sudo docker login --username=cx**** registry.cn-hangzhou.aliyuncs.com
$ sudo docker tag [ImageId] registry.cn-hangzhou.aliyuncs.com/cxxzcc/demo:[镜像版本号]
$ sudo docker push registry.cn-hangzhou.aliyuncs.com/cxxzcc/demo:[镜像版本号]
```

## 监控

```shell
docker stats
```

## docker network





