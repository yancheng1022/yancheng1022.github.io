---
title: docker
date: 2024/05/18
categories:
  - coding
tags:
  - 编程基础
  - docker
  - 容器化技术
abbrlink: 49176
---

# 1、概念
## 1.1、基本概念

Docker本身是一个容器运行载体或称之为管理引擎。我们把引用程序和配置依赖打包好形成一个可交付的运行环境，这个打包好的运行环境就是image镜像文件。只有通过这个镜像文件才能生成Docker容器实例

>比较Docker和传统虚拟化方式的不同之处：
 1.传统虚拟机技术是虚拟出一套硬件后，在其上运行一个完整操作系统，在该系统上再运行所需要应用进程；
 2.容器内的应用进程直接运行于宿主的内核，容器内没有自己的内核且也没有进行硬件虚拟。因此容器要比传统虚拟机更为轻便；
 3.每个容器之间互相隔离，每个容器有自己的文件系统，容器之间进程不会相互影响，能区分计算资源
 
## 1.2、Docker组成
![image.png](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/20240515145403.png)

们发现在安装docker时，不仅会安装docker engine与docker cli工具，而且还会安装containerd，这是为什么呢？

Docker最初是一个单体引擎，主要负责容器镜像的制作、上传、拉取及容器的运行及管理。随着容器技术的繁荣发展，为了促进容器技术相关的规范生成和Docker自身项目的发展，Docker将单体引擎拆分为三部分，分别为runC、containerd和dockerd

其中：

- runC主要负责容器的运行和生命周期的管理（即低层运行时）
- containerd主要负责容器镜像的下载和解压等镜像管理功能（即高层运行时）
- dockerd主要负责提供镜像制作、上传等功能同时提供容器存储和网络的映射功能，同时也是Docker服务器端的守护进程，用来响应Docker客户端（命令行CLI工具）发来的各种容器、镜像管理的任务。

Docker公司将runC捐献给了OCI，将containerd捐献给了CNCF，剩下的dockerd作为Docker运行时由Docker公司自己维护。


# 2、Docker的安装

> 关于Docker版本：
> Docker-ce Docker社区版，主要用于个人开发者测试使用，免费版本
> Docker-ee Docker企业版，主要用于为企业开发及应用部署使用，收费版本，免费试用一个月，2020年因国际政治原因曾一度限制中国企业使用。

安装环境：CentOS 7.3+

1、如果之前安装了旧版docker，请先删除。

```shell
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

2、安装仓库
```shell
sudo yum install -y yum-utils

sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

3、安装docker engine

```shell
sudo yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

4、启动docker，运行hello world，查看是否成功

```shell
sudo systemctl start docker
sudo docker run hello-world
```

5、配置国内镜像仓库地址
新建/etc/docker/daemon.json文件，输入如下内容：

```json
{
  "registry-mirrors": [
    "https://registry.docker-cn.com",
    "http://hub-mirror.c.163.com",
    "https://fsp2sfpr.mirror.aliyuncs.com/"
  ]
}
```

6、然后重启，配置开机启动
```shell
systemctl restart docker
systemctl enable docker
systemctl enable containerd
```

# 3、docker run开箱即用

## 3.1、docker架构

![image.png](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/20240517143534.png)

**registry 镜像仓库**

registry可以理解为镜像仓库，用于保存docker image。

Docker Hub 是docker官方的镜像仓库，docker命令默认从docker hub中拉取镜像。我们也可以搭建自己的镜像仓库。

**image 镜像**

image可以理解为一个只读的应用模板。image包含了应用程序及其所需要的依赖环境，例如可执行文件、环境变量、初始化脚本、启动命令等。

**container 容器**

容器是image的一个运行实例。当我们运行一个image，就创建了一个容器。


## 3.2、docker pull拉取镜像

从镜像仓库拉取镜像到本地

```shell
docker pull nginx 不写默认是latest
docker pull nginx:latest
docker pull nginx:1.22
docker pull nginx:1.22.0-alpine
```

一般不建议使用latest，因为最新的镜像是滚动更新的，过一段时间，可能跟你本地的不是同一个。
使用docker images命令查看本地镜像

## 3.3、docker run命令

docker run [可选参数] 镜像名:版本 []

1、公开端口（-P）
```shell
docker run --name some-nginx -d -p 8080:80 nginx:1.22
默认情况下，容器无法通过外部网络访问。
需要使用-p参数将容器的80端口映射到宿主机8080端口，才可以通过宿主机IP进行访问。
浏览器打开 http://192.168.56.106:8080

