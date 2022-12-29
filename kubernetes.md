# kubernetes

中文官网:https://kubernetes.io/zh/【推荐这个，然后点击 （学习 Kubernetes 基础知识）】
中文社区:https://www.kubernetes.org.cn/
官方文档:https://kubernetes.io/zh/docs/home/
社区文档:http://docs.kubernetes.org.cn/

## kubernetes 简介

简称 K8s。

是一个开源的，用于管理云平台中多个主机上的容器化的应用，Kubernetes 的目标是让部署容器化的 应用简单并且高效（powerful）,Kubernetes 提供了应用部署，规划，更新，维护的一种 机制。

 		传统的应用部署方式是通过插件或脚本来安装应用。这样做的缺点是应用的运行、配 置、管理、所有生存周期将与当前操作系统绑定，这样做并不利于应用的升级更新/回滚等 操作，当然也可以通过创建虚拟机的方式来实现某些功能，但是虚拟机非常重，并不利于 可移植性。 

​		新的方式是通过部署容器方式实现，每个容器之间互相隔离，每个容器有自己的文件 系统 ，容器之间进程不会相互影响，能区分计算资源。相对于虚拟机，容器能快速部署， 由于容器与底层设施、机器文件系统解耦的，所以它能在不同云、不同版本操作系统间进 行迁移。

 		容器占用资源少、部署快，每个应用可以被打包成一个容器镜像，每个应用与容器间 成一对一关系也使容器有更大优势，使用容器可以在 build 或 release 的阶段，为应用创 建容器镜像，因为每个应用不需要与其余的应用堆栈组合，也不依赖于生产环境基础结构， 这使得从研发到测试、生产能提供一致环境。类似地，容器比虚拟机轻量、更“透明”， 这更便于监控和管理。 

Kubernetes 是 Google 开源的一个容器编排引擎，它支持自动化部署、大规模可伸缩、 应用容器化管理。在生产环境中部署一个应用程序时，通常要部署该应用的多个实例以便 对应用请求进行负载均衡。 在 Kubernetes 中，我们可以创建多个容器，每个容器里面运行一个应用实例，然后通 过内置的负载均衡策略，实现对这一组应用实例的管理、发现、访问，而这些细节都不需 要运维人员去进行复杂的手工配置和处理。



**功能:** 

1. **自动装箱** 

   基于容器对应用运行环境的资源配置要求自动部署应用容器 

2. **自我修复(自愈能力)** 

   当容器失败时，会对容器进行重启 当所部署的 Node 节点有问题时，会对容器进行重新部署和重新调度

   当容器未通过监控检查时，会关闭此容器直到容器正常运行时，才会对外提供服务

3. **水平扩展** 

   通过简单的命令、用户 UI 界面或基于 CPU 等资源使用情况，对应用容器进行规模扩大 或规模剪裁 

4. **服务发现** 

   用户不需使用额外的服务发现机制，就能够基于 Kubernetes 自身能力实现服务发现和 负载均衡 

5. **滚动更新** 

   可以根据应用的变化，对应用容器运行的应用，进行一次性或批量式更新

6. **版本回退** 

   可以根据应用部署情况，对应用容器运行的应用，进行历史版本即时回退 

7. **密钥和配置管理** 

   在不需要重新构建镜像的情况下，可以部署和更新密钥和应用配置，类似热部署。 

8. **存储编排** 

   自动实现存储系统挂载及应用，特别对有状态应用实现数据持久化非常重要 存储系统可以来自于本地目录、网络存储(NFS、Gluster、Ceph 等)、公共云存储服务 

9. **批处理** 

   提供一次性任务，定时任务；满足批量数据处理和分析的场景



应用部署架构分类 

1. 无中心节点架构 

   GlusterFS

2. 有中心节点架构 

   HDFS 

   K8S

### **k8s 集群架构**

![image-20210519094109568](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210519094109568.png)

Kubernetes Cluster = N Master Node + N Worker Node：N主节点+N工作节点； N>=1



Master(主控节点)

1. apiserver

   集群统一入口,以restful方式, 交给etcd存储

2. Scheduler

   节点调度, 选择node节点应用部署

3. Controller MangerServer

   处理集群中常规后台任务,一个资源对应一个控制器

4. etcd

   存储系统

Node(工作节点)

1.  kubelet

   master排到node节点代表,管理本机容器

2. kube proxy 

   网络代理,负载均衡

3. ContainerRuntime；

**核心概念**

1. pod 

   最小部署单元 

   一组容器的集合 

   共享网络 

   生命周期短

2. controller 

   确保预期的pod副本数量 

   无状态应用部署 

   有状态应用部署 

   确保所有node运行同一个pod

   一次性任务和定时任务

3. service

   定义一组pod访问规则

4. Volume

   pod可访问文件目录

   可以挂载在pod中一个或多个容器指定目录

5. Label 标签 用于对象查询筛选

6. namespace 命名空间 逻辑隔离



![image-20211020154902527](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20211020154902527.png)

node --worker 服务器

