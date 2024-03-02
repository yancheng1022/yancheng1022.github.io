---
title: springCloud
date: 2022/07/29
tags:
  - springCloud
  - 编程基础
abbrlink: 26325
categories:
---
#  1、微服务架构
## 1.1、基本概念
微服务是一种架构，这种架构是将单个整体应用程序分割成更小的项目关联的独立服务。一个服务通常实现一组独立的特性或功能，包含自己的业务逻辑和适配器。各个微服务之间的关联通过暴露api来实现，这些独立的微服务

## 1.2、为什么要用微服务？
### 1.2.1、优点

1. 将服务拆分成单一职责的小服务，进行单独部署，服务之间通过网络进行通信
2. 每个服务应该有自己的管理团队，高度自治
3. 服务各自有自己单独的职责，服务之间松耦合，避免因一个模块的问题导致服务崩溃

### 1.2.2、缺点

1. 开发人员需要处理分布式系统的复杂性
2. 随着服务的增加，运维的压力也在增大
3. 服务治理（负载均衡，服务熔断，服务配置管理）和服务监控

## 1.3、架构的演变

### 1.3.1、单一架构（all in one）
一个应用，将所有功能都部署在一起，以减少部署节点和成本。此时，用于简化增删改查工作量的数据访问框架(**ORM**)是影响项目开发的关键（mybatis，hibernate）

> 缺点：随着应用功能的增多，代码量越来越大，越来越难维护


### 1.3.2、垂直架构（vertical application）
当访问量逐渐增大，单一应用无法满足需求，我们就需要增加节点来提供系统的访问能力，但是并不是所有的模块都需要进行性能的提高，这时候单体应用架构无法满足我们的需求；我们需要将系统里面的模块进行拆分，这样对于后面的水平扩容是非常友好的；
![垂直架构](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202307290956007.jpg)

> 优点：系统拆分实现了流量分担，提高了系统并发量
>    垂直架构中可以针对不同模块进行针对性优化
>    方便水平扩展，负载均衡，系统容错率提高

> 缺点：垂直架构中相同逻辑代码需要不断的复制，不能复用。每个垂直模块都相当于一个独立的系统


### 1.3.3、分布式服务架构（distributed service）
当垂直应用越来越多，重复的业务代码就会越来越多，并且在垂直架构中应用之间的交互不可避免，此时，为了解决基础代码重复太多、应用之间的调用等问题；我们将重复的代码抽取出来作为独立的服务，对外提供服务；
> rpc是传输层传输协议，效率比应用层传输要高
> dubbo采用rpc
> springcloud采用http，属于应用层传输

> 优点：将基础服务进行了抽取，系统间相互调用，提高了代码复用和开发效率

> 缺点：服务越来越多，需要管理每个服务的地址，调用关系错综复杂，难以理清依赖关系，服务状态难以管理，无法根据服务情况动态管理

![分布式架构](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202307291006525.jpg)

### 1.3.4、SOA架构
在分布式架构下，当服务越来越多，容量的评估，小服务资源等浪费等问题逐渐显现，此时需增加一个调度中心对集群进行实时管理（根据请求量动态的分配资源）。它一般使用中心化的思想实现，服务的管理和调度都由中央的服务总线（ESB）来负责

### 1.3.4、微服务架构
微服务架构模式是从SOA架构模式演变过来， 比SOA架构模式粒度更加精细，让专业的人去做专业的事情（专注），目的是提高效率，每个服务与服务之间互不影响，微服务架构中每个服务独立，互不影响；（怎么理解粒度更细？：微服务采用去中心化的思想来治理，注册中心只是服务发现的工具，而服务之间的调用，熔断，负载均衡等都是都是由服务自己控制的）

![微服务架构](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202307291013480.png)


## 1.4、微服务架构解决方案

- **国内阿里系**

springboot + dubbo + zookeeper

- **spring cloud技术栈**

spring cloud netflix
spring cloud spring （spring自己封装的微服务解决方案）
spring cloud alibaba（阿里巴巴解决方案）

