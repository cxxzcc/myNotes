# wsl

https://learn.microsoft.com/en-us/windows/wsl/



https://blog.csdn.net/qq_43479892/article/details/123591840?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-123591840-blog-122594928.pc_relevant_recovery_v2&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-123591840-blog-122594928.pc_relevant_recovery_v2&utm_relevant_index=1

```shell
# Settings apply across all Linux distros running on WSL 2
[wsl2]
 
# Limits VM memory to use no more than 4 GB, this can be set as whole numbers using GB or MB
memory=2GB 
 
# Sets the VM to use two virtual processors
processors=4
 
# Specify a custom Linux kernel to use with your installed distros. The default kernel used can be found at https://github.com/microsoft/WSL2-Linux-Kernel
# kernel=C:\\temp\\myCustomKernel
 
# Sets additional kernel parameters, in this case enabling older Linux base images such as Centos 6
# kernelCommandLine = vsyscall=emulate
 
# Sets amount of swap storage space to 8GB, default is 25% of available RAM
swap=4GB
 
# Sets swapfile path location, default is %USERPROFILE%\AppData\Local\Temp\swap.vhdx
# swapfile=C:\\temp\\wsl-swap.vhdx
 
# Disable page reporting so WSL retains all allocated memory claimed from Windows and releases none back when free
# pageReporting=false
 
# Turn off default connection to bind WSL 2 localhost to Windows localhost
localhostforwarding=true
 
# Disables nested virtualization
# nestedVirtualization=false
 
# Turns on output console showing contents of dmesg when opening a WSL 2 distro for debugging
# debugConsole=true
```

## wsl管理工具
LxRunOffline
wsl2-distro-manager




# VM

DHCP服务开启 获取ip

ifcfg-ens33 文件

vm 编辑 虚拟网络编辑 nat 设置 =GATEWAY

DNS1 = ipcofig/all 以太网ip

```shell
TYPE="Ethernet"
BOOTPROTO="static"
IPADDR=192.168.217.130
NETMASK=255.255.255.0
GATEWAY=192.168.217.2
DNS1=192.168.99.1
DEFROUTE="yes"
PEERDNS="yes"
PEERROUTES="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_PEERDNS="yes"
IPV6_PEERROUTES="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME="ens33"
UUID="f50c1acb-829e-4c6c-a9d7-3d9c5e6ea0d5"
DEVICE="ens33"
ONBOOT="yes"
```

## vagrant

[镜像](https://app.vagrantup.com/boxes/search)

```shell
//初始化
vagrant init  镜像
//启动
vagrant up
//连接虚拟机
vagrant ssh
//退出
exit
//重启
vagrant reload

vagrant box list 查看目前已有的box 
vagrant box add 新增加一个box 
vagrant box remove 删除指定box 
vagrant init 初始化配置
vagrantfile vagrant up 启动虚拟机 
vagrant ssh ssh登录虚拟机 
vagrant suspend 挂起虚拟机 
vagrant reload 重启虚拟机 
vagrant halt 关闭虚拟机 
vagrant status 查看虚拟机状态 
vagrant destroy 删除虚拟机
```

# VCS

## Git

[廖雪峰git](https://www.liaoxuefeng.com/wiki/896043488029600/896067074338496)

[阮一峰git](http://www.ruanyifeng.com/blog/2014/06/git_remote.html)

```shell
git init //初始化
git config --global user.name ""  //修改作者的名字和邮箱  未commit
git config --global user.email ""


git commit --amend --author="Author Name <email@address.com>" //改已commit最近一次commit
git commit --amend --reset-author --no-edit   //已改过git config中的y邮箱 改已commit


git add ./文件路径         //存储到git仓库第一步
git add ./          //全部
git commit -m "说明"       //存储到git仓库第二步
git commit --all -m "说明"

git status    //比较工作区和git仓库的代码
git log
git log --oneline //简版log
git reflog        //版本切换log

git reset --hard Head~0     //回退到上一次
git reset --hard Head~1      //回退到上上次
git reset --hard [版本号]
git reflog     //看到每一次切换版本的记录:可以看到所有提交的版本号

//分支      master 主分支
git branch  aaa //创建分支aaa
git checkout aaa //切换分支
git branch    // 查看分支
git merge aaa //合并分支 与aaa合并


git push [地址] master //提交到GitHub   加-u之后只需写git push   
git pull [地址] master //取GitHub 需要先init
git clone [地址]   //得到GitHub的数据 多次执行会覆盖

//ssh方式
// 公钥 私钥
//生成公钥私钥
ssh-keygen -t rsa -C "邮箱"
```

### Gogs
极易搭建的自助 Git 服务
```shell
docker pull gogs/gogs
docker run -di --name=gogs -p 10022:22 -p 3000:3000 -v /var/gogsdata:/data gogs/gogs
```




# fore-end tools

## NPM

node.js package management

全球最大的模块生态系统，里面所有的模块都是开源免费的；也是Node.js的包管理工具。
https://www.npmjs.com/ 

安装
	全局安装 -g：
    全局安装的包位于Node.js环境的node_modules目录下，全局安装的包一般用于命令行工具
    本地安装：
    本地安装的包在当前目录下的node_modules里面，本地安装的包一般用于实际的开发工作

命令

```shell
安装包（如果没有指定版本号，那么安装最新版本）
npm install -g 包名称 (全局安装)
npm install 包名称 (本地安装)

安装包的时候可以指定版本
npm install -g 包名称@版本号

卸载包
npm uninstall -g 包名

更新包（更新到最新版本）
npm update -g 包名

--save 向生产环境添加依赖 dependencies
--save-dev 向开发环境添加依赖 DevDependencies

```

## yarn
js包管理工具
```shell
安装yarn工具：
npm install -g yarn

初始化包
npm init
yarn init

安装包
npm install xxx --save
yarn add xxx

移除包
npm uninstall xxx
yarn remove xxx

更新包
npm update xxx
yarn upgrade xxx

安装开发依赖的包
npm install xxx --save-dev
yarn add xxx --dev

全局安装
npm install -g xxx
yarn global add xxx

设置下载镜像的地址
npm config set registry url
yarn config set registry url

安装所有依赖
npm install
yarn install

执行包
npm run
yarn run

```
## gulp

流式构建工具
[官网](http://www.gulpjs.com) [中文网](http://www.gulpjs.com.cn)

5个核心方法
	gulp.task('任务名',function(){}) // 创建任务。
	gulp.src('./*.css') 指定想要处理的文件
	gulp.dest() // 指定最终处理后的文件的存放路径
	gulp.watch() // 自动的监视文件的变化，然后执行相应任务。
	gulp.run('任务名')，直接执行相应的任务。

```shell
npm install gulp-cli -g //安装
npm install gulp --save //项目安装
```
还需要在当前项目中新建一个文件: gulpfile.js
```js
 var gulp =  require('gulp');

    // 创建任务
    // 第一个参数: 任务名
    // 第二个参数: 回调函数,当我们执行任务时就会执行这个函数
    gulp.task('test', function(){
      console.log(123)
})
```
执行任务: gulp 任务名
	示例: gulp test











