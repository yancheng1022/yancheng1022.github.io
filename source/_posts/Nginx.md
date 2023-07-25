---
title: nginx
date: 2023/07/15
categories:
  - coding
tags:
  - nginx
abbrlink: 58122
---
# 1、nginx简介
## 1.1、nginx概述
Nginx (“engine x”) 是一个高性能的 HTTP 和 反向代理服务器，特点是占有内存少，并发能力强，能经受高负载的考验,有报告表明能支持高达 50,000 个并发连接数 。
## 1.2、正向代理
nginx不仅能做反向代理，实现负载均衡，还能用作正向代理来进行上网等功能
正向代理：个位于客户端和原始服务器之间的服务器，为了从原始服务器取得内容，客户端向代理发送一个请求并制定目标（原始服务器），然后代理向原始服务器转发请求并将获得的内容返回给客户端，客户端才能使用正向代理。我们平时说的代理就是指正向代理【**代理客户端，服务端不知道实际发起请求的客户端**】
> 例子：A向C借钱，由于一些情况不能直接向C借钱，于是A想了一个办法，他让B去向C借钱，这样B就代替A向C借钱，A就得到了C的钱，C并不知道A的存在，B就充当了A的代理人的角色

## 1.3、反向代理
反向代理，以代理服务器来接受internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给internet上请求的客户端，此时代理服务器对外表现为一个反向代理服务器【**代理服务端，客户端不知道实际提供服务的服务端**】
> 例子：A向B借钱，B没有拿自己的钱，而是悄悄地向C借钱，拿到钱之后再交给A,A以为是B的钱，他并不知道C的存在

## 1.4、负载均衡
增加服务器的数量，然后将请求分发到各个服务器上，将原先请求集中到单个服务器上的情况改为将请求分发到多个服务器上，将负载分发到不同的服务器，也就是我们所说的负载均衡 
## 1.5、动静分离
为了加快网站的解析速度，可以把 动态页面 和 静态页面 由不同的服务器来解析，加快解析速度。降低原来单个服务器的压力
> 静态资源请求（如html、css、图片等）由静态资源服务器处理，动态资源请求（如 jsp页面、servlet程序等）由 tomcat 服务器处理，tomcat 本身是用来处理动态资源的，同时 tomcat 也能处理静态资源，**但是 tomcat 本身处理静态资源的效率并不高**，而且还会带来额外的资源开销。利用 nginx 实现动静分离的架构，能够让 tomcat 专注于处理动态资源，静态资源统一由静态资源服务器处理，从而提升整个服务系统的性能 


# 2、nginx安装

1. **安装pcre依赖**

（1）联网下载 pcre 压缩文件依赖：
```shell
wget http://downloads.sourceforge.net/project/pcre/pcre/8.37/pcre-8.37.tar.gz
```
（2）解压压缩文件
```shell
tar –xvf pcre-8.37.tar.gz
```
（3）编译
```shell
make && make install
```
（4）查看
```shell
pcre-config --version
```

2. **安装安装 openssl 、zlib 、 gcc 依赖**

```shell
yum -y install make zlib zlib-devel gcc-c++ libtool openssl openssl-devel
```

3. **安装nginx**

（1）解压
```shell
tar -xvf nginx-1.12.2.tar.gz
```
（2）进入解压后目录，执行config命令
```shell
./configure
```
（3）编译
```shell
make && make install
```
（4）启动nginx
```shell
# 进入进入目录 /usr/local/nginx/sbin/nginx
./nginx
# 停止
./nginx -s stop
# 重启
./nginx -s reload

```

# 3、nginx配置文件
Nginx配置文件分为三大块：全局块，events块，http块

1. 从配置文件开始到events块开始之前的内容，都属于全局块。在全局块中配置的都是影响Nginx整体运行的配置。比如说：worker(工作进程)的数量，错误日志的位置等
2. events块主要影响nginx服务器与⽤户的⽹络连接，⽐如worker_connections 1024，标识每个 workderprocess进程⽀持的最⼤连接数为1024
3. http块是配置最频繁的部分，虚拟主机的配置，监听端⼝的配置，请求转发、反向代理、负载均衡 等