## 1.5、SpringCloud概念
### 1.5.1、基本概念
Spring Cloud是一个含概多个子项目的微服务开发工具集,集合了众多的开源框架,他利用了Spring Boot开发的便利性实现了很多功能,如服务注册,服务注册发现,负载均衡等.Spring Cloud在整合过程中主要是针对Netflix(耐非),alibaba开源组件的封装

### 1.5.2、版本
springcloud版本采用伦敦地铁站命名，根据首字母顺序排序 这样设计的目的是为了更好的管理每个springcloud子项目清单，避免了总版本号与子项目版本号混淆

![image.png](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202307291014046.png)


### 1.5.3、组件
|  | spring cloud官方 | spring cloud netflix | spring cloud alibaba |
| --- | --- | --- | --- |
| 服务注册发现 | - | Eureka | nacos |
| 分布式配置 | spring cloud config | Archaius | nacos |
| 服务熔断 | - | Hystrix | Sentinel |
| 服务调用 | Open Fegin | Fegin | Dubbo RPC |
| 服务路由 | spring cloud gateway | zuul | dubbo proxy |
| 分布式消息 | rabbitmq | - | rocketmq |
| 负载均衡 | - | ribbon | dubbo LB |
| 分布式事务 | - | - | Seata |


## 1.6、分布式和微服务的关系

我理解的分布式是一种系统架构，指一个系统由多个独立的组件组成，这些组件可以在不同的物理位置上运行，从而提升了系统的性能
而微服务可以看作是分布式的一种实现方案。分布式实现方案有soa，基于rpc远程调用（dubbo），微服务（springcloud）

# 2、项目搭建（父工程）
## 2.1、引入依赖

```xml
<!--springBoot父项目-->
<parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-parent</artifactId>
  <version>2.2.5.RELEASE</version>
</parent>
    <dependencyManagement>
        <dependencies>
            <!--springBoot-->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter</artifactId>
            </dependency>
            <!--springCloud-->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Hoxton.SR6</version>
                <!--maven只能有一个parent 所以这里以pom引入父项目-->
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```
# 3、eureka（注册中心）
## 3.1、基本概念
注册中心可以说是微服务架构中的”通讯录“，它记录了服务和服务地址的映射关系。在分布式架构中，服务会注册到这里，当服务需要调用其它服务时，就到这里找到服务的地址，进行调用。两个重要的功能：**服务注册**和**服务发现**

## 3.2、常用的注册中心

![常用的注册中心](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202307291044882.png)


## 3.3、eureka角色

- **服务注册中心（server**）：Eureka的服务端应用，提供服务注册和发现功能.
- **服务提供者（client）**： 启动后向Eureka注册自己信息（地址，提供什么服务）

> 心跳(续约)：提供者定期通过http方式向Eureka刷新自己的状态

- **服务消费者（client）**：向Eureka订阅服务，Eureka会将对应服务的所有提供者地址列表发送给消费者，并且定期更新

## 3.4、eureka server开发
搭建eureka server子项目
### 3.4.1、引入依赖

```xml
        <!--springBoot-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--eurekaServer-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
```
### 3.4.2、配置文件

```properties
# eureka server端口号 （默认就是8761）
server.port=8761
# 指定服务名称
spring.application.name=EUREKASERVER
# eureka server服务注册中心地址 暴露服务地址
eureka.client.service-url.defaultZone=http://localhost:8761/eureka
# 关闭立即注册（避免控制台报错）
eureka.client.fetch-registry=false
# 让当前应用仅仅是服务注册中心
eureka.client.register-with-eureka=false
```
### 3.4.3、入口类加注解

```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}

```
## 3.5、eureka client开发
### 3.5.1、引入依赖

```xml
<dependencies>
  <!--eureka client依赖-->
  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
  </dependency>
  <!--springboot依赖-->
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
</dependencies>

```
### 3.5.2、配置文件