```
2、后台运行
```shell
docker run --name db-mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7
```
>使用run命令，部署mysql，docker先去本地查找镜像，如果找不到，就去docker hub中拉取镜像
--name 定义容器的名称
-e 声明环境变量
-d容器在后台运行

```shell
# 查看运行ip
docker inspect \
	--format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' db-mysql
```
3、前台交互运行

创建一个新的容器，使用mysql客户端

```shell
docker run -it --rm mysql:5.7 mysql -h172.17.0.2 -uroot -p
```
>
-it 使用交互模式，可以在控制台里输入、输出
--rm在容器退出时自动删除容器。一般在使用客户端程序时使用此参数。
如果每次使用客户端都创建一个新的容器，这样将占用大量的系统空间。
mysql -h172.17.0.2 -uroot -p表示启动容器时执行的命令。


```shell
# docker exec在运行的容器中执行命令，一般配合-it参数使用交互模式
docker exec -it db-mysql /bin/bash
```

## 3.4、常用命令

```shell
# 查看正在运行的容器
docker ps 
# 查看所有容器，包括正在运行和停止的
docker ps -a
# 查看容器信息
docker inspect
# 查看日志
docker logs
# 在容器和宿主机间复制文件
docker cp ./some_file 容器名:/work
docker cp 容器名:/var/logs/ /tmp/app_logs
```

![image.png|625](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/20240515161620.png)

# 4、Docker网络

## 4.1、默认网络

docker会自动创建三个网络，bridge,host,none

![image.png](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/20240517150727.png)

1、bridge桥接网络

如果不指定，新创建的容器默认将连接到bridge网络。
默认情况下，使用bridge网络，宿主机可以ping通容器ip，容器中也能ping通宿主机。
容器之间只能通过 IP 地址相互访问，由于容器的ip会随着启动顺序发生变化，因此不推荐使用ip访问。

2、host

慎用，可能会有安全问题。
容器与宿主机共享网络，不需要映射端口即可通过宿主机IP访问。（-p选项会被忽略）
主机模式网络可用于优化性能，在容器需要处理大量端口的情况下，它不需要网络地址转换 （NAT），并且不会为每个端口创建“用户空间代理”。

3、none

禁用容器中所用网络，在启动容器时使用

## 4.2、用户自定义网络

创建用户自定义网络

```shell
docker network create my-net
```

将已有容器连接到此网络

```shell
docker network connect my-net db-mysql
```

创建容器时指定网络

```shell
docker run -it --rm --network my-net mysql:5.7 mysql -h**db-mysql** -uroot -p
```


在用户自定义网络上，容器之间可以通过容器名进行访问。

用户自定义网络使用 Docker 的嵌入式 DNS 服务器将容器名解析成 IP。

# 5、docker存储

将数据存储在容器中，一旦容器被删除，数据也会被删除。同时也会使容器变得越来越大，不方便恢复和迁移。

将数据存储到容器之外，这样删除容器也不会丢失数据。一旦容器故障，我们可以重新创建一个容器，将数据挂载到容器里，就可以快速的恢复

![image.png](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/20240517153044.png)

## 5.1、挂载绑定（bind mount）

绑定挂载可以将主机文件系统上目录或文件装载到容器中，但是主机上的非 Docker 进程可以修改它们，同时在容器中也可以更改主机文件系统，包括创建、修改或删除文件或目录，使用不当，可能会带来安全隐患

绑定挂载适用以下场景：

- 将配置文件从主机共享到容器。
- 在 Docker 主机上的开发环境和容器之间共享源代码或编译目录。
- 例如，可以将 Maven 的`target/`目录挂载到容器中，每次在主机上用 Maven打包项目时，容器内都可以使用新编译的程序包

1、-V

绑定挂载将主机上的目录或者文件装载到容器中。绑定挂载会覆盖容器中的目录或文件
如果宿主机目录不存在，docker会自动创建这个目录。但是docker只自动创建文件夹，不会创建文件。
例如，mysql的配置文件和数据存储目录使用主机的目录。可以将配置文件设置为只读（read-only）防止容器更改主机中的文件。

```shell
docker run -e MYSQL_ROOT_PASSWORD=123456 \
           -v /home/mysql/mysql.cnf:/etc/mysql/conf.d/mysql.cnf:ro  \
           -v /home/mysql/data:/var/lib/mysql  \
           -d mysql:5.7 
```

2、--tmpfs临时挂载
临时挂载将数据保留在主机内存中，当容器停止时，文件将被删除。

```shell
docker run -d -it --tmpfs /tmp nginx:1.22-alpine
```

## 5.2、volume卷

卷 是docker 容器存储数据的首选方式，卷有以下优势：

- 卷可以在多个正在运行的容器之间共享数据。仅当显式删除卷时，才会删除卷。
- 当你想要将容器数据存储在外部网络存储上或云提供商上，而不是本地时。
- 卷更容易备份或迁移，当您需要备份、还原数据或将数据从一个 Docker 主机迁移到另一个 Docker 主机时，卷是更好的选择。

```shell
docker volume create my-data

