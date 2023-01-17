# Nginx

高性能的 HTTP 和反向代理的服务器，高并发能力是十分强，有报告表明能支持高达 50,000 个并发连接数。



**正向代理**

需要在客户端配置代理服务器进行指定网站访问

**反向代理** 

暴露的是代理服务器地址，隐藏了真实服务器 IP 地址。

**负载均衡**

将负载分发到不同的服务器

**动静分离**



## 1.1 Nginx 安装

http://nginx.org/

（1）安装 pcre 依赖 

第一步 联网下载 pcre 压缩文件依赖 

```bash
wget http://downloads.sourceforge.net/project/pcre/pcre/8.37/pcre-8.37.tar.gz
```

第二步 解压压缩文件 

```shell
tar –xvf pcre-8.37.tar.gz
```

第三步

```bash
./configure 
```

完成后，回到 pcre 目录下执行 make，最后执行 make install

```bash
pcre-config --version //查看安装版本
```

（2）安装 openssl 、zlib 、 gcc 依赖 

```bash
yum -y install make zlib zlib-devel gcc-c++ libtool openssl openssl-devel
```

（3）安装 nginx 

```bash
//使用命令解压 
./configure 
make && make install
```

进入目录 /usr/local/nginx/sbin/nginx 启动服务

```
./nginx
```

```
查看开放的端口号
firewall-cmd --list-all
设置开放的端口号
firewall-cmd --add-service=http –permanent
firewall-cmd --add-port=80/tcp --permanent
重启防火墙
firewall-cmd –reload
```

### docker

# 



随便启动一个 nginx 实例，只是为了复制出配置

```
docker run -p80:80 --name nginx -d nginx:1.10   
```

将容器内的配置文件拷贝到当前目录 （注意后面有个小点）

```bash
docker container cp nginx:/etc/nginx .  
```

创建nginx文件夹

```shell
mkdir -p /mydata/nginx/html
mkdir -p /mydata/nginx/logs
 #由于拷贝完成后会在config中存在一个nginx文件夹，所以需要将它的内容移动到conf中
 #conf 文件夹下就是原先nginx的配置
mv /mydata/nginx/conf/nginx/* /mydata/nginx/conf/
rm -rf /mydata/nginx/conf/nginx
```

别忘了后面的点

修改文件名称：mv nginx.conf 把这个conf 移动到 /mydata/nginx 下

终止原容器， docker stop nginx

执行命令删除容器：docker rm $Containerid

创建新的 nginx 执行以下命令

```bash
 docker run -p 80:80 --name nginx \
 -v /mydata/nginx/html:/usr/share/nginx/html \
 -v /mydata/nginx/logs:/var/log/nginx \
 -v /mydata/nginx/conf/:/etc/nginx \
 -d nginx:1.10
```







## 1.2 nginx 常用的命令和配置文件

### 1.2.1 nginx 常用的命令

启动

```xml
在/usr/local/nginx/sbin 目录下执行 ./nginx
./nginx -v
```

关闭命令 

```bash
在/usr/local/nginx/sbin 目录下执行 ./nginx -s stop
```

重新加载命令

```bash
在/usr/local/nginx/sbin 目录下执行 ./nginx -s reload
```

### 1.2.2 nginx.conf 配置文件

```
/usr/local/nginx/conf/nginx.conf
```

2、配置文件中的内容
包含三部分内容

1. 全局块：配置服务器整体运行的配置指令

   从配置文件开始到 events 块之间的内容，主要会设置一些影响 nginx 服务器整体运行的配置指令，主要包括配 置运行 Nginx 服务器的用户（组）、允许生成的 worker process 数，进程 PID 存放路径、日志存放路径和类型以 及配置文件的引入等。

   比如 worker_processes 1;处理并发数的配置

2. events 块：影响 Nginx 服务器与用户的网络连接

   events 块涉及的指令主要影响 Nginx 服务器与用户的网络连接，常用的设置包括是否开启对多 work process  下的网络连接进行序列化，是否允许同时接收多个网络连接，选取哪种事件驱动模型来处理连接请求，每个 word  process 可以同时支持的最大连接数等。

   比如 worker_connections 1024; 支持的最大连接数为 1024

   这部分的配置对 Nginx 的性能影响较大，在实际中应该灵活配置。