```shell

#================ 全局快 ==================#
#定义Nginx运行的用户和用户组
user  nginx;
# 工作进程数，一般配置成和CPU数一样
worker_processes  auto;
#全局错误日志定义类型，[ debug | info | notice | warn | error | crit ]
error_log  /var/log/nginx/error.log notice;
#进程pid文件
pid        /var/run/nginx.pid;

#=============== events块 ================#
events {
    # 标识单个woker进程最大并发数
    worker_connections  1024;
}

#=============== http块 =================#
http {
    #文件扩展名与文件类型映射表
    include       /etc/nginx/mime.types;
    #默认文件类型
    default_type  application/octet-stream;
    #日志格式设定
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    #定义本虚拟主机的访问日志
    access_log  /var/log/nginx/access.log  main;
     #开启高效文件传输模式，sendfile指令指定nginx是否调用sendfile函数来输出文件，对于普通应用设为 on，如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络I/O处理速度，降低系统的负载
    sendfile        on;
    #长连接超时时间，单位是秒
    keepalive_timeout  65;
    #包括多个server块，而每个server块就相当于一个虚拟主机
    server {
          listen       80;
          server_name  localhost;
          # 对特定地址进行处理，地址定向
          location / {
              root   html;
              index  index.html index.htm;
          }
  
          error_page   500 502 503 504  /50x.html;
          location = /50x.html {
              root   html;
      }
}
```

# 4、反向代理实现
实现效果：输入 http://www.test.com, 自动跳转到百度首页


```shell
server {
         listen          80;
         server_name     www.test.com;  #你的域名
         location / {
               proxy_pass          http://www.baidu.com/;  #需要反代的域名
               proxy_redirect      off;
               proxy_set_header    X-Real-IP      $remote_addr;
               proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
        }
}

```
# 5、正向代理实现
场景1：从本地无法直接调用第三方接口，因为本地ip不在白名单中的问题
场景2：内网机器访问外网，就需要正向代理，类似VPN
```shell
server {
	listen 8090;
	
	location / {
    # resolver后面填写dns地址，可以多个，将以轮询方式请求
		resolver 218.85.157.99 218.85.152.99;
    # resolver_timeout 解析超时时间
		resolver_timeout 30s;
    # 代理服务器地址（即要请求的地址）
		proxy_pass http://$host$request_uri;
	}
	access_log /data/httplogs/proxy-$host-aceess.log;
}

```
# 6、负载均衡实现
## 6.1、轮询（默认）
每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器 down 掉，能自动剔除
```shell


http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    upstream webservers{
      # weight 多台机器，可以配置权重值，权重高的服务将会优先被访问
      server  192.168.9.134:8081;
      server  192.168.9.134:8082;
    }
 
    server {
        listen       80;
        server_name  localhost;

        location / {
             #转发到负载服务上
            proxy_pass http://webservers/api/;
         }
    }
}
```
## 6.2、weight
weight 代表权重默认为 1,权重越高被分配的客户端越多。指定轮询几率，weight权重大小和访问比率成正比。用于后端服务器性能不均衡的情况。
```shell

    upstream webservers{
      server  192.168.9.134:8081 weight=8;
      server  192.168.9.134:8082 weight=2;
    }
```
## 6.3、ip_hash
每个请求按访问 ip 的 hash 结果分配，这样每个访客固定访问一个后端服务器
> 使用nginx+ip_hash这种策略代理，很好解决了同一用户访问同一个应用，session不共享的问题,实现session共享的问题

```shell
    upstream webservers{
      ip_hash;
      server  192.168.9.134:8081;
      server  192.168.9.134:8082;
    }
```
## 6.4、fair
按后端服务器的响应时间来分配请求，响应时间短的优先分配。
```shell
upstream webservers{
        server 192.168.9.134:8081;
        server 192.168.9.134:8082;
        fair;
}
```
## 6.5、url_hash
按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，后端服务器为缓存时比较有效
> 相同的url会被分配到同一个节点，主要为了提高缓存命中率

```shell
upstream webservers{
    hash &request_uri;
    server 192.168.9.134:8081;
    server 192.168.9.134:8082;
}
```
## 6.6、least_conn
按节点连接数分配，把请求优先分配给连接数少的节点。该策略主要为了解决，各个节点请求处理时间长短不一造成某些节点超负荷的情况
```shell
upstream webservers{
    least_conn;
    server 192.168.9.134:8081;
    server 192.168.9.134:8082;
}
```
# 7、动静分离
利用 nginx 实现动静分离的架构，能够让 tomcat 专注于处理动态资源，静态资源统一由nginx处理，从而提升整个服务系统的性能
```shell
#所有js,css相关的静态资源文件的请求由Nginx处理
location ~.*\.(js|css)$ {
    root    /opt/static-resources; #指定文件路径
    expires     12h; #过期时间为12小时
}
#所有图片等多媒体相关静态资源文件的请求由Nginx处理
location ~.*\.(html|jpg|jpeg|png|bmp|gif|ico|mp3|mid|wma|mp4|swf|flv|rar|zip|txt|doc|ppt|xls|pdf)$ {
    root    /opt/static-resources; #指定文件路径
    expires     7d; #过期时间为7天
}
```