docker run -e MYSQL_ROOT_PASSWORD=123456 \
           -v /home/mysql/conf.d/my.cnf:/etc/mysql/conf.d/my.cnf:ro  \
           -v my-data:/var/lib/mysql  \
           -d mysql:5.7 
# 创建nfs卷
docker volume create --driver local \
    --opt type=nfs \
    --opt o=addr=192.168.1.1,rw \
    --opt device=:/path/to/dir \
    vol-nfs
```

# 6、部署自己的应用

本例子我们使用docker来部署一个RuoYi应用系统
将源码编译打包成ruoyi-admin.jar文件，放到宿主机/home/app目录下，/home/app/sql目录下是数据库初始化脚本

1、准备工作：创建网络和存储卷

```shell
docker volume create ruoyi-data
docker network create ruoyi-net
```
2、部署mysql并初始化数据库

```shell
docker run -e MYSQL_ROOT_PASSWORD=123456 \
           -e MYSQL_DATABASE=ry \
					 -v /home/app/sql:/docker-entrypoint-initdb.d \
           -v ruoyi-data:/var/lib/mysql  \
        	 --network ruoyi-net \
           --name ruoyi-db \
           -d mysql:5.7 \
           --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
```

3、部署应用

```shell
docker run -p 8080:8080 \
					 -v /home/app/ruoyi-admin.jar:/usr/local/src/ruoyi-admin.jar \
        	 --network ruoyi-net \
           --name ruoyi-java \
					 -d openjdk:8u342-jre \
           java -jar /usr/local/src/ruoyi-admin.jar
```

# 7、Docker compose容器编排

在实际工作中，部署一个应用可能需要部署多个容器，一个一个部署非常不方便。docker compose可以一键部署和启动多个容器，它使用yaml文件来编排服务。

github和docker hub很多项目都提供了docker-compose.yaml文件，我们可以一键部署项目，非常方便

## 7.1、compose文件结构

```yml
version: '3'  # 指定Compose文件的版本
services:     # 定义应用程序服务
  web:
    image: nginx  # 指定服务的Docker镜像
    ports:
      - "80:80"  # 映射端口
    volumes:
      - ./html:/usr/share/nginx/html  # 挂载卷
  db:
    image: postgres
    environment:
      POSTGRES_PASSWORD: example  # 设置环境变量
 
networks:   # 定义网络
  front-tier:
    driver: bridge
 
volumes:    # 定义卷
  db-data:
```

## 7.2、一键部署wordpress

wordpress是一个著名的开源博客系统。
将以下内容保存到本地的docker-compose.yml文件中。
docker compose命令启动时，默认在当前目录下寻找compose.yaml或compose.yml，
为了兼容之前的版本，也会查找docker-compose.yaml或docker-compose.yml。
也可以使用-f参数手动指定文件docker compose -f docker-compose-dev.yml up -d

```yml
services: 

  ruoyi-app:
    #  docker run --name ruoyi-app      \
    #             -p 8080:8080        \
    #             --network ruoyi-net      \
    #             -v /home/app/ruoyi-admin.jar:/usr/local/src/ruoyi-admin.jar   \
    #             -d openjdk:8u342-jre    \
    #             java -jar /usr/local/src/ruoyi-admin.jar
    image: openjdk:8u342-jre
    restart: always
    ports:
      - 8080:8080
    networks:
      - ruoyi-net
    volumes:
      - /home/app/ruoyi-admin.jar:/usr/local/src/ruoyi-admin.jar
    command: [ "java", "-jar", "/usr/local/src/ruoyi-admin.jar" ]
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 10s
    depends_on:
      ruoyi-db:
        condition: service_healthy

  ruoyi-db:
    #  docker run --name ruoyi-db -p 3303:3306 \
    #             --network ruoyi-net        \
    #             -v ruoyi-data:/var/lib/mysql  \
    #             -v /home/app/sql:/docker-entrypoint-initdb.d   \
    #             -e MYSQL_DATABASE=ry         \
    #             -e MYSQL_ROOT_PASSWORD=123456    \
    #             -d mysql:5.7      \
    #             --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci --skip-character-set-client-handshake
    image: mysql:5.7
    environment:
      - MYSQL_DATABASE=ry
      - MYSQL_ROOT_PASSWORD=123456
    volumes:
      - ruoyi-data:/var/lib/mysql
      - /home/app/sql:/docker-entrypoint-initdb.d
    networks:
      - ruoyi-net
    command:
      [
        "--character-set-server=utf8mb4",
        "--collation-server=utf8mb4_unicode_ci",
        "--skip-character-set-client-handshake"
      ]
    healthcheck:
      test: ["CMD", 'mysqladmin', 'ping', '-h', 'localhost', '-u', 'root', '-p$$MYSQL_ROOT_PASSWORD']
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 10s