```properties
server.port=8989
# 应用名称
spring.application.name=EUREKACLIENT
# 注册中心地址
eureka.client.service-url.defaultZone=http://localhost:8761/eureka
```
### 3.5.3、启动类
```java
@SpringBootApplication
@EnableEurekaClient
public class EurekaClientApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaClientApplication.class, args);
    }
}
```

## 3.6、eureka自我保护机制 
### 3.6.1、自我保护机制
默认情况下，如果eureka server在一定时间内（默认90秒）没有接收到某个微服务实例的心跳，eureka server将会移除掉该实例（eureka客户端每隔60秒会发送一次心跳包）

但是当网络分区发生故障，微服务和eureka server之间无法通信，但是微服务本身是正常的，此时不应该移除这个服务，所以就引入了自我保护机制

Eureka服务端会检查最近15分钟内所有Eureka 实例正常心跳占比，如果低于85%就会触发自我保护机制。触发了保护机制，Eureka将暂时把这些失效的服务保护起来，不让其过期

> 自我保护机制清除：
> （1）心跳次数高于预期值
> （2）自我保护机制被禁用

### 3.6.2、如何关闭自我保护机制

```properties
eureka:
  server:
    #服务端是否开启自我保护机制 （默认true）
    enable-self-preservation: false
    # eureka客户端每隔多长时间发一次心跳（单位毫秒，默认是60*1000）即60秒
    eviction-interval-timer-in-ms: 60000
```

## 3.7、eureka server集群

多个注册中心之间相互注册就可搭建eureka server集群

> 服务端：eureka.client.service-url.defaultZone 配置其它的注册中心地址即可

```properties
# eureka server端口号 （默认就是8761）
server.port=8761
# 指定服务名称
spring.application.name=EUREKASERVER01
# eureka server服务注册中心地址 暴露服务地址
eureka.client.service-url.defaultZone=http://localhost:8762/eureka
# 关闭立即注册（避免控制台报错）
eureka.client.fetch-registry=false
# 让当前应用仅仅是服务注册中心
eureka.client.register-with-eureka=false

# eureka server端口号 
server.port=8762
# 指定服务名称
spring.application.name=EUREKASERVER02
# eureka server服务注册中心地址 暴露服务地址
eureka.client.service-url.defaultZone=http://localhost:8761/eureka
# 关闭立即注册（避免控制台报错）
eureka.client.fetch-registry=false
# 让当前应用仅仅是服务注册中心
eureka.client.register-with-eureka=false
```
 
> 客户端：所有的注册中心地址都需要指明

```properties
server.port=8989
# 应用名称
spring.application.name=EUREKACLIENT
# 注册中心地址
eureka.client.service-url.defaultZone=http://localhost:8761/eureka,http://localhost:8762/eureka
```
# 4、consul（注册中心）
## 4.1、简介
consul是基于go语言进行开发的服务注册中心，是一个可以直接运行的注册中心工具，不需要像eureka注册中心一样再进行编码。使用起来较为简单

## 4.2、安装

