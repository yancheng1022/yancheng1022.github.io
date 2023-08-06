---
title: dubbo
date: 2023/08/04
categories:
  - coding
tags:
  - dubbo
  - 编程基础
abbrlink: 22179
---

# 1、基本概念
Apache Dubbo是一款高性能、轻量级的开源Java RPC框架，它提供了三大核心能力：面向接口的远程方法调用，智能容错和负载均衡，以及服务自动注册和发现

 >**dubbo可以是微服务的一部分，但不能以偏概全说dubbo就是微服务**，微服务涉及的面比较广，比如服务发现，服务治理，服务网关，服务监控，链路追踪等等，可以用到的组件也比较多，而dubbo最多只能说是专注于服务治理的组件，所以从这一点上来看，可替代它的技术也是相当之多的，比如一系列rpc框架都可以

## 1.1、基本架构

![dubbo架构](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202308041522327.png)


Provider 暴露服务的服务提供方
Consumer 调用远程服务的服务消费方
Registry 服务注册与发现的注册中心
Monitor 统计服务的调用次数和调用时间的监控中心

## 1.2、调用流程

1. 服务启动，包括服务提供者和消费者的启动，封装服务调用链路。
2. 服务提供者在启动时，向注册中心注册自己提供的服务。
3. 服务消费者在启动时，向注册中心订阅自己所需的服务。
4. 注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。
5. 服务消费者，从提供者地址列表中，基于服务路由信息、负载均衡规则，选一台提供者进行调用。
6. 服务消费者和提供者，在内存中累计调用次数和调用时间，定时发送一次统计数据到监控中心。
7. 服务提供方停止服务或者服务调用方关闭JVM的时候，会将provider及consumer进行销毁处理。


# 2、Dubbo环境搭建
## 2.1、安装zookeeper

1. 下载zookeeper

网址:
[https://archive.apache.org/dist/zookeeper/zookeeper-3.4.13/](https://archive.apache.org/dist/zookeeper/zookeeper-3.4.13/)  

2. 解压zookeeper
解压运行zkServer.cmd ，初次运行会报错，没有zoo.cfg配置文件 

3. 修改zoo.cfg配置文件
将conf下的zoo_sample.cfg复制一份改名为zoo.cfg即可,修改完成后再次启动zookeeper 

>注意几个重要位置:
dataDir=./   临时数据存储的目录（可写相对路径）
clientPort=2181   zookeeper的端口号

4. 使用zkCli.cmd/sh测试

>ls /：列出zookeeper根下保存的所有节点
create –e /kaka123：创建一个kaka节点，值为123
get /kaka：获取/kaka节点的值
>(dubbo注册后一个group是一个节点)


## 2.2、安装监控中心
为了让用户更好的管理监控众多的dubbo服务，官方提供了一个可视化的监控程序，不过这个监控即使不装也不影响使用

1. 下载dubbo-admin
[https://github.com/apache/incubator-dubbo-ops](https://github.com/apache/incubator-dubbo-ops) 
（注意是master分支） |


2. 进入目录，修改dubbo-admin配置
修改src\\main\\resources\\application.properties 指定zookeeper地址

3. 打包dubbo-admin

```shell
mvn clean package
```

4. 运行dubbo-admin

```shell
java -jar dubbo-admin-0.0.1-SNAPSHOT.jar
```

默认使用root/root 登陆

## 2.3、父工程
创建依赖
```java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>dubboDemo</artifactId>
    <packaging>pom</packaging>
    <version>1.0-SNAPSHOT</version>
    <modules>
        <module>provider</module>
        <module>api</module>
        <module>consumer</module>
    </modules>

    <!-- 父级引用 -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.1.RELEASE</version>
        <relativePath/>
    </parent>


    <!--配置-->
    <properties>
        <java.version>1.8</java.version>
        <dubbo.version>2.7.5</dubbo.version>
        <zookeeper.version>3.4.14</zookeeper.version>
    </properties>


    <!--声明全局依赖（子项目需要显示的引用才会继承依赖）-->
    <dependencyManagement>
        <dependencies>
            <!-- dubbo-start依赖 -->
            <dependency>
                <groupId>org.apache.dubbo</groupId>
                <artifactId>dubbo-spring-boot-starter</artifactId>
                <version>${dubbo.version}</version>
            </dependency>
            <!--zookeeper 注册中心客户端引入 使用的是curator客户端 -->
            <dependency>
                <groupId>org.apache.dubbo</groupId>
                <artifactId>dubbo-dependencies-zookeeper</artifactId>
                <version>${dubbo.version}</version>
                <type>pom</type>
                <exclusions>
                    <exclusion>
                        <artifactId>slf4j-log4j12</artifactId>
                        <groupId>org.slf4j</groupId>
                    </exclusion>
                </exclusions>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <!--声明全局依赖（子项目不需要显示的引用，自动继承依赖）-->
    <dependencies>
        <!-- spring boot 依赖 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>

    <!-- 打包插件 -->
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```
## 2.4、api模块
创建接口

```java
public interface Api1 {
    public String test();
}

```

## 2.5、服务提供者模块

1. 引入依赖

```java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>dubboDemo</artifactId>
        <groupId>org.example</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>provider</artifactId>

    <dependencies>
        <!--接口模块-->
        <dependency>
            <groupId>org.example</groupId>
            <artifactId>api</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
        </dependency>
        <!-- zookeeper依赖 -->
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-dependencies-zookeeper</artifactId>
            <version>${dubbo.version}</version>
            <type>pom</type>
            <exclusions>
                <exclusion>
                    <artifactId>slf4j-log4j12</artifactId>
                    <groupId>org.slf4j</groupId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

2. 配置文件

```java
server:
  # 服务端口
  port: 7777
spring:
  application:
    name: provider
# dubbo 相关配置(dubbo 的配置不再以 Spring 开头)
dubbo:
  application:
    # 应用名称
    name: provider
  scan:
    # 接口实现者（服务实现）包
    base-packages: com.kaka.service
  # 注册中心信息
  registry:
    address: zookeeper://127.0.0.1:2181
  protocol:
    # 协议名称
    name: dubbo
    # 协议端口
    port: 20880
```

3. 服务实现

```java
@Service
public class Service1 implements Api1 {
    @Override
    public String test() {
        System.out.println("服务提供者提供服务...");
        return "success";
    }
}
```
## 2.6、服务消费者

1. 依赖

```java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>dubboDemo</artifactId>
        <groupId>org.example</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>consumer</artifactId>
    <dependencies>
        <!--接口模块-->
        <dependency>
            <groupId>org.example</groupId>
            <artifactId>api</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <!-- web项目依赖 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- dubbo依赖 -->
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
        </dependency>

        <!-- dubbo的zookeeper依赖 -->
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-dependencies-zookeeper</artifactId>
            <version>${dubbo.version}</version>
            <type>pom</type>
            <exclusions>
                <exclusion>
                    <artifactId>slf4j-log4j12</artifactId>
                    <groupId>org.slf4j</groupId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

2. 配置文件

```java
server:
  port: 7000
spring:
  application:
    name: consumer
# dubbo 相关配置
dubbo:
  application:
    name: consumer
  registry:
    address: zookeeper://127.0.0.1:2181
```

3. 消费者类

```java
@RestController
public class Consumer1 {
    @Reference
    Api1 api1;

    @GetMapping("/test")
    public String getOrder() {
        String test = api1.test();
        return test;
    }
}
```