volumes:
  ruoyi-data:

networks:
  ruoyi-net:

```

相关命令：
```shell
#一键部署启动
docker compose up -d 
# 启动/停止服务
docker compose start/stop
# 停止并删除容器，不会删除存储卷volume
docker compose down 
```

# 8、Dockerfile镜像制作


dockerfile通常包含以下几个常用命令：

```shell
FROM ubuntu:18.04
WORKDIR /app
COPY . .
RUN make .
CMD python app.py
EXPOSE 80
```
`FROM` 打包使用的基础镜像
`WORKDIR`相当于`cd`命令，进入工作目录
`COPY` 将宿主机的文件复制到容器内
`RUN`打包时执行的命令，相当于打包过程中在容器中执行shell脚本，通常用来安装应用程序所需要的依赖、设置权限、初始化配置文件等
`CMD`运行镜像时执行的命令
`EXPOSE`指定容器在运行时监听的网络端口，它并不会公开端口，仅起到声明的作用，公开端口需要容器运行时使用-p参数指定。

## 8.1、制作自己的镜像

```shell
FROM openjdk:8u342-jre
WORKDIR /app
COPY ./ruoyi-admin.jar .
CMD [ "java", "-jar", "ruoyi-admin.jar" ]
EXPOSE 8080
```

docker build . 打包

## 8.2、image镜像与layer层

image文件由一系列层构建而成，dockerfile每一个命令都会生成一个层，每一层都是只读的
可以使用：docker image history + 镜像名 命令查看

创建容器时，会创建一个新的可写层，通常称为“容器层”，对正在运行的容器所做的所有更改（如写入新文件，修改现有文件或删除文件）都将写入容器层而不会修改镜像

![image.png](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202405181122438.png)

## 8.3、多阶段构建

在构建基于java应用时，需要一个JDK将源码编译为java字节码，但是在生产环境中不需要该JDK。多阶段构建可以将生成时依赖和运行时依赖分开，减少整个image文件大小

以maven/tomcat为例。使用 Maven来构建应用，在最终的image中不需要包含maven。我们可以使用多阶段构建，每一个阶段从`FROM`开始，最终的image只会从最后一个阶段构建，不会包含前面阶段产生的层，因此可以减少镜像体积

```shell
FROM maven AS build
WORKDIR /source
COPY . .
RUN mvn package

FROM  openjdk:8u342-jre
WORKDIR /app
COPY --from=build /source/ruoyi-admin/target/ruoyi-admin.jar .
EXPOSE 80
ENTRYPOINT ["java","-jar","ruoyi-admin.jar"]
```

> ENTRYPOINT和CMD的区别：
> dockerfile至少应包含一个ENTRYPONIT和CMD
> ENTRYPOINT指定容器启动时执行的默认程序，一般运行容器时不会被替换或覆盖，除非使用--entrypoint进行指定
> cmd可以在容器启动时被替换或覆盖

# 9、私有仓库

我们可以使用docker push将自己的image推送到docker hub中进行共享，但是在实际工作中，很多公司的代码不能上传到公开的仓库中，因此我们可以创建自己的镜像仓库。
docker 官网提供了一个docker registry的私有仓库项目，可以方便的通过docker部署。

```shell
docker run -d -p 5000:5000 --restart always --name registry registry:2 
docker image tag ruoyi-java:4.7.4 localhost:5000/ruoyi-java:4.7.4
docker push localhost:5000/ruoyi-java:4.7.4
docker pull localhost:5000/ruoyi-java:4.7.4
```


# 10、镜像导入导出

当我们处于离线状态，比如在很多内网上不能访问互联网，这时候不能通过镜像仓库的方式共享image，我们可以使用导出和导入功能，手动拷贝镜像

```shell
# docker save会包含所有层，以及所有标签 + 版本信息
docker save alpine:3.15 > alpine-3.15.tar
# 加载image
docker load < alpine-3.15.tar 
```

>注意：
 不要跟export和import命令混淆
 docker save/load IMAGE save和load操作的是镜像
 docker export/import CONTAINER export和import操作对象是容器
 image包含多个层，每一层都不可变，save保存的信息包含每个层和所有标签 + 版本信息。
 容器运行的时候会创建一个可写入的容器层，所有的更改都写入容器层，export导出的只有容器层，不包含父层和标签信息。