3. http 块

   这算是 Nginx 服务器配置中最频繁的部分，代理、缓存和日志定义等绝大多数功能和第三方模块的配置都在这里

   还包含两部分：

   1. http 全局块

      http 全局块配置的指令包括文件引入、MIME-TYPE 定义、日志自定义、连接超时时间、单链接请求数上限等

   2. server 块

      这块和虚拟主机有密切关系，虚拟主机从用户角度看，和一台独立的硬件主机是完全一样的，该技术的产生是为了 节省互联网服务器硬件成本。 每个 http 块可以包括多个 server 块，而每个 server 块就相当于一个虚拟主机。 而每个 server 块也分为全局 server 块，以及可以同时包含多个 locaton 块

      1. 全局 server 块 最常见的配置是本虚拟机主机的监听配置和本虚拟主机的名称或 IP 配置。
      2. 、location 块 一个 server 块可以配置多个 location 块

      这块的主要作用是基于 Nginx 服务器接收到的请求字符串（例如 server_name/uri-string），对虚拟主机名称 （也可以是 IP 别名）之外的字符串（例如 前面的 /uri-string）进行匹配，对特定的请求进行处理。地址定向、数据缓 存和应答控制等功能，还有许多第三方模块的配置也在这里进行。

## 1.3 nginx 配置实例-反向代理

./startup.sh 启动 tomcat 服务器

对外开放访问的端口 firewall-cmd --add-port=8080/tcp --permanent firewall-cmd –reload







## 配置demo

### 侦听端口

```nginx
server {
  # Standard HTTP Protocol
  listen 80;

  # Standard HTTPS Protocol
  listen 443 ssl;

  # For http2
  listen 443 ssl http2;

  # Listen on 80 using IPv6
  listen [::]:80;

  # Listen only on using IPv6
  listen [::]:80 ipv6only=on;
}
```

### 访问日志

```nginx
server {
  # Relative or full path to log file
  access_log /path/to/file.log;

  # Turn 'on' or 'off'
  access_log on;
}
```

### 域名

```nginx
server {
  # Listen to yourdomain.com
  server_name yourdomain.com;

  # Listen to multiple domains
  server_name yourdomain.com www.yourdomain.com;

  # Listen to all domains
  server_name *.yourdomain.com;

  # Listen to all top-level domains
  server_name yourdomain.*;

  # Listen to unspecified Hostnames (Listens to IP address itself)
  server_name "";

}
```

### 静态资产

```
server {
  listen 80;
  server_name yourdomain.com;

  location / {
          root /path/to/website;
  } 
}
```

### 重定向

```nginx
server {
  listen 80;
  server_name www.yourdomain.com;
  return 301 http://yourdomain.com$request_uri;
}
server {
  listen 80;
  server_name www.yourdomain.com;

  location /redirect-url {
     return 301 http://otherdomain.com;
  }
}
```

### 反向代理

```
server {
  listen 80;
  server_name yourdomain.com;

  location / {
     proxy_pass http://0.0.0.0:3000;
     # where 0.0.0.0:3000 is your application server (Ex: node.js) bound on 0.0.0.0 listening on port 3000
  }

}
```

### 负载均衡

```nginx
upstream node_js {
  server 0.0.0.0:3000;
  server 0.0.0.0:4000;
  server 123.131.121.122;
}

server {
  listen 80;
  server_name yourdomain.com;

  location / {
     proxy_pass http://node_js;
  }
}
```

### SSL 协议

```nginx
server {
  listen 443 ssl;
  server_name yourdomain.com;

  ssl on;

  ssl_certificate /path/to/cert.pem;
  ssl_certificate_key /path/to/privatekey.pem;

  ssl_stapling on;
  ssl_stapling_verify on;
  ssl_trusted_certificate /path/to/fullchain.pem;

  ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
  ssl_session_timeout 1h;
  ssl_session_cache shared:SSL:50m;
  add_header Strict-Transport-Security max-age=15768000;
}

# Permanent Redirect for HTTP to HTTPS
server {
  listen 80;
  server_name yourdomain.com;
  return 301 https://$host$request_uri;
}
```

## [在线配置](https://www.digitalocean.com/community/tools/nginx?global.app.lang=zhCN)







