1. 下载 [https://www.consul.io/downloads](https://www.consul.io/downloads)
2. 解压
3. 启动服务注册中心（cmd）
```properties
consul agent -dev
```

4. 访问管理界面：[http://localhost:8500/](http://localhost:8500/)

## 4.3、consul client开发

1. 引入依赖

```xml
<dependencies>
        <!--springboot依赖-->
        <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--consul客户端组件-->
        <dependency>
          <groupId>org.springframework.cloud</groupId>
          <artifactId>spring-cloud-starter-consul-discovery</artifactId>
        </dependency>
          <!--健康检查依赖 actuator-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
</dependencies>

```

2. 编写配置文件

```properties
server.port=8082
spring.application.name=CONSULCLIENT
# 向consul server服务注册地址
spring.cloud.consul.host=localhost
spring.cloud.consul.port=8500
# 默认为${spring.application.name}
spring.cloud.consul.discovery.service-name=CONSULCLIENT
```

3. 启动类添加注解

```java
@SpringBootApplication
// 除了eureka，其它注册中心客户端都可以用该注解
@EnableDiscoveryClient
public class ConsulClientApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConsulClientApplication.class, args);
    }
}

```
## 4.4、consul健康检查

![image (10).png](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202307291356805.png)


1. 默认情况下consul监控健康是开启的，但必须依赖健康监控依赖才能正常响应客户端发来的心跳，否则界面会显示错误，引入健康依赖之后服务正常
```xml
<!--健康检查依赖 actuator-->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

# 5、openFeign（服务间通信）
## 5.1、基于RestTemplate实现调用
```java
@RestController
public class Client2Controller {
    @RequestMapping("/test")
    public String test(){
        return "client 2 OK!";
    }
}

@RestController
public class ClientController {
    @RequestMapping("/test")
    public String test(){
        RestTemplate restTemplate = new RestTemplate();
        String result = restTemplate.getForObject("http://localhost:8088/test", String.class);
        return "调用client2成功："+result;
    }
}
```

## 5.2、基于RestTemplate+Ribbon实现负载均衡调用
spring cloud ribbon是一个基于HTTP和TCP的客户端负载均衡工具，它基于netflix ribbon实现，通过spring cloud封装，可以让我们轻松的进行客户端负载均衡调用。Ribbon可从注册中心中获取服务提供者地址列表，并基于负载均衡算法，请求其中一个服务提供者实例

1. **编码形式**

```java
@RestController
public class ClientController {
    @Autowired // 服务注册与发现客户端对象（获取服务列表后需要手动选取）
    private DiscoveryClient discoveryClient;
    @Autowired // 具有负载均衡的服务注册与发现客户端对象（根据负载均衡策略自动选取一个）
    private LoadBalancerClient loadBalancerClient;

    @RequestMapping("/test")
    public String test() {
        // （1）DiscoveryClient
        List<ServiceInstance> serviceInstances = discoveryClient.getInstances("CONSULCLIENT2");
        String result = new RestTemplate().getForObject(serviceInstances.get(0).getUri() + "/test", String.class);
        // （2）LoadBalanceClient
        ServiceInstance serviceInstances2 = loadBalancerClient.choose("CONSULCLIENT2");
        String result2 = new RestTemplate().getForObject(serviceInstances2.getUri() + "/test", String.class);
        return "调用client2成功：" + result2;
    }
}
```

2. **@LoadBalance注解实现负载均衡**

```java
@Configuration
public class BeanConfig {
    @Bean
    @LoadBalanced // 使RestTemplate对象具有ribbon负载均衡特性
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }
}

@RestController
public class ClientController {
    @Autowired
    private RestTemplate restTemplate;

    @RequestMapping("/test")
    public String test() {
        String result3 = restTemplate.getForObject("http://CONSULCLIENT2/test", String.class);
        return "调用client2成功：" + result3;
    }
}
```

3. **Ribbon负载均衡策略**

| 策略类 | 命名 | 描述 |
| --- | --- | --- |
| RoundRobinRule | 轮询策略 | 轮询选择，轮询index，选择index对应位置的Server |
| RandomRule | 随机策略 | 随机选择server |
| BestAvailableRule | 最低并发策略 | 选择其中并发链接最低的server |
| RetryRule | 重试策略 | 其实就是轮询策略的加强版，轮询策略服务不可用时不处理，重试策略会尝试其它节点 |
| AvailabilityFilteringRule | 可用过滤策略 | 过滤掉一直连接失败的provider和繁忙的provider |
| ZoneAvoidanceRule | 区域权重策略 | 如果某个ip区域内有一个或多个示例不可达或响应慢，都会降低该ip区域内其它ip被选中的权重 |

4. **负载均衡策略设置**

（1）局部修改

```java
# 负载均衡策略 provider为调用的服务的名称
# 格式：服务应用名.ribbon.负载均衡策略名字
provider:
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
```
（2）全局修改

```java
@SpringBootApplication
@EnableDiscoveryClient
public class ConsulClientApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConsulClientApplication.class, args);
    }

    // 所有被调用服务均使用该策略
    @Bean
    public RandomRule randomRule(){
        return new RandomRule();
    }
}
```
## 5.3、基于openFeign实现调用
### 5.3.1、Feign


> Feign是Spring Cloud组件中一个轻量级RESTful的HTTP服务客户端，Feign内置了Ribbon，用来做客户端负载均衡，去调用服务注册中心的服务。Feign的使用方式是：使用Feign的注解定义接口，调用接口，就可以调用服务注册中心的服务

### 5.3.2、openFeign

> OpenFeign是Spring Cloud 在Feign的基础上支持了Spring MVC的注解，如@RequesMapping等等。
> OpenFeign的@FeignClient可以解析SpringMVC的@RequestMapping注解下的接口，
> 并通过动态代理的方式产生实现类，实现类中做负载均衡并调用其他服务


### 5.3.3、openFeign调用
#### 5.3.3.1、编码
（1）引入依赖

```xml
<!--openFeign-->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```
（2）启动类开启openFeign的调用

```java
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class ConsulClientApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConsulClientApplication.class, args);
    }
}
```
（3）创建业务层接口，添加@FeginClient注解声明需要调用的服务

```java
@FeignClient("CONSULCLIENT2")
public interface Client2 {
    @RequestMapping("/test")
    String test2();
}
```
（4）调用

```java
@RestController
public class ClientController {
    @Autowired
    private Client2 client2;

