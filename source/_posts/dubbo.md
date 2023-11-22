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

调用过程:

(1) 服务容器 Container 负责启动加载运行服务提供者 Provider。根据配置中的 Registry 地址连接 Registry，在 Registry 注册自己提供的服务。
(2) Consumer 在启动时，根据配置文件中的服务引用信息，连接到 Registry，向 Registry 订阅自己所需的服务。
(3) Registry 根据服务订阅关系，返回 Provider 地址列表给 Consumer。如果有变更，Registry 会基于长连接推送最新的服务地址信息给 Consumer。
(4) Consumer 调用远程服务时，基于负载均衡算法，从缓存的 Provider 地址列表中选择一台进行跨进程调用服务
(5) 服务 Provider 和 Consumer，会在内存中记录调用次数和调用时间，每分钟发送一次统计数据到 Monitor。

## 1.2、调用流程

1. 服务启动，包括服务提供者和消费者的启动，封装服务调用链路。
2. 服务提供者在启动时，向注册中心注册自己提供的服务。
3. 服务消费者在启动时，向注册中心订阅自己所需的服务。
4. 注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。
5. 服务消费者，从提供者地址列表中，基于服务路由信息、负载均衡规则，选一台提供者进行调用。
6. 服务消费者和提供者，在内存中累计调用次数和调用时间，定时发送一次统计数据到监控中心。
7. 服务提供方停止服务或者服务调用方关闭JVM的时候，会将provider及consumer进行销毁处理。

## 1.3、dubbo和springCloud区别
两者都是现在主流的分布式框架，但却存在不少差异：

- **生态环境不同：** SpringCloud定位为微服务架构下的一站式解决方案（网关，分布式配置，服务跟踪）；Dubbo 是 SOA 时代的产物，它的关注点主要在于服务的调用和治理
- **调用方式：** SpringCloud是采用Http协议做远程调用；Dubbo是基于RPC调用
- **组件差异比较多**，例如SpringCloud注册中心一般用Eureka，而Dubbo用的是Zookeeper

![image.png](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202311141619063.png)



## 1.4、dubbo支持协议

**1、dubbo 默认协议：**

- 单一 TCP 长连接，Hessian 二进制序列化和 NIO 异步通讯
- 不适合传送大数据包的服务

**2、rmi 协议：**

- 采用 JDK 标准的 java.rmi.* 实现，采用阻塞式短连接和 JDK 标准序列化方式
- 对传输数据包不限，消费者和传输者个数相当

**3、hessian 协议：**

- 底层 Http 通讯，Servlet 暴露服务，Dubbo 缺省内嵌 Jetty 作为服务器实现
- 通讯效率高于 WebService 和 Java 自带的序列化
- 适用于传输数据包较大，提供者比消费者个数多，提供者压力较大

**4、http 协议：**

- 基于 http 表单的远程调用协议，短连接，json 序列化
- 对传输数据包不限，不支持传文件

**5、webservice 协议：**

- 基于 Apache CXF 的 frontend-simple 和 transports-http 实现，短连接，SOAP文本序列化
- 可与原生 WebService 服务互操作
- 适用于系统集成、跨语言调用

**6、thrift 协议：**

- 对 thrift 原生协议的扩展添加了额外的头信息
- 使用较少，不支持传 null 值

**7、基于 Redis实现的 RPC 协议**
**8、基于 Memcached 实现的 RPC 协议**


## 1.5、dubbo负载均衡策略

```java
dubbo:
  provider:
    loadbalance: roundrobin
```


> 也可以在注解上进行配置
> @Service(version = "${product.service.version}",loadbalance="roundrobin")

1. RandomLoadBalance:随机负载均衡。随机的选择一个。是Dubbo的**默认**负载均衡策略。
2. RoundRobinLoadBalance:轮询负载均衡。轮询选择一个。
3. LeastActiveLoadBalance: 最小活跃数负载均衡，活跃数也就是dubbo的连接数，每当收到一个请求活跃数+1，结束请求活跃数-1，假设如果多台机器的连接数是相同的，如果一台机器性能比较好，处理请求比较快那么活跃数减少的就快，活跃数就少。所以活跃数少的就会获取到的请求会变多，这样就可以合理的使用性能不同的机器了
4. ConsistentHashLoadBalance:一致性哈希负载均衡。相同参数的请求总是落在同一台机器上。

## 1.6、dubbo集群容错

| **类型** | **负载均衡** | **备注** |
| --- | --- | --- |
| failover | 会 | 失败后会尝试调用其他服务器实例，默认尝试2次， 可以通过设置retries来设置次数。这是dubbo默认的容错机制，由于常常可能因为超时待原因触发异常但远程服务已经完成操作，所以这个类型不应就在数据更新的操作 |
| failfast | 会 | 有异常立即返回，不做尝试，我认为可以用在数据更新上，以保证数据的一致性 |
| fastsafe | 会 | 有异常会直接忽略，为的是保证调用方接下来的正常运行，一般用于日志收集等与正常流程无关的操作 |
| failback | 会 | 失败后会将任务丢到失败队列中，并会异步再次尝试 |
| forking | 否 | 同时调用多个服务，取最先返回的结果，可以通过forks设置最大并行数，这比较浪费资源 |
| broadcast | 否 | 调用所有可用的服务，任意一个有错都会返回异常 |
| Mock | 否 | 调用失败时返回伪造的响应结果 |

> 一般在@DubboService或@DubboReference指定cluster即可，如
  @DubboService(cluster = “failover”) //默认重试2次


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