* kubelet --一个在集群中每个[节点（node）](https://kubernetes.io/zh/docs/concepts/architecture/nodes/)上运行的代理。 它保证[容器（containers）](https://kubernetes.io/zh/docs/concepts/overview/what-is-kubernetes/#why-containers)都 运行在 [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) 中
* kube-proxy -- 是集群中每个节点上运行的网络代理

control plane  --master 控制面板

* etcd -- 存储 [etcd 文档](https://etcd.io/docs/)

* scheduler -- 调度
* controller manager --[控制器](https://kubernetes.io/zh/docs/concepts/architecture/controller/)
* cloud controller manager --将集群连接到云提供商的 API 之上， 并将与该云平台交互的组件同与你的集群交互的组件分离开来
* api server --[控制面](https://kubernetes.io/zh/docs/reference/glossary/?all=true#term-control-plane)



## 搭建

https://blog.csdn.net/hancoder/article/details/107612802

1. 搭建k8s环境平台规划

   单master

   多master

2. 服务器硬件配置要求

   测试环境 master 2核 4g 20g node 4核 8g 40g

   生产环境 更高

3. 搭建k8s集群部署方式

   1. kubeadm 

      Kubeadm 是一个 K8s 部署工具，提供 kubeadm init 和 kubeadm join，用于快速部 署 Kubernetes 集群。 官方地址：https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm/ 

      https://www.orchome.com/kubeadm/index

   2. 二进制包 
   
      从 github 下载发行版的二进制包，手动部署每个组件，组成 Kubernetes 集群。 Kubeadm 降低部署门槛，但屏蔽了很多细节，遇到问题很难排查。如果想更容易可 控，推荐使用二进制包部署 Kubernetes 集群，虽然手动部署麻烦点，期间可以学习很 多工作原理，也利于后期维护

### vagrant

#### 一键部署K8s集群

https://github.com/rootsongjc/kubernetes-vagrant-centos-cluster
http://github.com/davidkbainbridge/k8s-playground

#### 创建三台虚拟机

Vagrantfile文件

当前目录下cmd 

```bash
vagrant up
```

```bash
Vagrant.configure("2") do |config|
   (1..3).each do |i|
        config.vm.define "k8s-node#{i}" do |node|
            # 设置虚拟机的Box
            node.vm.box = "centos/7"

            # 设置虚拟机的主机名
            node.vm.hostname="k8s-node#{i}"

            # 设置虚拟机的IP
            node.vm.network "private_network", ip: "192.168.56.#{99+i}", netmask: "255.255.255.0"

            # 设置主机与虚拟机的共享目录
            # node.vm.synced_folder "~/Documents/vagrant/share", "/home/vagrant/share"

            # VirtaulBox相关配置
            node.vm.provider "virtualbox" do |v|
                # 设置虚拟机的名称
                v.name = "k8s-node#{i}"
                # 设置虚拟机的内存大小
                v.memory = 4096
                # 设置虚拟机的CPU个数
                v.cpus = 4
            end
        end
   end
end
```

进入到三个虚拟机，开启root的密码访问权限

```bash
vagrant ssh xxx #进入到系统后
# vagrant ssh k8s-node1
su root #密码为vagrant

vi /etc/ssh/sshd_config

#修改
PermitRootLogin yes 
PasswordAuthentication yes

#所有的虚拟机设为4核4G

service sshd restart
```

设置不同IP

选择三个节点，然后执行“管理”->“全局设定”->“网络”，添加一个NAT网络。

分别修改每台设备的网络类型，并刷新重新生成MAC地址。

https://blog.csdn.net/hancoder/article/details/107612802

### kubeadm

官方社区推出的一个用于快速部署 kubernetes 集群的工具，这个工具能通 过两条指令完成一个 kubernetes 集群的部署： 

```bash
# 创建一个 Master 节点
$ kubeadm init

# 将一个 Node 节点加入到当前集群中
$ kubeadm join <Master节点的IP和端口 >
```

安装要求

* 一台或多台机器，操作系统 CentOS7.x-86_x64
* 硬件配置：2GB 或更多 RAM，2 个 CPU 或更多 CPU，硬盘 30GB 或更多
* 集群中所有机器之间网络互通
* 可以访问外网，需要拉取镜像
* 禁止 swap 分区

最终目标

1. 在所有节点上安装 Docker 和 kubeadm 

2. 部署 Kubernetes Master 

3. 部署容器网络插件 

4. 部署 Kubernetes Node，将节点加入 Kubernetes 集群中 

5. 部署 Dashboard Web 页面，可视化查看 Kubernetes 资源

   

#### 准备环境

```bash
# 关闭防火墙
systemctl stop firewalld
systemctl disable firewalld

# 关闭selinux
sed -i 's/enforcing/disabled/' /etc/selinux/config  # 永久
setenforce 0  # 临时

# 关闭swap
swapoff -a  # 临时
sed -ri 's/.*swap.*/#&/' /etc/fstab    # 永久

# 根据规划设置主机名
hostnamectl set-hostname <hostname>

# 在master添加hosts
cat >> /etc/hosts << EOF
192.168.48.134    k8smaster134
192.168.48.129    k8snode129
EOF

# 将桥接的IPv4流量传递到iptables的链
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system  # 生效
	#遇见提示是只读的文件系统，运行如下命令
	mount -o remount rw /
	
# 时间同步
yum install ntpdate -y
ntpdate time.windows.com
```

####  所有节点安装Docker/kubeadm/kubelet

Kubernetes默认CRI（容器运行时）为Docker，因此先安装Docker。

##### 安装Docker

```bash
$ wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo
$ yum -y install docker-ce-18.06.1.ce-3.el7
$ systemctl enable docker && systemctl start docker
$ docker --version
Docker version 18.06.1-ce, build e68fc7a
```

```bash
$ cat > /etc/docker/daemon.json << EOF
{
  "registry-mirrors": ["https://b9pmyelo.mirror.aliyuncs.com"]
}
EOF
$ systemctl restart docker
```

或

```bash
#卸载之前的docker
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
#安装Docker -CE
sudo yum install -y yum-utils \
device-mapper-persistent-data \
lvm2
# 设置docker repo的yum位置
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
    
    # 安装docker，docker-cli
sudo yum -y install docker-ce docker-ce-cli containerd.io   

#配置docker加速
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://ke9h1pt4.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
#启动Docker && 设置docker开机启动
systemctl enable docker
```



##### 添加阿里云YUM软件源

```bash
$ cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
#或
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

##### 安装kubeadm，kubelet和kubectl

```bash
yum list|grep kube

$ yum install -y kubelet-1.18.0 kubeadm-1.18.0 kubectl-1.18.0
$ systemctl enable kubelet && systemctl start kubelet

systemctl status kubelet
kubelet --version

#删除
yum erase -y kubelet kubectl kubeadm
```

#### 部署Kubernetes Master

https://www.jianshu.com/p/69ba068a7e02



在（Master）执行。

```bash
$ kubeadm init \
  --apiserver-advertise-address=192.168.48.134 \
  --image-repository registry.cn-hangzhou.aliyuncs.com/google_containers \
  --kubernetes-version v1.18.0 \
  --service-cidr=10.96.0.0/12 \
  --pod-network-cidr=10.244.0.0/16
  
registry.cn-hangzhou.aliyuncs.com/google_containers#也可以
registry.aliyuncs.com/google_containers
#科普:无类别域间路由(Classless Inter-Domain Routing、CIDR) 是一个用于给用户分配IP
#-地址以及在互联网上有效地路由IP数据包的对IP地址进行归类的方法。

# Error writing Crisocket information for the control-plane node: timed out waiting for the condition
kubeadm reset
#再init
```

由于默认拉取镜像地址k8s.gcr.io国内无法访问，这里指定阿里云镜像仓库地址。

使用kubectl工具：

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
$ kubectl get nodes
```

#### 加入Kubernetes Node

在（Node）执行。

向集群添加新节点，执行在kubeadm init输出的kubeadm join命令：

```bash
$ kubeadm join 192.168.1.11:6443 --token esce21.q6hetwm8si29qxwn \
    --discovery-token-ca-cert-hash sha256:00603a05805807501d7181c3d60b478788408cfe6cedefedb1f97569708be9c5
```

默认token有效期为24小时，当过期之后，该token就不可用了。这时就需要重新创建token，操作如下

```bash
kubeadm token create --print-join-command
```

####  部署CNI网络插件

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

默认镜像地址无法访问，sed命令修改为docker hub镜像仓库。

```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

kubectl get pods -n kube-system
NAME                          READY   STATUS    RESTARTS   AGE
kube-flannel-ds-amd64-2pc95   1/1     Running   0          72s

#查看日志
journalctl -fu kubelet
```

#### 测试kubernetes集群

在Kubernetes集群中创建一个pod，验证是否正常运行：

```bash
$ kubectl create deployment nginx --image=nginx
$ kubectl expose deployment nginx --port=80 --type=NodePort
$ kubectl get pod,svc
```

访问地址：http://NodeIP:Port  

### 二进制

### minikube

https://github.com/kubernetes/minikube/releases

下载minikube -windows- amd64.exe改名为minikube.exe
打开VirtualBox,打开cmd,
运行

```shell
minikube start --vm-driver=virtualbox  --registry-mirror=https://registry.docker-cn.com
```



等待20分钟左右即可

```bash
minikube start --vm-driver=none --registry-mirror=https://registry.docker-cn.com

kubectl get pods --all-namespaces
```

### test

体验nginx部署升级

```bash
#提交一个nginx deployment
kubectl apply -f https://k8s.io/examples/application/deployment.yaml

#升级 nginx deployment
kubectl apply -f https://k8s.io/examples/application/deployment-update.yaml

#扩容 
nginx deployment
```

在主节点上部署一个tomcat

```bash
kubectl create deployment tomcat6 --image=tomcat:6.0.53-jre8

kubectl get all
kubectl get all -o wide
kubectl get pods
kubectl get pods --all-namespaces

#暴露端口访问
kubectl expose deployment tomcat6 --port=80 --target-port=8080 --type=NodePort 

#查看服务
kubectl get svc
kubectl get svc -o wide

#动态扩容测试
kubectl get deployment
kubectl scale --replicas=3 deployment tomcat6 #动态扩容3份
kubectl scale --replicas=1 deployment tomcat6 #缩容
```

yml部署

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: tomcat6
  name: tomcat6
spec:
  replicas: 3
  selector:
    matchLabels:
      app: tomcat6
  template:
    metadata: 
      labels:
        app: tomcat6
    spec:
      containers:
      - image: tomcat:6.0.53-jre8
        name: tomcat
---
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: tomcat6
  name: tomcat6
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 8080
  selector:
    app: tomcat6
  type: NodePort
```

```bash
kubectl apply -f tomcat6-deployment.yaml
```

#### Ingress

通过Ingress发现pod进行关联。基于域名访问
通过Ingress controller实现POD负载均衡
支持TCP/UDP 4层负载均衡和HTTP 7层负载均衡

- service管理多个pod
- Ingress管理多个service

```bash
kubectl apply -f ingress-controller.yaml 
```

### 安装kubernetes可视化界面——DashBoard

```
kubectl appy -f  kubernetes-dashboard.yaml
```

暴露DashBoard为公共访问

默认DashBoard只能集群内部访问，修改Service为NodePort类型，暴露到外部

```yaml
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  type: NodePort
  ports:
    - port: 443
      targetPort: 8443
      nodePort: 3001
  selector:
    k8s-app: kubernetes-dashboard

```

```bash
#创建授权账号
$ kubectl create serviceaccount dashboar-admin -n kube-sysem
$ kubectl describe secrets -n kube-system $( kubectl -n kube-system get secret |awk '/dashboard-admin/{print $1}' )
$ kubectl create clusterrolebinding dashboar-admin --clusterrole=cluter-admin --serviceaccount=kube-system:dashboard-admin

```

### kubesphere

https://kubesphere.io

https://www.yuque.com/leifengyang/oncloud/gby0ar

https://www.kubernetes.org.cn/7315.html 

```bash
./kk delete cluster
```

#### 安装

##### 安装helm（master节点执行）

```bash
curl -L https://git.io/get_helm.sh|bash
#由于被墙的原因，使用我们给定的get_helm.sh。
sh get_helm.sh
```

#### 部署

![image-20211008170036391](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20211008170036391.png)



### Kuboard

kuboard也很不错，集群要求不高

https://kuboard.cn/



### 高可用集群搭建

Etcd 数据库的高可用性和Kubernetes Master 组件的高可用性

Etcd 我们已经采用3 个节点组建集群实现高可用

Master 节点主要有三个服务kube-apiserver、kube-controller-mansger 和kubescheduler，其中kube-controller-mansger 和kube-scheduler 组件自身通过选择机制已经实现了高可用，所以Master 高可用主要针对kube-apiserver 组件，而该组件是以HTTPAPI 提供服务，因此对他高可用与Web 服务器类似，==增加负载均衡器对其负载均衡，并可水平扩容。==















## yaml

资源清单文件

资源编排

### 格式

--- 代表新的yaml文件

支持的数据结构

```yml
#对象
#键值对的集合，又称为映射(mapping) / 哈希（hashes） / 字典（dictionary)
name: tom
age: 18
#或
hash: { name: tom, age: 18 }

#数组
#一组按次序排列的值，又称为序列（sequence） / 列表 （list)
People
- tom
- jack
#或
People: [tom, jack]

#null用 ~ 表示
parent: ~

#!! 强制转换数据类型
e: !!str 123
e: !!str true

#单引号之中如果还有单引号，必须连续使用两个单引号转义
#字符串可以写成多行，从第二行开始，必须有一个单空格缩进。换行符会被转为空格
#多行字符串可以使用|保留换行符，也可以使用>折叠换行
```

### 组成部分

1. 控制器定义
2. 被控制对象

### 常用字段

\* 必须存在的属性

![image-20210521110603561](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210521110603561.png)



```shell
kubectl apply -f xx.yaml
```



### 如何快速编写yaml文件

1. 使用kubectl create命 令生成yaml文件

   ```bash
   kubectl create deployment web --images=nginx -o yaml --dry-run > my1.yaml
   ```

2. 使用kubectl get命令导出yaml文件

   ```bash
   kubectl get deploy nginx -o=yaml --export > my2.yaml
   ```

   

## kubectl

https://kubernetes.io/zh/docs/reference/kubectl/overview/

https://kubernetes.io/zh/docs/reference/kubectl/overview/#资源类型

https://kubernetes.io/zh/docs/reference/kubectl/overview/

https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#create 命令文档



Kubernetes 集群的命令行工具，通过 kubectl 能够对集群本身进行管理，并能 够在集群上进行容器化应用的安装部署。

语法

```bash
kubectl [command] [type] [name] [flags]
#comand：指定要对资源执行的操作，例如 create、get、describe 和 delete
#TYPE：指定资源类型，资源类型是大小写敏感的，开发者能够以单数、复数和缩略的形式。例如：
    kubectl get pod pod1
    kubectl get pods pod1
    kubectl get po pod1
#NAME：指定资源的名称，名称也大小写敏感的。如果省略名称，则会显示所有的资源，例如
	kubectl get pods
#flags：指定可选的参数。例如，可用-s 或者–server 参数指定 Kubernetes API server 的地址和端口。

kubectl --help

kubectl get all
kubectl get all -o
kubectl get cs
kubectl get nodes
kubectl get svc,pod
#以下命令将单个 pod 的详细信息输出为 YAML 格式的对象：
kubectl get pod web-pod-13je7 -o yaml

–dry-run：
#–dry-run=‘none’: Must be “none”, “server”, or “client”通过–dry-run选项，并不会真正的执行这条命令。

kubectl create deployment tomcat6 --image=tomcat:6.0.53-jre8 --dry-run -o yaml >tomcat6.yaml

#查看某个pod的具体信息：
kubectl get pods tomcat6-7b84fb5fdc-5jh6t  -o yaml
```

kubeconfig

```shell
cat $HOME/.kube/config
```



## 概念

### namespace

名称空间用来隔离资源
默认只隔离资源，不隔离网络

```shell
kubectl create ns hello
kubectl delete ns hello
```

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: hello
```

### Label

标签, 用于对象资源查询筛选

### pod

1. 基本概念

   运行中的一组容器，Pod是kubernetes中应用的最小单位

   一个pod中的容器共享网络命名空间

    pod是短暂的

2. Pod存在意义
   1. docker, 单进程
   
   2.  Pod是多进程设计，运行多个应用程序
      
      * 一个Pod 可以有多个容器，彼此间共享网络和存储资源, 互相用loaclhost调用, 占用端口不能重复，每个Pod 中有一个Pause 容器保存所有的容器状态， 通过管理pause 容器，达到管理pod 中所有容器的效果
      
   3. Pod存在为了亲密性应用
        * 两个应用之间进行交互
        * 网络之间调用
        * 两个应用需要频繁调用
        
   4. pod特点
   
        * 资源共享 网络 (Pause 添加其他业务容器在同名称空间实现网络共享) 存储 (Volume)
        * 生命周期短暂
        * pod间通过ip调用
   
        

```shell
kubectl run mynginx --image=nginx

# 查看default名称空间的Pod
kubectl get pod 
# 描述
kubectl describe pod 你自己的Pod名字
# 删除
kubectl delete pod Pod名字
# 查看Pod的运行日志
kubectl logs Pod名字

# 每个Pod - k8s都会分配一个ip
kubectl get pod -owide
# 使用Pod的ip+pod里面运行容器的端口
curl 192.168.169.136

# 集群中的任意一个机器以及任意的应用都能通过Pod分配的ip来访问这个Pod

#进入pod
kubectl exec -it mynginx -- /bin/bash
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: mynginx
  name: mynginx
#  namespace: default
spec:
  containers:
  - image: nginx
    name: mynginx
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: myapp
  name: myapp
spec:
  containers:
  - image: nginx
    name: nginx
  - image: tomcat:8.5.68
    name: tomcat
```

**此时的应用还不能外部访问**

```yaml
imagesPullPolicy: 拉取镜像的策略 always每次都拉 IfNotPresent:没有在拉 never不拉
#资源限制
resources.limits.cpu #最大
resources.limits.memory
resources..requests.cpu #最小

restartPolicy: [Always|Never|OnFailure (退出码非0时)] #重启策略，默认值为Always

#健康检查 http/exec/tcpSocket
livenessProbe: #存活检查 失败杀死容器 走restartPolicy
readinessProbe: #就绪检查 失败从service endpoint剔除
```

静态Pod 是由kubelet 进行管理的仅存在于特定Node 上的Pod

Pod 的状态

| 值        |                                                              |
| --------- | ------------------------------------------------------------ |
| Pending   | API Server已经创建了该Pod,但Pod中的一一个或多个容器的镜像还没有创建,包括镜像下载过程 |
| Running   | Pod内所有容器已创建，且至少-一个容器处于运行状态、正在启动状态或正在重启状态 |
| Completed | Pod内所有容器均成功执行退出，且不会再重启                    |
| Failed    | Pod内所有容器均已退出，但至少一个容器退出失败                |
| Unkown    | 由于某种原因无法获取Pod状态，例如网络通信不畅                |

pod调度策略

* 资源限制

* nodeSelector.env_role: dev 节点选择器  

  ```shell
  kubectl label node node1 env_role=dev
  kubectl get nodes node1 --show-label
  ```

* 节点亲和性 nodeAffinity

  * 硬亲和性(必须满足)
  * 软亲和性(尝试满足)![image-20211029172441109](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20211029172441109.png)

Taint污点: 节点不做普通分配, 节点属性

```shell
kubectl describe node master1 | grep Taint #查
kubectl taint node node1 key=value:三个值 #加taint
kubectl taint node node1 env_role:NoSchedule- #删除taint
```

* NoSchedule 不调用
* PreferNoSchdule: 尽量不调
* NoExecute 不调, 驱逐已有pod

污点容忍

![image-20211029173614047](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20211029173614047.png)

### 工作负载

有状态应用使用  `StatefulSet`  部署，无状态应用使用 `Deployment` 部署

https://kubernetes.io/zh/docs/concepts/workloads/controllers/



Deployment:无状态应用部署,比如微服务，提供多副本等功能
StatefulSet:有状态应用部署， 比如redis, 提供稳定的存储、网络等功能
DaemonSet:守护型应用部署 ，比如日志收集组件，在每个机器都运行一份
Job/CronJob:定时任务部署，比如垃圾清理组件， 可以在指定时间运行



#### [Deployments](https://kubernetes.io/zh/docs/concepts/workloads/controllers/deployment/)

通过label控制Pod，使Pod拥有多副本，自愈，扩缩容等能力

```shell
# 清除所有Pod，比较下面两个命令有何不同效果？
kubectl run mynginx --image=nginx

kubectl create deployment mytomcat --image=tomcat:8.5.68
# 自愈能力
```

##### 多副本

```shell
kubectl create deployment my-dep --image=nginx --replicas=3
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: my-dep
  name: my-dep
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-dep
  template:
    metadata:
      labels:
        app: my-dep
    spec:
      containers:
      - image: nginx
        name: nginx
```

##### 扩容缩容

```shell
kubectl scale --replicas=5 deployment/my-dep
```

```shell
kubectl edit deployment my-dep
#修改 replicas
```

##### 自愈&故障转移

- 停机
- 删除Pod

- 容器崩溃
- ....

##### 滚动更新

```shell
kubectl set image deployment/my-dep nginx=nginx:1.16.1 --record
kubectl rollout status deployment/my-dep
```

```shell
# 修改 kubectl edit deployment/my-dep
```

##### 版本回退

```shell
#历史记录
kubectl rollout history deployment/my-dep

#查看某个历史详情
kubectl rollout history deployment/my-dep --revision=2

#回滚(回到上次)
kubectl rollout undo deployment/my-dep

#回滚(回到指定版本)
kubectl rollout undo deployment/my-dep --to-revision=2
```

#### [ReplicaSet](https://kubernetes.io/zh/docs/concepts/workloads/controllers/replicaset/)

#### [StatefulSets](https://kubernetes.io/zh/docs/concepts/workloads/controllers/statefulset/)

#### [DaemonSet](https://kubernetes.io/zh/docs/concepts/workloads/controllers/daemonset/)

#### [Jobs](https://kubernetes.io/zh/docs/concepts/workloads/controllers/job/)

#### [垃圾收集](https://kubernetes.io/zh/docs/concepts/workloads/controllers/garbage-collection/)

#### [已完成资源的 TTL 控制器](https://kubernetes.io/zh/docs/concepts/workloads/controllers/ttlafterfinished/)

#### [CronJob](https://kubernetes.io/zh/docs/concepts/workloads/controllers/cron-jobs/)

#### [ReplicationController](https://kubernetes.io/zh/docs/concepts/workloads/controllers/replicationcontroller/)

### Service

Service：Pod的服务发现与负载均衡

将一组 [Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) 公开为网络服务的抽象方法

```shell
kubectl get svc -A
```



```shell
#暴露Deploy
kubectl expose deployment my-dep --port=8000 --target-port=80

#使用标签检索Pod
kubectl get pod -l app=my-dep
```

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: my-dep
  name: my-dep
spec:
  selector:
    app: my-dep
  ports:
  - port: 8000
    protocol: TCP
    targetPort: 80
```

#### ClusterIP

集群内使用service的ip:port就可以负载均衡的访问

服务名-所在名称空间.svc
my-dep.default.svc

```shell
kubectl get pods -A --show-labels

# 等同于没有--type的
kubectl expose deployment my-dep --port=8000 --target-port=80 --type=ClusterIP
```

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: my-dep
  name: my-dep
spec:
  ports:
  - port: 8000
    protocol: TCP
    targetPort: 80
  selector:
    app: my-dep
  type: ClusterIP
```

#### NodePort

集群外访问

```shell
kubectl expose deployment my-dep --port=8000 --target-port=80 --type=NodePort
```

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: my-dep
  name: my-dep
spec:
  ports:
  - port: 8000
    protocol: TCP
    targetPort: 80
  selector:
    app: my-dep
  type: NodePort
```

> NodePort范围在 30000-32767 之间

#### LoadBalancer

对外访问, 公有云







### Ingress

Ingress：Service的统一网关入口

底层nginx

#### 安装

```shell
wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.47.0/deploy/static/provider/baremetal/deploy.yaml

#修改镜像
vi deploy.yaml
#将image的值改为如下值：
registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images/ingress-nginx-controller:v0.46.0

# 检查安装的结果
kubectl get pod,svc -n ingress-nginx

# 最后别忘记把svc暴露的端口要放行
```

如果下载不到，用以下文件

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx

---
# Source: ingress-nginx/templates/controller-serviceaccount.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    helm.sh/chart: ingress-nginx-3.33.0
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/version: 0.47.0
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: controller
  name: ingress-nginx
  namespace: ingress-nginx
automountServiceAccountToken: true
---
# Source: ingress-nginx/templates/controller-configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  labels:
    helm.sh/chart: ingress-nginx-3.33.0
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/version: 0.47.0
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: controller
  name: ingress-nginx-controller
  namespace: ingress-nginx
data:
---
# Source: ingress-nginx/templates/clusterrole.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    helm.sh/chart: ingress-nginx-3.33.0
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/version: 0.47.0
    app.kubernetes.io/managed-by: Helm
  name: ingress-nginx
rules:
  - apiGroups:
      - ''
    resources:
      - configmaps
      - endpoints
      - nodes
      - pods
      - secrets
    verbs:
      - list
      - watch
  - apiGroups:
      - ''
    resources:
      - nodes
    verbs:
      - get
  - apiGroups:
      - ''
    resources:
      - services
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - extensions
      - networking.k8s.io   # k8s 1.14+
    resources:
      - ingresses
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - ''
    resources:
      - events
    verbs:
      - create
      - patch
  - apiGroups:
      - extensions
      - networking.k8s.io   # k8s 1.14+
    resources:
      - ingresses/status
    verbs:
      - update
  - apiGroups:
      - networking.k8s.io   # k8s 1.14+
    resources:
      - ingressclasses
    verbs:
      - get
      - list
      - watch
---
# Source: ingress-nginx/templates/clusterrolebinding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    helm.sh/chart: ingress-nginx-3.33.0
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/version: 0.47.0
    app.kubernetes.io/managed-by: Helm
  name: ingress-nginx
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: ingress-nginx
subjects:
  - kind: ServiceAccount
    name: ingress-nginx
    namespace: ingress-nginx
---
# Source: ingress-nginx/templates/controller-role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  labels:
    helm.sh/chart: ingress-nginx-3.33.0
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/version: 0.47.0
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: controller
  name: ingress-nginx
  namespace: ingress-nginx
rules:
  - apiGroups:
      - ''
    resources:
      - namespaces
    verbs:
      - get
  - apiGroups:
      - ''
    resources:
      - configmaps
      - pods
      - secrets
      - endpoints
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - ''
    resources:
      - services
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - extensions
      - networking.k8s.io   # k8s 1.14+
    resources:
      - ingresses
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - extensions
      - networking.k8s.io   # k8s 1.14+
    resources:
      - ingresses/status
    verbs:
      - update
  - apiGroups:
      - networking.k8s.io   # k8s 1.14+
    resources:
      - ingressclasses
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - ''
    resources:
      - configmaps
    resourceNames:
      - ingress-controller-leader-nginx
    verbs:
      - get
      - update
  - apiGroups:
      - ''
    resources:
      - configmaps
    verbs:
      - create
  - apiGroups:
      - ''
    resources:
      - events
    verbs:
      - create
      - patch
---
# Source: ingress-nginx/templates/controller-rolebinding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  labels:
    helm.sh/chart: ingress-nginx-3.33.0
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/version: 0.47.0
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: controller
  name: ingress-nginx
  namespace: ingress-nginx
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: ingress-nginx
subjects:
  - kind: ServiceAccount
    name: ingress-nginx
    namespace: ingress-nginx
---
# Source: ingress-nginx/templates/controller-service-webhook.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    helm.sh/chart: ingress-nginx-3.33.0
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/version: 0.47.0
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: controller
  name: ingress-nginx-controller-admission
  namespace: ingress-nginx
spec:
  type: ClusterIP
  ports:
    - name: https-webhook
      port: 443
      targetPort: webhook
  selector:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/component: controller
---
# Source: ingress-nginx/templates/controller-service.yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
  labels:
    helm.sh/chart: ingress-nginx-3.33.0
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/version: 0.47.0
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: controller
  name: ingress-nginx-controller
  namespace: ingress-nginx
spec:
  type: NodePort
  ports:
    - name: http
      port: 80
      protocol: TCP
      targetPort: http
    - name: https
      port: 443
      protocol: TCP
      targetPort: https
  selector:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/component: controller
---
# Source: ingress-nginx/templates/controller-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    helm.sh/chart: ingress-nginx-3.33.0
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/version: 0.47.0
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: controller
  name: ingress-nginx-controller
  namespace: ingress-nginx
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: ingress-nginx
      app.kubernetes.io/instance: ingress-nginx
      app.kubernetes.io/component: controller
  revisionHistoryLimit: 10
  minReadySeconds: 0
  template:
    metadata:
      labels:
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/component: controller
    spec:
      dnsPolicy: ClusterFirst
      containers:
        - name: controller
          image: registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images/ingress-nginx-controller:v0.46.0
          imagePullPolicy: IfNotPresent
          lifecycle:
            preStop:
              exec:
                command:
                  - /wait-shutdown
          args:
            - /nginx-ingress-controller
            - --election-id=ingress-controller-leader
            - --ingress-class=nginx
            - --configmap=$(POD_NAMESPACE)/ingress-nginx-controller
            - --validating-webhook=:8443
            - --validating-webhook-certificate=/usr/local/certificates/cert
            - --validating-webhook-key=/usr/local/certificates/key
          securityContext:
            capabilities:
              drop:
                - ALL
              add:
                - NET_BIND_SERVICE
            runAsUser: 101
            allowPrivilegeEscalation: true
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
            - name: LD_PRELOAD
              value: /usr/local/lib/libmimalloc.so
          livenessProbe:
            failureThreshold: 5
            httpGet:
              path: /healthz
              port: 10254
              scheme: HTTP
            initialDelaySeconds: 10
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 1
          readinessProbe:
            failureThreshold: 3
            httpGet:
              path: /healthz
              port: 10254
              scheme: HTTP
            initialDelaySeconds: 10
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 1
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
            - name: https
              containerPort: 443
              protocol: TCP
            - name: webhook
              containerPort: 8443
              protocol: TCP
          volumeMounts:
            - name: webhook-cert
              mountPath: /usr/local/certificates/
              readOnly: true
          resources:
            requests:
              cpu: 100m
              memory: 90Mi
      nodeSelector:
        kubernetes.io/os: linux
      serviceAccountName: ingress-nginx
      terminationGracePeriodSeconds: 300
      volumes:
        - name: webhook-cert
          secret:
            secretName: ingress-nginx-admission
---
# Source: ingress-nginx/templates/admission-webhooks/validating-webhook.yaml
# before changing this value, check the required kubernetes version
# https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/#prerequisites
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  labels:
    helm.sh/chart: ingress-nginx-3.33.0
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/version: 0.47.0
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: admission-webhook
  name: ingress-nginx-admission
webhooks:
  - name: validate.nginx.ingress.kubernetes.io
    matchPolicy: Equivalent
    rules:
      - apiGroups:
          - networking.k8s.io
        apiVersions:
          - v1beta1
        operations:
          - CREATE
          - UPDATE
        resources:
          - ingresses
    failurePolicy: Fail
    sideEffects: None
    admissionReviewVersions:
      - v1
      - v1beta1
    clientConfig:
      service:
        namespace: ingress-nginx
        name: ingress-nginx-controller-admission
        path: /networking/v1beta1/ingresses
---
# Source: ingress-nginx/templates/admission-webhooks/job-patch/serviceaccount.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: ingress-nginx-admission
  annotations:
    helm.sh/hook: pre-install,pre-upgrade,post-install,post-upgrade
    helm.sh/hook-delete-policy: before-hook-creation,hook-succeeded
  labels:
    helm.sh/chart: ingress-nginx-3.33.0
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/version: 0.47.0
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: admission-webhook
  namespace: ingress-nginx
---
# Source: ingress-nginx/templates/admission-webhooks/job-patch/clusterrole.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: ingress-nginx-admission
  annotations:
    helm.sh/hook: pre-install,pre-upgrade,post-install,post-upgrade
    helm.sh/hook-delete-policy: before-hook-creation,hook-succeeded
  labels:
    helm.sh/chart: ingress-nginx-3.33.0
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/version: 0.47.0
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: admission-webhook
rules:
  - apiGroups:
      - admissionregistration.k8s.io
    resources:
      - validatingwebhookconfigurations
    verbs:
      - get
      - update
---
# Source: ingress-nginx/templates/admission-webhooks/job-patch/clusterrolebinding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: ingress-nginx-admission
  annotations:
    helm.sh/hook: pre-install,pre-upgrade,post-install,post-upgrade
    helm.sh/hook-delete-policy: before-hook-creation,hook-succeeded
  labels:
    helm.sh/chart: ingress-nginx-3.33.0
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/version: 0.47.0
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: admission-webhook
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: ingress-nginx-admission
subjects:
  - kind: ServiceAccount
    name: ingress-nginx-admission
    namespace: ingress-nginx
---
# Source: ingress-nginx/templates/admission-webhooks/job-patch/role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: ingress-nginx-admission
  annotations:
    helm.sh/hook: pre-install,pre-upgrade,post-install,post-upgrade
    helm.sh/hook-delete-policy: before-hook-creation,hook-succeeded
  labels:
    helm.sh/chart: ingress-nginx-3.33.0
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/version: 0.47.0
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: admission-webhook
  namespace: ingress-nginx
rules:
  - apiGroups:
      - ''
    resources:
      - secrets
    verbs:
      - get
      - create
---
# Source: ingress-nginx/templates/admission-webhooks/job-patch/rolebinding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: ingress-nginx-admission
  annotations:
    helm.sh/hook: pre-install,pre-upgrade,post-install,post-upgrade
    helm.sh/hook-delete-policy: before-hook-creation,hook-succeeded
  labels:
    helm.sh/chart: ingress-nginx-3.33.0
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/version: 0.47.0
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: admission-webhook
  namespace: ingress-nginx
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: ingress-nginx-admission
subjects:
  - kind: ServiceAccount
    name: ingress-nginx-admission
    namespace: ingress-nginx
---
# Source: ingress-nginx/templates/admission-webhooks/job-patch/job-createSecret.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: ingress-nginx-admission-create
  annotations:
    helm.sh/hook: pre-install,pre-upgrade
    helm.sh/hook-delete-policy: before-hook-creation,hook-succeeded
  labels:
    helm.sh/chart: ingress-nginx-3.33.0
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/version: 0.47.0
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: admission-webhook
  namespace: ingress-nginx
spec:
  template:
    metadata:
      name: ingress-nginx-admission-create
      labels:
        helm.sh/chart: ingress-nginx-3.33.0
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/version: 0.47.0
        app.kubernetes.io/managed-by: Helm
        app.kubernetes.io/component: admission-webhook
    spec:
      containers:
        - name: create
          image: docker.io/jettech/kube-webhook-certgen:v1.5.1
          imagePullPolicy: IfNotPresent
          args:
            - create
            - --host=ingress-nginx-controller-admission,ingress-nginx-controller-admission.$(POD_NAMESPACE).svc
            - --namespace=$(POD_NAMESPACE)
            - --secret-name=ingress-nginx-admission
          env:
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
      restartPolicy: OnFailure
      serviceAccountName: ingress-nginx-admission
      securityContext:
        runAsNonRoot: true
        runAsUser: 2000
---
# Source: ingress-nginx/templates/admission-webhooks/job-patch/job-patchWebhook.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: ingress-nginx-admission-patch
  annotations:
    helm.sh/hook: post-install,post-upgrade
    helm.sh/hook-delete-policy: before-hook-creation,hook-succeeded
  labels:
    helm.sh/chart: ingress-nginx-3.33.0
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/version: 0.47.0
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: admission-webhook
  namespace: ingress-nginx
spec:
  template:
    metadata:
      name: ingress-nginx-admission-patch
      labels:
        helm.sh/chart: ingress-nginx-3.33.0
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/version: 0.47.0
        app.kubernetes.io/managed-by: Helm
        app.kubernetes.io/component: admission-webhook
    spec:
      containers:
        - name: patch
          image: docker.io/jettech/kube-webhook-certgen:v1.5.1
          imagePullPolicy: IfNotPresent
          args:
            - patch
            - --webhook-name=ingress-nginx-admission
            - --namespace=$(POD_NAMESPACE)
            - --patch-mutating=false
            - --secret-name=ingress-nginx-admission
            - --patch-failure-policy=Fail
          env:
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
      restartPolicy: OnFailure
      serviceAccountName: ingress-nginx-admission
      securityContext:
        runAsNonRoot: true
        runAsUser: 2000
```

#### 使用

官网地址：https://kubernetes.github.io/ingress-nginx/

> https://139.198.163.211:32401/
>
> [http://139.198.163.211:31405/](https://139.198.163.211:32401/)

##### 测试环境

应用如下yaml，准备好测试环境

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-server
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hello-server
  template:
    metadata:
      labels:
        app: hello-server
    spec:
      containers:
      - name: hello-server
        image: registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images/hello-server
        ports:
        - containerPort: 9000
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-demo
  name: nginx-demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-demo
  template:
    metadata:
      labels:
        app: nginx-demo
    spec:
      containers:
      - image: nginx
        name: nginx
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: nginx-demo
  name: nginx-demo
spec:
  selector:
    app: nginx-demo
  ports:
  - port: 8000
    protocol: TCP
    targetPort: 80
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: hello-server
  name: hello-server
spec:
  selector:
    app: hello-server
  ports:
  - port: 8000
    protocol: TCP
    targetPort: 9000
```

##### 1、域名访问

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress  
metadata:
  name: ingress-host-bar
spec:
  ingressClassName: nginx
  rules:
  - host: "hello.atguigu.com"
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: hello-server
            port:
              number: 8000
  - host: "demo.atguigu.com"
    http:
      paths:
      - pathType: Prefix
        path: "/nginx"  # 把请求会转给下面的服务，下面的服务一定要能处理这个路径，不能处理就是404
        backend:
          service:
            name: nginx-demo  ## java，比如使用路径重写，去掉前缀nginx
            port:
              number: 8000
```

##### 2、路径重写

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress  
metadata:
  annotations: 
    nginx.ingress.kubernetes.io/rewrite-target: /$2 #开启重写
  name: ingress-host-bar
spec:
  ingressClassName: nginx
  rules:
  - host: "hello.atguigu.com"
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: hello-server
            port:
              number: 8000
  - host: "demo.atguigu.com"
    http:
      paths:
      - pathType: Prefix
        path: "/nginx(/|$)(.*)"  # 把请求会转给下面的服务，下面的服务一定要能处理这个路径，不能处理就是404
        backend:
          service:
            name: nginx-demo  ## java，比如使用路径重写，去掉前缀nginx
            port:
              number: 8000
```

##### 3、流量限制

```yaml 
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-limit-rate
  annotations:
    nginx.ingress.kubernetes.io/limit-rps: "1" #限流
spec:
  ingressClassName: nginx
  rules:
  - host: "haha.atguigu.com"
    http:
      paths:
      - pathType: Exact
        path: "/"
        backend:
          service:
            name: nginx-demo
            port:
              number: 8000
```

### 存储抽象

#### 环境准备

##### 1、所有节点

```bash
#所有机器安装
yum install -y nfs-utils
```

##### 2、主节点

```bash
#nfs主节点
echo "/nfs/data/ *(insecure,rw,sync,no_root_squash)" > /etc/exports

mkdir -p /nfs/data
systemctl enable rpcbind --now
systemctl enable nfs-server --now
#配置生效
exportfs -r
```

##### 3、从节点

```bash
showmount -e 172.31.0.4

#执行以下命令挂载 nfs 服务器上的共享目录到本机路径 /root/nfsmount
mkdir -p /nfs/data

mount -t nfs 172.31.0.4:/nfs/data /nfs/data
# 写入一个测试文件
echo "hello nfs server" > /nfs/data/test.txt
```

##### 4、原生方式数据挂载

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-pv-demo
  name: nginx-pv-demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-pv-demo
  template:
    metadata:
      labels:
        app: nginx-pv-demo
    spec:
      containers:
      - image: nginx
        name: nginx
        volumeMounts:
        - name: html
          mountPath: /usr/share/nginx/html
      volumes:
        - name: html
          nfs:
            server: 172.31.0.4
            path: /nfs/data/nginx-pv
```

#### PV&PVC

> *[PV：持久卷（Persistent Volume）](https://kubernetes.io/zh/docs/concepts/storage/volumes/#portworxvolume)，将应用需要持久化的数据保存到指定位置*
>
> *[PVC：持久卷申明（**Persistent Volume Claim**）](https://kubernetes.io/zh/docs/concepts/storage/volumes/#persistentvolumeclaim)，申明需要使用的持久卷规格*

##### 创建pv池

静态供应

```shell
#nfs主节点
mkdir -p /nfs/data/01
mkdir -p /nfs/data/02
mkdir -p /nfs/data/03
```

创建PV

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv01-10m
spec:
  capacity:
    storage: 10M
  accessModes:
    - ReadWriteMany
  storageClassName: nfs
  nfs:
    path: /nfs/data/01
    server: 172.31.0.4
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv02-1gi
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  storageClassName: nfs
  nfs:
    path: /nfs/data/02
    server: 172.31.0.4
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv03-3gi
spec:
  capacity:
    storage: 3Gi
  accessModes:
    - ReadWriteMany
  storageClassName: nfs
  nfs:
    path: /nfs/data/03
    server: 172.31.0.4
```

##### PVC创建与绑定

创建PVC

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: nginx-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 200Mi
  storageClassName: nfs
```

创建Pod绑定PVC

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deploy-pvc
  name: nginx-deploy-pvc
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-deploy-pvc
  template:
    metadata:
      labels:
        app: nginx-deploy-pvc
    spec:
      containers:
      - image: nginx
        name: nginx
        volumeMounts:
        - name: html
          mountPath: /usr/share/nginx/html
      volumes:
        - name: html
          persistentVolumeClaim:
            claimName: nginx-pvc
```

#### [ConfigMap](https://kubernetes.io/zh/docs/concepts/storage/volumes/#configmap)
> 抽取应用配置，并且可以自动更新

##### redis示例

###### 1、把之前的配置文件创建为配置集

```shell
# 创建配置，redis保存到k8s的etcd；
kubectl create cm redis-conf --from-file=redis.conf
```

```yaml
apiVersion: v1
data:    #data是所有真正的数据，key：默认是文件名   value：配置文件的内容
  redis.conf: |
    appendonly yes
kind: ConfigMap
metadata:
  name: redis-conf
  namespace: default
```

###### 2. pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
  - name: redis
    image: redis
    command:
      - redis-server
      - "/redis-master/redis.conf"  #指的是redis容器内部的位置
    ports:
    - containerPort: 6379
    volumeMounts:
    - mountPath: /data
      name: data
    - mountPath: /redis-master
      name: config
  volumes:
    - name: data
      emptyDir: {}
    - name: config
      configMap:
        name: redis-conf
        items:
        - key: redis.conf
          path: redis.conf
```

###### 3、检查默认配置

```bash
kubectl exec -it redis -- redis-cli

127.0.0.1:6379> CONFIG GET appendonly
127.0.0.1:6379> CONFIG GET requirepass
```

###### 4、修改ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: example-redis-config
data:
  redis-config: |
    maxmemory 2mb
    maxmemory-policy allkeys-lru 
```

###### 5、检查配置是否更新

```bash
kubectl exec -it redis -- redis-cli

127.0.0.1:6379> CONFIG GET maxmemory
127.0.0.1:6379> CONFIG GET maxmemory-policy
```

配置值未更改，因为需要重新启动 Pod 才能从关联的 ConfigMap 中获取更新的值。

原因：我们的Pod部署的中间件自己本身没有热更新能力

#### [Secret](https://kubernetes.io/zh/docs/concepts/storage/volumes/#secret)

Secret 对象类型用来保存敏感信息，例如密码、OAuth 令牌和 SSH 密钥。 将这些信息放在 secret 中比放在 [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) 的定义或者 [容器镜像](https://kubernetes.io/zh/docs/reference/glossary/?all=true#term-image) 中来说更加安全和灵活。

加密存到etcd

```shell
kubectl create secret docker-registry leifengyang-docker \
--docker-username=leifengyang \
--docker-password=Lfy123456 \
--docker-email=534096094@qq.com

##命令格式
kubectl create secret docker-registry regcred \
  --docker-server=<你的镜像仓库服务器> \
  --docker-username=<你的用户名> \
  --docker-password=<你的密码> \
  --docker-email=<你的邮箱地址>
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  name: private-nginx
spec:
  containers:
  - name: private-nginx
    image: leifengyang/guignginx:v1.0
  imagePullSecrets:
  - name: leifengyang-docker
```











## Helm

Kubernetes的包管理工具



* helm：一个命令行客户端工具，主要用于 Kubernetes 应用 chart的创建、打包、发布和管理。
* Chart：应用描述，一系列用于描述 k8s 资源相关文件的集合。
* Release：基于 Chart 的部署实体，一个 chart 被 Helm 运行后将会生成对应的一个release；将在 k8s 中创建出真实运行的资源对象。



https://github.com/helm/helm/releases

解压移动到/usr/bin/目录即可。

```shell
wget https://get.helm.sh/helm-vv3.2.1-linux-amd64.tar.gz
tar zxvf helm-v3.2.1-linux-amd64.tar.gz
mv linux-amd64/helm /usr/bin/

```

| 命令       | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| create     | create chart                                                 |
| dependency | 管理chart依赖                                                |
| get        | 下载一个 release。可用子命令：all、hooks、manifest、notes、values |
| history    | history                                                      |
| install    | inatall chart                                                |
| list       | 列出release                                                  |
| package    | 将 chart 目录打包到 chart 存档文件中                         |
| pull       | 从远程仓库中下载 chart 并解压到本地  helm pull stable/mysql --untar |
| repo       | 添加，列出，移除，更新和索引 chart 仓库。可用子命令：add、index、 list、remove、update |
| rollback   | rollback                                                     |
| search     | search chart  可用子命令：hub、repo                          |
| show       | 查看 chart 详细信息。可用子命令：all、chart、readme、values  |
| status     | 显示已命名版本的状态                                         |
| template   | 本地呈现模板                                                 |
| uninstall  | uninstall                                                    |
| upgrade    | 更新                                                         |
| version    | version                                                      |

配置国内chart仓库

* 微软仓库（http://mirror.azure.cn/kubernetes/charts/）这个仓库推荐，基本上官网有的 chart 这里都有。
* 阿里云仓库（https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts ）
* 官方仓库（https://hub.kubeapps.com/charts/incubator）

```shell
#添加
helm repo add stable http://mirror.azure.cn/kubernetes/charts
helm repo add aliyun https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
helm repo update

#查看配置的存储库
helm repo list
helm search repo stable

helm repo remove aliyun#删除
```

### helm基本使用

```shell
#查找 chart
helm search repo weave

#查看 chrt信息
helm show chart stable/mysql

#安装
helm install ui stable/weave-scope

#查看发布状态
helm list
helm status ui
```

#### 安装前自定义chart 配置选项

* --values（或-f）：指定带有覆盖的YAML 文件。这可以多次指定，最右边的文件优先
* --set：在命令行上指定替代。如果两者都用，--set 优先级高

--values 使用，先将修改的变量写到一个文件中

```shell
helm install db --set persistence.storageClass="managed-nfs-storage" stable/mysql
```

该helm install 命令可以从多个来源安装：

* chart 存储库
* 本地chart 存档（helm install foo-0.1.1.tgz）
* chart 目录（helm install path/to/foo）
* 完整的URL（helm install https://example.com/charts/foo-1.2.3.tgz）

#### 开发自己的chart

1、先创建模板
2、修改Chart.yaml，Values.yaml，添加常用的变量
3、在templates 目录下创建部署镜像所需要的yaml 文件，并变量引用yaml 里经常变动的字段

```shell
#构建一个Helm Chart
helm create mychart

Chart.yaml：用于描述这个 Chart的基本信息，包括名字、描述信息以及版本等。
values.yaml ：用于存储 templates 目录中模板文件中用到变量的值。
Templates： 目录里面存放所有 yaml 模板文件。
charts：目录里存放这个 chart 依赖的所有子 chart。
NOTES.txt ：用于介绍 Chart帮助信息， helm install部署后展示给用户。例如：如何使用这个Chart、列出缺省的设置等。
_helpers.tpl：放置模板助手的地方，可以在整个 chart 中重复使用

#部署
helm install web mychart/
#打包
helm package mychart/
#升级
helm upgrade web1 mychart/
```

### helm模板

helm最核心的就是模板，即模板化的 K8S manifests文件

本质上就是一个 Go的 template模板。Helm在 Go template模板的基础上，还会增加很多东西。如一些自定义的元数据信息、扩展的库以及一些类似于编程形式的工作流，例如条件语句、管道等等

```shell
#values.yaml中定义变量
replicas: 3

#templates/deployment.yaml中使用
{{ .Values.replicas }}
```

#### 调试

```shell
#Helm也提供了--dry-run --debug调试参数，帮助你验证模板正确性
helm install web --dry-run nginx/
```

#### 内置对象

Release.Name 			release 名字
Release.Namespace release 命名空间
Release.Service 		release 服务的名称
Release.Revision 	  release 修订版本号，从1 开始累加

#### Values

为Chart 模板提供值

* chart 包中的values.yaml 文件
* 父chart 包的values.yaml 文件
* 通过helm install 或者helm upgrade 的-f 或者--values 参数传入的自定义的yaml 文件
* 通过--set 参数传入的值



chart 的values.yaml 提供的值可以被用户提供的values 文件覆盖，而该文件同样可以被--set 提供的参数所覆盖。

```shell
helm upgrade web --set replicas=5 nginx/
helm history web
```

#### 升级、回滚和删除

```shell
helm upgrade --set imageTag=1.17 web nginx
helm upgrade -f values.yaml web nginx

#回滚到某版本
helm rollback web 1

#卸载
helm uninstall web

#查看历史版本配置信息
helm get all --revision 1 web
```

#### 管道与函数

```yaml
{{ quote .Values.label.app }} #quote函数 从.Values 中读取的值变成字符串
{{ .Values.name | default "nginx" }} #指定默认值

#缩进：
{{ .Values.resources | indent 12 }}
#大写：
{{ upper .Values.resources }}
#首字母大写：
{{ title .Values.resources }}
```

#### 流程控制

* if/else 条件块
* with 指定范围
* range 循环块

```yaml
{{ if PIPELINE }}
# Do something
{{ elseif OTHER PIPELINE }}
# Do something else
{{ else }}
# Default case
{{ end }}

{{ if eq .Values.devops "k8s" }}
devops: 123
{{ else }}
devops: 456
{{ end }}

#eq 运算符判断是否相等，除此之外，还支持ne、lt、gt、and、or 等运算符 注意数据类型

#使用{{- if ...}} 的方式消除此空行
{{- ifeq .Values.env.hello "world" }}
- name: hello
value: 123
{{- end }}

#条件判断就是判断条件是否为真，如果值为以下几种情况则为false：
# 一个布尔类型的false
# 一个数字零
# 一个空的字符串
# 一个空的集合（ map、slice、tuple、dict、array）
#除了上面的这些情况外，其他所有条件都为真。
```

```shell
#当前的作用域就在当前循环内，这个.引用的当前读取的元素。
{{- range .Values.test }}
{{ . }}
{{- end }}

test:
- 1
- 2
- 3
```

```yaml
#with ：控制变量作用域
{{ with PIPELINE }}
# restricted scope
{{ end }}



{{- with .Values.nodeSelector }}
nodeSelector:
team: {{ .team }}
gpu: {{ .gpu }}
{{- end }}


nodeSelector:
team: a
gpu: yes

#优化后
{{- with .Values.nodeSelector }}
nodeSelector:
{{- toYaml . | nindent 8 }}
{{- end }}
```

增加了一个{{- with .Values.nodeSelector}} xxx {{- end }}的一个块，这样的话就可以在当前的块里面直接引用.team 和.gpu 了。
with 是一个循环构造。使用.Values.nodeSelector 中的值：将其转换为Yaml。toYaml 之后的点是循环中.Values.nodeSelector 的当前值

#### 变量

变量，在模板中，使用变量的场合不多，但我们将看到如何使用它来简化代码，并更好地利用with 和range。

```yaml
#with 中不能使用内置对象 
#with 语句块内不能再.Release.Name 对象，否则报错
{{- with .Values.label }}
project: {{ .project }}
app: {{ .app }}
release: {{ .Release.Name }}
{{- end }}

#上面会出错
{{- $releaseName := .Release.Name -}}
{{- with .Values.label }}
project: {{ .project }}
app: {{ .app }}
release: {{ $releaseName }}
# 或者可以使用$符号,引入全局命名空间
release: {{ $.Release.Name }}
{{- end }}
```

#### 命名模板

需要复用代码的地方用

使用define 定义，template 引入，在templates 目录中默认下划线开头的文件为公共模板(helpers.tpl)

```yaml
cat _helpers.tpl
{{- define "demo.fullname" -}}
{{- .Chart.Name -}}-{{ .Release.Name }}
{{- end -}}

{{ template"demo.fullname" . }}
#template 指令是将一个模板包含在另一个模板中的方法。但是，template 函数不能用于Go 模板管道。为了解决该问题，增加include 功能

cat _helpers.tpl
{{- define "demo.labels" -}}
app: {{ template"demo.fullname" . }}
chart: "{{ .Chart.Name }}-{{ .Chart.Version }}"
release: "{{ .Release.Name }}"
{{- end -}}

{{ include"demo.fullname" . }}
{{- include "demo.labels" . | nindent 4 }}
```



## 部署性能监控平台

开源软件cAdvisor（Container Advisor）用于监控所在节点的容器运行状态，当前已经被默认集成到kubelet 组件内，默认使用tcp 4194 端口。在大规模容器集群，一般使用Heapster+Influxdb+Grafana 平台实现集群性能数据的采集，存储与展示

![image-20211027133945385](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20211027133945385.png)

Heapster：集群中各node 节点的cAdvisor 的数据采集汇聚系统，通过调用node 上kubelet 的api，再通过kubelet 调用cAdvisor 的api 来采集所在节点上所有容器的性能数据。Heapster 对性能数据进行聚合，并将结果保存到后端存储系统，heapster 支持多种后端存储系统，如memory，Influxdb 等。

Influxdb：分布式时序数据库（每条记录有带有时间戳属性），主要用于实时数据采集，时间跟踪记录，存储时间图表，原始数据等。Influxdb 提供rest api 用于数据的存储与查询。

Grafana：通过dashboard 将Influxdb 中的时序数据展现成图表或曲线等形式，便于查看集群运行状态。



### 部署

prometheus (http抓取数据)+ grafana (数据展示可视化)

#### prometheus

```powershell
kubectl create -f node-exporter.yaml
```

node-exporter.yaml

```yaml
---
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: node-exporter
  namespace: kube-system
  labels:
    k8s-app: node-exporter
spec:
  template:
    metadata:
      labels:
        k8s-app: node-exporter
    spec:
      containers:
      - image: prom/node-exporter
        name: node-exporter
        ports:
        - containerPort: 9100
          protocol: TCP
          name: http
---
apiVersion: v1
kind: Service
metadata:
  labels:
    k8s-app: node-exporter
  name: node-exporter
  namespace: kube-system
spec:
  ports:
  - name: http
    port: 9100
    nodePort: 31672
    protocol: TCP
  type: NodePort
  selector:
    k8s-app: node-exporter

```

##### configmap.yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: kube-system
data:
  prometheus.yml: |
    global:
      scrape_interval:     15s
      evaluation_interval: 15s
    scrape_configs:

    - job_name: 'kubernetes-apiservers'
      kubernetes_sd_configs:
      - role: endpoints
      scheme: https
      tls_config:
        ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
      bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
      relabel_configs:
      - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_service_name, __meta_kubernetes_endpoint_port_name]
        action: keep
        regex: default;kubernetes;https

    - job_name: 'kubernetes-nodes'
      kubernetes_sd_configs:
      - role: node
      scheme: https
      tls_config:
        ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
      bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
      relabel_configs:
      - action: labelmap
        regex: __meta_kubernetes_node_label_(.+)
      - target_label: __address__
        replacement: kubernetes.default.svc:443
      - source_labels: [__meta_kubernetes_node_name]
        regex: (.+)
        target_label: __metrics_path__
        replacement: /api/v1/nodes/${1}/proxy/metrics

    - job_name: 'kubernetes-cadvisor'
      kubernetes_sd_configs:
      - role: node
      scheme: https
      tls_config:
        ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
      bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
      relabel_configs:
      - action: labelmap
        regex: __meta_kubernetes_node_label_(.+)
      - target_label: __address__
        replacement: kubernetes.default.svc:443
      - source_labels: [__meta_kubernetes_node_name]
        regex: (.+)
        target_label: __metrics_path__
        replacement: /api/v1/nodes/${1}/proxy/metrics/cadvisor

    - job_name: 'kubernetes-service-endpoints'
      kubernetes_sd_configs:
      - role: endpoints
      relabel_configs:
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scheme]
        action: replace
        target_label: __scheme__
        regex: (https?)
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__address__, __meta_kubernetes_service_annotation_prometheus_io_port]
        action: replace
        target_label: __address__
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
      - action: labelmap
        regex: __meta_kubernetes_service_label_(.+)
      - source_labels: [__meta_kubernetes_namespace]
        action: replace
        target_label: kubernetes_namespace
      - source_labels: [__meta_kubernetes_service_name]
        action: replace
        target_label: kubernetes_name

    - job_name: 'kubernetes-services'
      kubernetes_sd_configs:
      - role: service
      metrics_path: /probe
      params:
        module: [http_2xx]
      relabel_configs:
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_probe]
        action: keep
        regex: true
      - source_labels: [__address__]
        target_label: __param_target
      - target_label: __address__
        replacement: blackbox-exporter.example.com:9115
      - source_labels: [__param_target]
        target_label: instance
      - action: labelmap
        regex: __meta_kubernetes_service_label_(.+)
      - source_labels: [__meta_kubernetes_namespace]
        target_label: kubernetes_namespace
      - source_labels: [__meta_kubernetes_service_name]
        target_label: kubernetes_name

    - job_name: 'kubernetes-ingresses'
      kubernetes_sd_configs:
      - role: ingress
      relabel_configs:
      - source_labels: [__meta_kubernetes_ingress_annotation_prometheus_io_probe]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_ingress_scheme,__address__,__meta_kubernetes_ingress_path]
        regex: (.+);(.+);(.+)
        replacement: ${1}://${2}${3}
        target_label: __param_target
      - target_label: __address__
        replacement: blackbox-exporter.example.com:9115
      - source_labels: [__param_target]
        target_label: instance
      - action: labelmap
        regex: __meta_kubernetes_ingress_label_(.+)
      - source_labels: [__meta_kubernetes_namespace]
        target_label: kubernetes_namespace
      - source_labels: [__meta_kubernetes_ingress_name]
        target_label: kubernetes_name

    - job_name: 'kubernetes-pods'
      kubernetes_sd_configs:
      - role: pod
      relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
        action: replace
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
        target_label: __address__
      - action: labelmap
        regex: __meta_kubernetes_pod_label_(.+)
      - source_labels: [__meta_kubernetes_namespace]
        action: replace
        target_label: kubernetes_namespace
      - source_labels: [__meta_kubernetes_pod_name]
        action: replace
        target_label: kubernetes_pod_name

```

##### prometheus.svc.yml

```yaml
---
kind: Service
apiVersion: v1
metadata:
  labels:
    app: prometheus
  name: prometheus
  namespace: kube-system
spec:
  type: NodePort
  ports:
  - port: 9090
    targetPort: 9090
    nodePort: 30003
  selector:
    app: prometheus

```

##### prometheus.deploy.yml

```yaml
---
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  labels:
    name: prometheus-deployment
  name: prometheus
  namespace: kube-system
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      containers:
      - image: prom/prometheus:v2.0.0
        name: prometheus
        command:
        - "/bin/prometheus"
        args:
        - "--config.file=/etc/prometheus/prometheus.yml"
        - "--storage.tsdb.path=/prometheus"
        - "--storage.tsdb.retention=24h"
        ports:
        - containerPort: 9090
          protocol: TCP
        volumeMounts:
        - mountPath: "/prometheus"
          name: data
        - mountPath: "/etc/prometheus"
          name: config-volume
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
          limits:
            cpu: 500m
            memory: 2500Mi
      serviceAccountName: prometheus    
      volumes:
      - name: data
        emptyDir: {}
      - name: config-volume
        configMap:
          name: prometheus-config   

```

#### grafana 

##### grafana-svc.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: grafana
  namespace: kube-system
  labels:
    app: grafana
    component: core
spec:
  type: NodePort
  ports:
    - port: 3000
  selector:
    app: grafana
    component: core

```

##### grafana-ing.yaml

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
   name: grafana
   namespace: kube-system
spec:
   rules:
   - host: k8s.grafana
     http:
       paths:
       - path: /
         backend:
          serviceName: grafana
          servicePort: 3000

```

##### grafana-deploy.yaml

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: grafana-core
  namespace: kube-system
  labels:
    app: grafana
    component: core
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: grafana
        component: core
    spec:
      containers:
      - image: grafana/grafana:4.2.0
        name: grafana-core
        imagePullPolicy: IfNotPresent
        # env:
        resources:
          # keep request = limit to keep this container in guaranteed class
          limits:
            cpu: 100m
            memory: 100Mi
          requests:
            cpu: 100m
            memory: 100Mi
        env:
          # The following env variables set up basic auth twith the default admin user and admin password.
          - name: GF_AUTH_BASIC_ENABLED
            value: "true"
          - name: GF_AUTH_ANONYMOUS_ENABLED
            value: "false"
          # - name: GF_AUTH_ANONYMOUS_ORG_ROLE
          #   value: Admin
          # does not really work, because of template variables in exported dashboards:
          # - name: GF_DASHBOARDS_JSON_ENABLED
          #   value: "true"
        readinessProbe:
          httpGet:
            path: /login
            port: 3000
          # initialDelaySeconds: 30
          # timeoutSeconds: 1
        volumeMounts:
        - name: grafana-persistent-storage
          mountPath: /var
      volumes:
      - name: grafana-persistent-storage
        emptyDir: {}

```

配置数据源, 导入模板(315)









































































































































































# 云原生

https://www.yuque.com/leifengyang/oncloud/vfvmcd

**基础概念**

- 云服务器作为应用的最终载体
- VPC为所有云服务器提供网络隔离

- 所有云服务器都是绑定某个私有网络
- 安全组控制每个服务器的防火墙规则

- 公网IP使得资源可访问
- 端口转发的方式访问到具体服务器



## DevOps

DevOps **是一系列做法和工具**，可以使 IT 和软件开发团队之间的**流程实现自动化**。其中，随着敏捷软件开发日趋流行，**持续集成 (CI)** 和**持续交付 (CD)** 已经成为该领域一个理想的解决方案。在 CI/CD 工作流中，每次集成都通过自动化构建来验证，包括编码、发布和测试，从而帮助开发者提前发现集成错误，团队也可以快速、安全、可靠地将内部软件交付到生产环境。

![image-20211104150310041](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20211104150310041.png)





dockerfile

```dockerfile
FROM java:8
EXPOSE 8080

VOLUME /tmp
ADD target/*.jar /app.jar
RUN bash -C 'touch /app. jar'
ENTRYPOINT ["java" ,"-jar","/app.jar","--spring.profiles.active=prod"]
```

```dockerfile
FROM nginx
MAINTAINER leifengyang

ADD dist.tar.gz /usr/share/nginx/html
EXPOSE 80
ENTRYPOINT nginx -g "daemon off;"
```

```dockerfile
FROM nginx
MAINTAINER leifengyang
ADD html.tar.gz /usr/share/nginx/html
ADD conf.tar.gz /etc/nginx
EXPOSE 80
ENTRYPOINT nginx -g "daemon off;"
```

jinkens

```shell
#!/bin/bash
mvn clean install -pl /iap-service/mes-me/mes-me-provider -am -amd -Dmaven.test.skip=true
cd iap-service/mes-me/mes-me-provider/target
BUILD_ID=dontKillMe
pid=`ps -ef | grep mes-me-provider-1.0.jar | grep -v grep | awk '{print $2}'`
echo "停止旧应用进程id：$pid"
if [ -n "$pid" ]
then
kill -9 $pid
fi
echo "启动程序"
nohup java -jar -DNACOS_HOST=192.168.2.201 -Xms512M -Xmx512M mes-me-provider-1.0.jar &
```



## 集群

* 高可用
* 备份容灾
* 压力分担
* 突破数据量上限
* 可扩展性



集群必须拥有以下两大能力

负载均衡

错误恢复



集群与分布式的区别

相同点：

​	都是需要有很多节点服务器通过网络协同工作完成整体的任务目标。

不同点：

​	分布式是指将业务系统进行拆分，即分布式的每一个节点都是实现不同的功能。而集群每个节点做的是同一件事情。

### 集群类型

#### 主从

主从复制, 同步方式   (mysql)

主从调度, 控制方式   (k8s)

#### 分片式

数据分片存储 , 分片之间备份   (redis)

#### 选主式

容灾选主

调度选主



### mysql

* mysql-mmm 主主复制管理器   主要用来监控mysql主主复制并作失败转移

  ![image-20211103114221634](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20211103114221634.png)

* MHA (Master High Availability) 成熟方案   MySQL高可用性环境下故障切换和主从提升的高可用软件

* InnoDB Cluster 官方方案

  ![image-20211103133052055](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20211103133052055.png)
  
* mycat/ShardingSphere 分库分表

  ![image-20211103133311382](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20211103133311382.png)

### redis

* 分区

* 代理分区

* redis-Cluster 官方 高可用   推荐6个 三主三从  槽slot

  * 批量操作 支持有限
  * 事务 支持有限     使用lua解决
  * key作为最小粒度
  * 不支持多数据库空间
  * 复制结构只能有一层
  * 命令大多重定向 , 耗时多

* 哨兵模式

  

### ES

* 主从 分片 副本

yellow

red

green



脑裂问题   减少误判   选举触发(主要)

### RabbitMQ

内存节点, 磁盘节点  至少一个磁盘节点

* 普通模式
* 镜像模式  高可用