    @RequestMapping("/test")
    public String test() {
        String result = client2.test2();
        return "调用client2成功：" + result;
    }
}
```
#### 5.3.3.2、接口参数传递详解
> （1）test？name=xxx&age=10  —— @RequestParam("name")
> （2）test/{id}   ——  @PathVarible("id")
> （3）test(@RequestBody User user)
> （4）集合和数组作为参数时封装到VO对象中

#### 5.3.3.3、超时处理

1. 默认超时时间：1s
2. 修改某个服务的超时时间

```properties
feign.client.config.PRODUCTS.connecTimeOut=5000 # 配置指定服务连接超时
feign.client.config.PRODUCTS.readTimeOut=5000 # 配置指定服务等待超时
```

3. 修改所有服务超时时间

```properties
feign.client.config.default.connecTimeOut=5000 # 配置服务连接超时
feign.client.config.default.readTimeOut=5000 # 配置服务等待超时
```
# 6、Hystrix（服务熔断）
## 6.1、什么是服务雪崩？
因“服务提供者的不可用”（原因）导致“服务调用者不可用”（结果），并将不可用逐渐放大的现象

> （1）程序bug导致服务不可用，或者运行缓慢 
> （2）缓存击穿，导致调用全部访问某服务，导致down掉 
> （3）访问量的突然激增

## 6.2、解决服务雪崩（ 熔断 + 降级）

1. **服务熔断**

一般是指软件系统中，由于某些原因使得服务出现了过载现象，为防止造成整个系统故障，从而采用的一种保护措施，向调用方返回一个符合预期的的备选响应，而不是长时间等待或抛出异常。所以很多地方把熔断亦称为过载保护

2. **服务降级**

当网站或服务流量突然增加时，为了保证系统核心服务正常运行，有策略的关闭系统中的边缘服务，以保证核心服务的正常运行

3. **熔断降级的关系**

熔断必会出发降级，所以熔断也是降级的一种，区别在于熔断是对调用链路的保护，降级是对系统过载的一种保护
## 6.3、Hystrix概念
Hystrix是处理分布式系统延迟和容错的开源库，在分布式系统中，许多依赖不可避免的会调用失败，超时，异常等。Hystrix能够保证在一个依赖出现问题时，不会导致整体服务失败，避免雪崩效应，提高分布式系统的弹性

## 6.4、编码

1. 所有服务引入Hystrix依赖

```xml
<!--hystrix-->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

2. 服务提供者

（直接调用服务提供者的方案，是一种服务降级的思想）

```java

// 开启熔断功能
@SpringBootApplication
@EnableDiscoveryClient
@EnableHystrix //开启hystrix熔断功能
public class ConsulClient2Application {
    public static void main(String[] args) {
        SpringApplication.run(ConsulClient2Application.class,args);
    }
}


@RestController
public class Client2Controller {
    @RequestMapping("/test2")
    @HystrixCommand(fallbackMethod = "demoFallBack",defaultFallback = "defaultFallBack") //熔断之后的处理
    public String test(Integer id){
        if (id <= 0){
            throw new RuntimeException("发生异常！");
        }
        return "client 2:8088 OK!";
    }

    public String demoFallBack(Integer id){
        return "服务被熔断";
    }

    public String defaultFallBack(){
        return "服务被熔断(默认)";
    }
}

```

3. 服务消费者

（通过服务消费者调用的方案，服务者完全挂掉时的处理，是一种服务熔断的思想）
```java
# 开启openFeign支持服务降级
feign.hystrix.enabled=true

@SpringBootApplication
@EnableDiscoveryClient  // 开启服务注册客户端
@EnableFeignClients  // 开启openFeign调用
@EnableHystrix // 开启hystrix熔断功能
public class ConsulClientApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConsulClientApplication.class, args);
    }
}

// fallback:指定当服务不可用时，默认的备选处理
@FeignClient(value = "CONSULCLIENT2",fallback = Client2FallBack.class)
public interface Client2 {
    @RequestMapping("/test2")
    String test2(@RequestParam("id") Integer id);
}

// 
@Component
public class Client2FallBack implements Client2{
    @Override
    public String test2(Integer id) {
        return "当前服务不可达";
    }
}


@RestController
public class ClientController {
    @Autowired
    private Client2 client2;
    @RequestMapping("/test")
    public String test(Integer id) {
        String result = client2.test2(id);
        return "调用client2成功：" + result;
    }
}
```
## 6.5、Hystrix服务熔断机制
### 6.5.1、Hystrix断路器打开的条件
a. 当满足一定阈值的时候（默认10秒内超过20个请求次数）
b. 当失败率达到一定的时候（默认10秒内超过50%的请求失败）

> 注意：一旦断路开启之后，所有到这个服务请求均不可用，只有在断路关闭之后才可用

### 6.5.2、Hystrix监控流程
![Hystrix监控流程](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202307291357597.png)

当服务调用达到两个阈值，会自动开启断路器，在熔断器打开期间，任何到该接口的请求均不可用，同时在断路器打开5s后断路器会处于半开状态，此时断路器允许放行一个请求到该服务接口，如果执行成功，断路器彻底关闭，执行失败则重新打开

## 6.6、Hystrix Dashboard（仪表盘）

1. 基本概念

Hystrix Dashboard主要用来实时监控Hystrix的各项指标信息。通过Hystrix Dashboard反馈的实时信息，可以帮助我们快速发现系统中存在的问题

2. 编码

（1）新建一个 Hystrix Dashboard 工程
（2）引入依赖

```xml
<!-- Spring Cloud Hystrix Dashboard -->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
</dependency>

```
（3）启动类开启监控功能

```java
/** 开启 Hystrix Dashboard 监控功能 */
@EnableHystrixDashboard
```
（4）访问仪表盘界面

> [http://localhost:9000/hystrix](http://localhost:9000/hystrix)
> 端口为项目的端口号

# 7、zuul（网关）
Zuul 是从设备和网站到应用程序后端的所有请求的前门。作为边缘服务应用程序，Zuul 旨在实现动态路由，监视，弹性和安全性。zuul 包含了对请求的**路由**和**过滤**两个最主要的功能。

Zuul是 Netflix 开源的微服务网关，它可以和 Eureka、Ribbon、Hystrix 等组件配合使用。Zuul 的核心是一系列的过滤器，这些过滤器可以完成以下功能:

（1）聚合接口使得服务对调用者透明，客户端与后端的耦合度降低
（2）聚合后台服务，节省流量，提高性能，提升用户体验
（3）提供安全、流控、过滤、缓存、计费、监控等 API 管理功能
## 7.1、搭建环境
创建一个应用，在启动类中添加注解@EnableZuulProxy，声明这是一个网关服务提供者

## 7.2、路由规则配置
### 7.2.1、URL地址路由

```java
# 路由规则
zuul:
	routes :
        # 路由 id 自定义
        product-service :
			path: /product-service/**配请求 ur1 的映射路径#
    		url: http://localhost:7070/ # 映射路径对应的微服务地址
```
通配符含义：

| 通配符 | 含义 | 举例 |
| --- | --- | --- |
| ？ | 匹配任意单个字符 | /product-service/? |
| * | 匹配任意数量字符不包括子路径 | /product-service/* |
| ** | 匹配任意数量字符包括所有子路径 | /product-service/** |

### 7.2.2、服务名称路由
微服务一般是由几十、上百个服务组成，对于 URL 地址路由的方式，如果对每个服务实例手动指定一个唯一访问地址，这样做显然是不合理的。
Zuul 支持与 Eureka 整合开发，根据 serviceld 自动从注册中心获取服务地并转发清求，这样做的好处不仅可以通过单个端点来访问应用的所有服务，而目在添加或移除服务实例时不用修改 Zuul 的路由配置

```java
路由规则
Zuul
	routes:
	# 路由 id 自定义
		product-service:
		path: /product-service/**#配置请 url 的映射路径
		serviceId: product-service # 根据 serviceId 自动从注册中心获取服务地址并转发请求
```
### 7.2.3、简化路由配置
Zuul 为了方便大家使用，提供了默认路由配置: 路由 id 和 微服务名称 一致，path 默认对应 微服务名称/* ，所以以下配置就没必要再写了。

## 7.3、路由排除
我们可以通过路由排除设置不允许被访问的资源。允许被访问的资源可以通过路由规则进行设置。
```java
# 路由规则
Zuul:
	#ignored-patterns: /**/order/** # URL 地址排除，排除所有包含 /order/ 的路径
	ignored-services: order-service # 服务名称排除，多个服务逗号分隔，'*’排除所有
```
## 7.4、路由前缀
```java
Zuul:
	prefix: /api
```
## 7.5、网关过滤器
### 7.5.1、关键名词
类型: 定义路由流程中应用过滤器的阶段。共 pre、routing、 post、error 4 个类型
执行顺序: 在同类型中，定义过滤器执行的顺序。比如多个 pre 类型的执行顺序.
条件: 执行过滤器所需的条件。true 开启，false 关闭.
动作: 如果符合条件，将执行的动作。具体操作。

### 7.5.2、过滤器类型
 **pre**: 请求被路由到源服务器之前执行的过滤器
> 身份认证
> 选路由
> 请求日志

**routing**: 处理将请求发送到源服务器的过滤器。 
**post**: 响应从源服务器返回时执行的过滤器
> 对响应增加 HTTP 头
> 收集统计和度量指标
> 将响应以流的方式发送回客户端

**error**:上述阶段中出现错误时执行的过滤器

```java
@Component 
public class LoginFilter extends ZuulFilter{ 
    @Override 
    public String filterType() { 
        // 登录校验，肯定是在前置拦截 
        return "pre"; 
    }
 
    @Override 
    public int filterOrder() { 
        // 顺序设置为1 
        return 1; 
    }
 
    @Override 
    public boolean shouldFilter() { 
        // 返回true，代表过滤器生效。 
        return true; 
    }
 
    @Override 
    public Object run() throws ZuulException {
    //处理业务逻辑
    return null;
    }    
}
```
## 7.6、网关限流
这些情况都是无法预知的，不知道什么时候会有 10 倍甚至 20 倍的流量打进来，如果真碰上这种情况，扩容是根本来不及的

**计数器**：计算单元时间内访问接口的次数，如果达到次数，则限制访问

**漏桶算法**：漏桶是一个固定容量的桶，按照固定的速率流出，可以以任意的速率流入到漏桶中，超出了漏桶的容量就被丢弃

**令牌桶算法：**
（1）所有的请求在处理之前都需要拿到一个可用的令牌才会被处理;
（2）根据限流大小，设置按照一定的速率往桶里添加令牌;
（3）桶设置最大的放置令牌限制，当桶满时、新添加的令牌就被丢弃或者拒绝;
（4）请求到达后首先要获取令牌桶中的令牌，拿着令牌才可以进行其他的业务逻辑，处理完业务逻辑之后，将令牌直接删除:
（5）令牌桶有最低限额，当桶中的令牌达到最低限额的时候，请求处理完之后将不会删除令牌，以此保证足够的限流。

> 令牌桶算法是对漏桶算法的一种改进，漏桶算法能够限制请求调用的速率，而令牌桶算法能够在限制调用的平均速率的同时还允许一定程度的突发调用。在令牌桶算法中，存在一个桶，用来存放回定数量的令牌

### 7.6.1、网关限流

1. 全局限流配置

使用全局限流配置，zuul会对代理的所有服务提供限流保护
```properties
# 开启限流保护
zuul.ratelimit.enabled=true
# 60s内请求超过3次，服务端就抛出异常，60s后可以恢复正常请求
zuul.ratelimit.default-policy.limit=3
zuul.ratelimit.default-policy.refresh-interval=60
# 针对IP进行限流，不影响其他IP
zuul.ratelimit.default-policy.type=origin
```

2. 局部限流配置

使用局部限流配置，zuul仅针对配置的服务提供限流保护
```properties
# 开启限流保护
zuul.ratelimit.enabled=true
# hystrix-application-client服务60s内请求超过3次，服务抛出异常。
zuul.ratelimit.policies.hystrix-application-client.limit=3
zuul.ratelimit.policies.hystrix-application-client.refresh-interval=60
# 针对IP限流。
zuul.ratelimit.policies.hystrix-application-client.type=origin
```
# 8、sleuth（链路追踪）
## 8.1、链路追踪
单纯的理解链路追踪，就是指一次任务的开始到结束，期间调用的所有系统及耗时(时间跨度)都可以完整记录下来.
## 8.2、sleuth
Spring Cloud sleuth 为 Spring Cloud 实现了分布式跟踪解决方案。兼容 Zipkin，和其他基于日志的追踪系统，例如ELK (Elasticsearch 、 Logstash、Kibana)
Spring cloud sleuth 提供了以下功能:

- 辩路追踪: 通过 Sleuth 可以很清楚的看出一个请求都经过了那些服务，可以很方便的理清服务间的调用关系等。
- 性能分析 :通过 sleuth 可以很方便的看出每个采样请求的耗时，分析哪些服务调用比较耗时，当服务调用的耗时随着请求量的增大而增大时，可以对服务的扩容提供一定的提醒。
- 数据分析，优化链路 : 对于频繁调用一个服务，或并行调用等，可以针对业务做一些优化措施.
- 可视化错误 : 对于程序未捕获的异常，可以配合 Zipkin 查看。

## 8.3、专业术语
### 8.3.1、span
基本工作单位，一次单独的调用链可以称为一个 Span，Dapper 记录的是 Span 的名称，以及每个 Span 的 ID 和父ID，以重建在一次追踪过程中不同 Span 之间的关系


# 9、原理相关

## 9.1、如何保证分布式事务一致性？

1. 首先是设计方案尽可能规避分布式事务的场景（相似的业务放在一起，不要过度的拆分）
2. 根据业务场景，选择使用柔性事务（ap）还是强事务(cp)
    如果可以允许消息存在一段时间不一致，只要保证最终一致性，可以用本地消息表来做。如果要保证一致性，可以用2pc，具体实现方案有阿里的seata
3. 本地消息表（柔性事务）
不去同步的调用，先将要请求的消息插入到本地的消息表中，消息状态为正在处理，起一个定时任务去查询消息表，将正在处理的消息发送到消息队列，B中消息处理完后，向一个return队列发送一个成功的消息，A订阅了该消息队列，收到成功的消息后将状态该为处理完毕。（被调用方应保证幂等性，如库存系统在减库存前先查流水表，看该订单是否扣过库存，扣过就不执行）

![image.png](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202311150941880.png)

![image.png](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202311150942538.png)
