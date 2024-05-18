---
title: springBoot
date: 2022/11/06
categories:
  - coding
tags:
  - spring
  - 编程基础
  - springBoot
updated: 2022/11/13
abbrlink: 59868
---
# 1、Spring Boot基础应用

## 1.1、springBoot特征

**概念：**

约定优于配置，简单来说就是你所期待的配置与约定的配置一致，那么就可以不做任何配置，约定不符合期待时才需要对约定进行替换配置。

**特征：**

1.SpringBoot Starter：他将常用的依赖分组进行了整合，将其合并到一个依赖中，这样就可以一次性添加到项目的Maven或Gradle构建中

2.使编码变得简单，SpringBoot采用 JavaConfig的方式对Spring进行配置，并且提供了大量的注解，极大的提高了工作效率，比如@Configuration和@bean注解结合，基于@Configuration完成类扫描，基于@bean注解把返回值注入IOC容器。

3.自动配置：SpringBoot的自动配置特性利用了Spring对条件化配置的支持，合理地推测应用所需的bean并自动化配置他们

4.使部署变得简单，SpringBoot内置了三种Servlet容器，Tomcat，Jetty,undertow.我们只需要一个Java的运行环境就可以跑SpringBoot的项目了，SpringBoot的项目可以打成一个jar包

## 1.2、springBoot热部署

### 1.2.1、实现方式

通过引入spring-bootdevtools插件，可以实现不重启服务器情况下，对项目进行即时编译。引入热部署插件的步骤如下：

1.添加依赖

```xml
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-devtools</artifactId>  
    <scope>runtime</scope>  
</dependency>
```

2.设置开启自动编译

![idea自动编译|850](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202311020900514.png)

3.在项目任意页面中使用组合快捷键“Ctrl+Shift+Alt+/”打开Maintenance选项框，选中并打开Registry页面，列表中找到“compiler.automake.allow.when.app.running”，将该选项后的Value值勾选，用于指定IDEA工具在程序运行过程中自动编译

![image.png](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202311020902140.png)

4.项目配置中开启自动更新

![image.png](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202311030924388.png)


### 1.2.2、基本原理

基本原理就是我们在编辑器上启动项目，然后改动相关的代码，然后编辑器自动触发编译，替换掉历史的.class文件后，项目检测到有文件变更后会重启srpring-boot项目。内部主要是通过引入的插件对我们的classpath资源变化进行监听，当classpath有变化，才会触发重启。

从官方文档可以得知，其实这里对类加载采用了两种类加载器，对于第三方jar包采用baseclassloader来加载，对于开发人员自己开发的代码则使用restartClassLoader来进行加载，这使得比停掉服务重启要快的多，因为使用插件只是重启开发人员编写的代码部分。

# 2、注入(IOC)

## 2.1、属性注入
属性注入是通过在类中声明属性，并使用注解将属性值直接注入到相应的属性中。这种方式通常用于注入简单的值，如配置文件中的属性值或常量值
### 2.1.1、@Value值注入

参数值在application.properties配置文件中编写，并利用@Value注解注入

```java
@Configuration
public class JdbcConfiguration {

    @Value("${jdbc.url}")
    String url;
    @Value("${jdbc.driverClassName}")
    String driverClassName;
    @Value("${jdbc.username}")
    String username;
    @Value("${jdbc.password}")
    String password;


    @Bean   //用bean注解将返回值添加到容器中
    public DataSource dataSource() {
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setUrl(url);
        dataSource.setDriverClassName(driverClassName);
        dataSource.setUsername(username);
        dataSource.setPassword(password);
        return dataSource;
    }
}

```

### 2.1.2、@ConfigurationProperties批量注入
1.注入到该类

```java
@Data
@Component
@ConfigurationProperties(prefix = "person") //将配置文件中以person开头的属性注入到该类
public class Person {

    private int id; //id
    private String name;    //名称
    private List hobby;     // 爱好
    private String[] family;    // 家庭
    private Map map;
    private Pet pet;    // 宠物
}

person.id=1
person.name=tom
person.hobby=音乐,篮球,阅读
person.family=father,mother
person.map.k1=v1
person.map.k2=v2
person.pet.type=dog
person.pet.name=旺财

```

2.第三方配置

除了 @ConfigurationProperties 用于注释类之外，您还可以在公共 @Bean 方法上使用它。当要将属 性绑定到控件之外的第三方组件时，这样做特别有用
```java
@Data
public class AnotherComponent {
    private boolean enabled;
    private InetAddress remoteAddress;
}

@Configuration
public class MyService {
    @ConfigurationProperties("another")
    @Bean
    public AnotherComponent anotherComponent(){
        return new AnotherComponent();
    }
}

another.enabled=true
another.remoteAddress=192.168.10.11

```

## 2.2、依赖注入
依赖注入是通过在类中声明依赖关系，并由Spring容器负责在运行时将相应的依赖注入到类中。这种方式通常用于注入其他类的实例

### 2.2.1、属性注入

这里是使用 @Autowired 注解注入。另外也有 @Resource 以及 @Inject 等注解，都可以实现注入
```java
@Service
public class BService {
	@autowired
	Aservice aService;
	// ...
}

```

### 2.2.2、构造方法注入

如果类只有一个构造方法，那么 @Autowired 注解可以省略；如果类中有多个构造方法，那么需要添加上 @Autowired 来明确指定到底使用哪个构造方法
```java
@Service
public class Aservice {
	Bservice bService;
	@autowired
	public Aservice(BService bService){
		this.bService = bService;
	}
}
```


### 2.2.3、set方法注入

set 方法注入太过于臃肿，实际上很少使用
```java
@Service
public class Bservice {
	Aservice aService;
	@Autowired
	public void setaService(Aservice aService){
		this.aService = aService;
	}
}
```

> spring官方不建议第一种属性注入的方式。推荐使用构造方法注入的方式。
> （1）属性注入无法注入一个不可变的对象（final 修饰的对象）在 Java 中 final 对象（不可变）要么直接赋值，要么在构造方法中赋值，所以当使用属性注入 final 对象时，它不符合 Java 中 final 的使用规范，所以就不能注入成功了
> （2）更容易违背单一设计原则。使用属性注入是比较简单的,我们可以很容易的在一个类中注入多个对象,但是这些对象可能会违背原则,因为很多对象没有必要去注入


# 3、日志框架

日志的基本概念参考java core文章中的第13节
Spring Boot 默认已经使用了 SLF4J + LogBack . 所以我们在不进行任何额外操作的情况下就可以使用 SLF4J + Logback 进行日志输出。SLF4J 日志级别从小到大trace,debug,info,warn,error，默认是info级别。

|日志级别|描述|
|---|---|
|**trace**|较低的日志级别，通常不会被使用，日志的输出很详细|
|**debug**|程序员调式代码的时候使用，开发过程中打印一些运行信息|
|**info**|记录运维（程序运行）过程的数据|
|**warn**|警告信息，潜在的问题信息，在生产日志中，作为给程序员的一种提醒而使用|
|**error**|打印错误日志，但是不会影响程序继续运行|

## 3.1、application配置文件

可以指定指定包下的日志级别
```yml
# 日志配置  
logging:  
  level:  
    com.baidu: debug  # 设置指定路径下的日志输出级别
    org.springframework: warn
```

## 3.2、logback配置

SpringBoot项目默认使用logback，首先SpringBoot会从resource包下查找logback-test.xml或logback.xml ，如果这两个都不存在，则会调用BasicConfigurator，创建一个最小化的基本配置。最小化配置由一个关联到根logger的ConsoleAppender组成，默认输出模式为%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n，root logger级别为DEBUG，所以并不会生成日志文件，只会输出到控制台

通过自定义`logback.xml`配置文件来控制日志输出情况，通常我们会配置三个日志组件：

- 控制台输出
- 输出info级别日志文件
- 输出error级别日志文件

```xml
<!-- Logback configuration. See http:<span class="hljs-comment">//logback.qos.ch/manual/index.html --></span>
<configuration scan=<span class="hljs-string">"true"</span> scanPeriod=<span class="hljs-string">"2 seconds"</span>>
    <!--定义日志文件的存储地址-->
    <property name=<span class="hljs-string">"LOG_PATH"</span> value=<span class="hljs-string">"./logs"</span> />
    <!-- 控制台输出 -->
    <appender name=<span class="hljs-string">"STDOUT"</span> <span class="hljs-class"><span class="hljs-keyword">class</span></span>=<span class="hljs-string">"ch.qos.logback.core.ConsoleAppender"</span>>
        <encoder <span class="hljs-class"><span class="hljs-keyword">class</span></span>=<span class="hljs-string">"ch.qos.logback.classic.encoder.PatternLayoutEncoder"</span>>
            <!--格式化输出：%d表示日期，%-<span class="hljs-number">5l</span>evel：级别从左显示<span class="hljs-number">5</span>个字符宽度，%t表示线程名，%msg：日志消息，%n是换行符-->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} %-<span class="hljs-number">5l</span>evel ${PID:-} --- [%t] %logger{<span class="hljs-number">50</span>} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- info级别日志文件输出 -->
    <appender name=<span class="hljs-string">"INFO_FILE"</span> <span class="hljs-class"><span class="hljs-keyword">class</span></span>=<span class="hljs-string">"ch.qos.logback.core.rolling.RollingFileAppender"</span>>
        <!-- 日志文件输出的文件名 -->
        <File>${LOG_PATH}/info.log</File>
        <rollingPolicy <span class="hljs-class"><span class="hljs-keyword">class</span></span>=<span class="hljs-string">"ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy"</span>>
            <!-- 每日生成日志文件或日志文件大小超出限制后输出的文件名模板 -->
            <fileNamePattern>${LOG_PATH}/info-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <!-- 日志文件保留天数 -->
            <maxHistory><span class="hljs-number">30</span></maxHistory>
            <!-- 日志文件最大大小：<span class="hljs-number">100</span>MB -->
            <maxFileSize><span class="hljs-number">100</span>MB</maxFileSize>
        </rollingPolicy>
        <encoder <span class="hljs-class"><span class="hljs-keyword">class</span></span>=<span class="hljs-string">"ch.qos.logback.classic.encoder.PatternLayoutEncoder"</span>>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} %-<span class="hljs-number">5l</span>evel ${PID:-} --- [%t] %logger{<span class="hljs-number">50</span>} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- error级别日志文件输出 -->
    <appender name=<span class="hljs-string">"ERROR_FILE"</span> <span class="hljs-class"><span class="hljs-keyword">class</span></span>=<span class="hljs-string">"ch.qos.logback.core.rolling.RollingFileAppender"</span>>
        <!-- 日志输出级别，优先级 > <span class="hljs-string">'<root level>'</span> -->
        <filter <span class="hljs-class"><span class="hljs-keyword">class</span></span>=<span class="hljs-string">"ch.qos.logback.classic.filter.ThresholdFilter"</span>>
            <level>ERROR</level>
        </filter>
        <File>${LOG_PATH}/error.log</File>
        <rollingPolicy <span class="hljs-class"><span class="hljs-keyword">class</span></span>=<span class="hljs-string">"ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy"</span>>
            <fileNamePattern>${LOG_PATH}/error-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <maxHistory><span class="hljs-number">30</span></maxHistory>
            <maxFileSize><span class="hljs-number">100</span>MB</maxFileSize>
        </rollingPolicy>
        <encoder <span class="hljs-class"><span class="hljs-keyword">class</span></span>=<span class="hljs-string">"ch.qos.logback.classic.encoder.PatternLayoutEncoder"</span>>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} %-<span class="hljs-number">5l</span>evel ${PID:-} --- [%t] %logger{<span class="hljs-number">50</span>} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- 默认日志输出级别 -->
    <root level=<span class="hljs-string">"INFO"</span>>
        <appender-ref ref=<span class="hljs-string">"STDOUT"</span> />
        <appender-ref ref=<span class="hljs-string">"INFO_FILE"</span> />
        <appender-ref ref=<span class="hljs-string">"ERROR_FILE"</span> />
    </root>

</configuration>
```

# 4、多环境配置

Spring Boot 自己本身带的有多环境配置，对多环境整合已经有了很好的支持，能够在打包，运行期间自由切换环境

## 4.1、springBoot多环境配置
### 4.1.1、创建不同环境配置文件

不同环境的配置文件需要进行分开，按照项目运行环境启用加载。新建 application-dev.yml, application-test.yml, application-prod.yml。加上 application.yml 一共有四个配置文件。注意：配置文件名称一定要是 application-name.yml 格式，name可以自定义

### 4.1.2、指定不同环境配置文件
在 application.yml 文件中指定启用哪个环境的配置文件
```yml
# 指定启用环境为 开发环境 dev
spring:
  profiles:
    active: dev
```

> 如果没有指定运行的环境，Spring Boot 会默认加载 application.yml 配置文件
### 4.1.3、运行jar时指定配置文件

```
java -jar xxx.jar --spring.profiles.active=dev
```


## 4.2、maven多环境配置
Maven 也提供了对多环境的支持，不仅仅支持 Spring Boot 项目，只要是基于 Maven 的项目都可以配置。Maven 对于多环境的支持在功能方面更加强大，支持 JDK 版本、资源文件、操作系统等等
### 4.2.1、创建不同环境配置文件
同4.1.1
### 4.2.2、定义激活的变量
需要将 Maven 激活的环境作用于 Spring Boot，实际还是利用了 spring.profiles.active 这个属性，只是现在这个属性的取值将是取值于 Maven，配置如下:
```properties
spring.profiles.active=@profile.active@
```

profile.active 实际上就是一个变量，在 maven 打包的时候指定的 -P dev 传入的就是值。

### 4.2.3、pom文件中定义profiles
```xml
<!--定义三种开发环境-->
<profiles>
    <profile>
        <!--不同环境的唯一id-->
        <id>dev</id>
        <activation>
            <!--默认激活开发环境-->
            <activeByDefault>true</activeByDefault>
        </activation>
        <properties>
            <!--profile.active对应application.yml中的@profile.active@-->
            <profile.active>dev</profile.active>
        </properties>
    </profile>
    <!--测试环境-->
    <profile>
        <id>test</id>
        <properties>
            <profile.active>test</profile.active>
        </properties>
    </profile>
    <!--生产环境-->
    <profile>
        <id>prod</id>
        <properties>
            <profile.active>prod</profile.active>
        </properties>
    </profile>
</profiles>

```

### 4.2.4、打包时指定环境
```
mvn clean package -Pdev -DskipTests
```


# 5、事务实现

## 5.1、事务实现方案

Spring 事务管理分为**编码式和声明式**的两种方式

编程式事务管理： 利用TransactionTemplate模板通过编程的方式实现事务管理,而无需关注资源获取、复用、释放、事务同步及异常处理等操作

声明式事务管理： 建立在AOP之上的。其本质是对方法前后进行拦截，然后在目标方法开始之前创建或者加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务（使用**isolation**属性声明事务的隔离级别,使用**propagation**属性声明事务的传播机制）

> 声明式事务管理不需要入侵代码，更快捷而且简单，推荐使用

## 5.2、@Transactional详解

### 5.2.1、常用参数

|**参 数 名 称**|**功 能 描 述**|
|---|---|
|readOnly|该属性用于设置当前事务是否为只读事务，设置为true表示只读，false则表示可读写，默认值为false。例如：`@Transactional(readOnly=true)`|
|rollbackFor|rollbackFor 该属性用于设置需要进行回滚的异常类数组，当方法中抛出指定异常数组中的异常时，则进行事务回滚。例如：指定单一异常类：@Transactional(rollbackFor=RuntimeException.class)指定多个异常类：@Transactional(rollbackFor={RuntimeException.class,Exception.class})|
|**propagation**|该属性用于设置事务的传播行为。例如：`@Transactional(propagation=Propagation.NOT_SUPPORTED, readOnly=true)`|
|**isolation**|该属性用于设置底层数据库的事务隔离级别，事务隔离级别用于处理多事务并发的情况，通常使用数据库的默认隔离级别即可，基本不需要进行设置|
|timeout|该属性用于设置事务的超时秒数，默认值为-1表示永不超时 事物超时设置：`@Transactional(timeout=30)` ，设置为30秒|

### 5.2.2、事务传播行为

Spring在TransactionDefinition接口中规定了7种类型的事务传播行为。Propagation枚举则引用了这些类型，开发过程中我们一般直接用Propagation枚举。例如@Transactional(propagation=Propagation.NOT_SUPPORTED,readOnly=true)，常用的三项已经加粗


|**事务传播行为类型**|**说明**|
|---|---|
|PROPAGATION_SUPPORTS|支持事务。若当前没有事务以非事务方式执行；若当前有事务，加入此事务中|
|PROPAGATION_NOT_SUPPORTED|不支持事务。若当前存在事务，把当前事务挂起，然后运行方法|
|PROPAGATION_NEVER|不使用事务。若当前方法存在事务，则抛出IllegalTransactionStateException异常，否则继续使用无事务机制运行|
|PROPAGATION_MANDATORY|强制使用事务。若当前有事务，就使用当前事务；若当前没有事务，抛出IllegalTransactionStateException异常|
|**PROPAGATION_REQUIRED**|需要事务（**默认**）。若当前无事务，新建一个事务；若当前有事务，加入此事务中|
|**PROPAGATION_REQUIRES_NEW**|新建事务。无论当前是否有事务，都新建事务运行|
|**PROPAGATION_NESTED**|嵌套。如果当前存在事务，则在嵌套事务内执行；如果当前没有事务，则执行与PROPAGATION_REQUIRED类似的操作|

### 5.2.3、事务隔离级别
@Transactional(isolation = Isolation.DEFAULT)，默认的隔离级别，使用数据库默认的事务隔离级别，下面四个与JDBC的隔离级别相对应

| **级别** | **名字** | **隔离级别** | **脏读** | **不可重复读** | **幻读** | **数据库默认隔离级别** | 解释 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | 读未提交 | read uncommitted | 是 | 是 | 是 |  | 事务A可以读取到事务B未提交的数据 |
| 2 | 读已提交 | read committed | 否 | 是 | 是 | Oracle和SQL Server | 事务A只能读取其它事务已提交的数据（避免了脏读） |
| 3 | 可重复读 | repeatable read | 否 | 否 | 是 | MySQL | 保证在同一个事务中多次读取同样数据的结果是一样的 |
| 4 | 串行化 | serializable | 否 | 否 | 否 |  | 事务串行化顺序执行 |

## 5.3、事务使用事项

1.@Transactional 注解应该只被应用在 public 修饰的方法上(注意)。 如果在 protected、private 或者 package-visible 的方法上使用 该注解，它也不会报错(IDEA会有提示)， 但事务并没有生效

2.@Transactional是基于动态代理的(注意)，需要一个类调用另一个类，类内调用会失效

> 当在类的内部调用被@Transactional注解修饰的方法时，实际上是通过类的实例直接调用方法，而不是通过代理对象。这样做会绕过代理对象，从而导致@Transactional注解失效

3.事务@Transactional由spring控制时，它会在抛出异常的时候进行回滚。如果自己使用try-catch捕获处理了，是不生效的。如果想事务生效可以进行手动回滚或者在catch里面将异常抛出throw new RuntimeException()

# 6、starter （自动配置原理）

spring boot 在配置上相比spring要简单许多, 其核心在于spring-boot-starter, 在使用spring boot来搭建一个项目时, 只需要引入官方提供的starter, 就可以直接使用, 免去了各种配置。starter简单来讲就是做了起步依赖和自动配置

> Spring官方提供了很多starter，第三方也可以定义starter。为了加以区分，starter从名称上进行了如下规范：
  -Spring官方提供的starter名称为：spring-boot-starter-xxx 例如Spring官方提供的spring-boot-starter-web
  -第三方提供的starter名称为：xxx-spring-boot-starter 例如由mybatis提供的mybatis-spring-boot-starter

## 6.1、起步依赖
起步依赖，其实就是将具备某种功能的坐标打包到一起，可以简化依赖导入的过程。例如，我们导入spring-boot-starter-web这个starter，则和web开发相关的jar包都一起导入到项目中了。如下图所示：

![起步依赖](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202311031115469.png)

## 6.2、自动配置

>spring boot默认扫描启动类所在的包下的主类与子类的所有组件，但并没有包括依赖包中的类，那么依赖包中的bean是如何被发现和加载的。我们需要从Spring Boot项目的启动类开始跟踪

在启动类上我们一般会加入SpringBootApplication注解，它是一个复合注解，它下面的@EnableAutoConfiguration这个注解，这个注解也是一个复合注解。@EnableAutoConfiguration注解导入AutoConfigurationImportSelector类，这个类中有一个selectImports()方法，扫描了所有包含META-INF/spring.factories的jar包，这些jar包实际上就是我们引入的starter中的依赖。而自动配置的类的全类名就在这个文件中，然后就可以通过反射加载这些类

这些配置类@Configuration和@Bean这两个注解一起使用就可以创建一个基于java代码的配置类.比如**MybatisAutoConfiguration**这个类，会先通过依赖条件判断@ConditionOnClass等，判断有没有SqlSessionFactory类和Datasource实例。满足条件时创建对应的需要的实例

![image.png](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202311031446904.png)


## 6.3、自定义starter

SpringBoot中的starter是一种非常重要的机制，能够抛弃以前繁杂的配置，将其统一集成进starter，应用者只需要在maven中引入starter依赖，Spring Boot就能自动扫描各个jar包下classpath路径的spring.factories文件，加载自动配置类信息，加载相应的bean信息并启动相应的`默认配置`

### 6.3.1、创建starter项目

SpringBoot官方命名方式
格式：spring-boot-starter-{模块名}
举例：spring-boot-starter-web
自定义命名方式
格式：{模块名}-spring-boot-starter
举例：mystarter-spring-boot-starter


### 6.3.2、添加依赖

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-autoconfigure</artifactId>
   <optional>2.2.9.RELEASE</optional>
</dependency>
```

### 6.3.3、编写bean
```java
//@Component
@EnableConfigurationProperties(MyBean .class)
@ConfigurationProperties(prefix = "mybean")
public class MyBean {
	private int id;private String name;//省略setter和getter
}
```

### 6.3.4、编写自动配置类

```java
@Configuration
public class MyBeanAutoConfiguration {
	@Bean
	public MyBean myBean(){
		return new MyBean();
	}
}
```


### 6.3.5、编写spring.factories

```txt
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\ com.blue.config.MyBeanAutoConfiguration
```


### 6.3.6、打包

```txt
mvn clean install
```

### 6.3.7、测试

```java
<dependency>
	<groupId>com.blue</groupId>
	<artifactId>mybean-spring-boot-starter</artifactId>
	<version>1.0-SNAPSHOT</version>
</dependency>

@RunWith(SpringRunner.class)
@SpringBootTest(classes = TestApplication.class)
class TestApplicationTests {
	@Autowiredprivate 
	MyBean ba;
	
	@Testvoid contextLoads() {
		System.out.println(ba.getName());
	}
}
```

### 6.3.8、实现热插拔
```java
// 1. 新增标记类
public class ConfigMarker {
}

// 2.定义@EnableMyBean注解
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Import({ConfigMarker.class})
public @interface EnableMyBean{}

// 3.改造配置类
@Configuration
@ConditionalOnBean(ConfigMarker.class)
public class MyBeanAutoConfiguration {
	@Beanpublic 
	MyBean myBean(){
		return new MyBean();
	}
}

// 在测试启动类上开启MyBean功能
@EnableMyBean
@SpringBootApplication
public class TestApplication {
	public static void main(String[] args) {
		SpringApplication.run(TestApplication.class, args);
	}
}
```


## 6.4、springBoot内置tomcat原理

1、springboot启动时会自动加载spring-boot-starter-web依赖中的配置类：servletWebServerFactoryAutoConfiguration  
2、该自动配置类通过@Import导入了可用的web容器工厂（默认tomcat，通过@ConditionalOnClass判断决定使用哪个）

```java
// 不使用默认tomcat，使用jetty容器时的配置
// 1. application配置文件
spring:
	main:
		web-application-type: SERVLET
	servlet:
		container: org.eclipse.jetty.server.Server
// 2.maven配置文件
<dependencies>
    <!-- Jetty容器 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <exclusions>
            <exclusion>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-tomcat</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>org.eclipse.jetty</groupId>
        <artifactId>jetty-server</artifactId>
        <version>${jetty.version}</version>
    </dependency>
</dependencies>
```


3、在调用run()方法的时候完成Tomcat对象的创建，环境设置和启动，从而实现Tomcat容器的自动化处理

# 7、spring Bean的生命周期
## 7.1、spring bean声明周期

![spring bean生命周期](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202307230927436.png)


1、 实例化（Instantiation）：当Spring容器接收到Bean的定义时，会使用反射机制创建一个Bean实例。
2、属性赋值（Populate Bean）： Spring 将值和bean的引用注入到bean对应的属性中
3、回调实现Aware接口的方法。BeanNameAware，BeanFactoryAware，ApplicationContextAware对应的方法。

> Spring的依赖注入的最大亮点就是你所有的Bean对Spring容器的存在是没有意识的。即你可以将你的容器替换成别的容器，例如Goggle Guice,这时Bean之间的耦合度很低。
> 但是在实际的项目中，我们不可避免的要用到Spring容器本身的功能资源，这时候Bean必须要意识到Spring容器的存在，才能调用Spring所提供的资源，这就是所谓的Spring Aware。其实Spring Aware本来就是Spring设计用来框架内部使用的，若使用了Spring Aware，你的Bean将会和Spring框架耦合。  

4、初始化（Initialization）：分别调用（1）BeanPostProcessor的前置处理器，（2）InitialzingBean的afterPropertiesSet（）方法，（3）调用init初始化方法（4）调用BeanPostProcessor的后置处理器
5、使用（In Use）：在初始化完成之后，Bean就可以被使用了。
6、销毁（Destruction）：当Spring容器关闭时，会销毁所有的Bean。在销毁Bean之前，（1）如果实现DisposableBean接口，Spring将调用它的destory()接口方法（2）如果bean使用destroy-method声明了销毁方法，该方法也会被调用

# 8、拦截器和过滤器

在构建 Web 应用时，我们经常需要对请求进行拦截和处理，以实现诸如身份验证、授权、日志记录等功能。在 Spring Boot 中，为我们提供了两种强大的工具来实现这些功能：过滤器（Filter）和拦截器（Interceptor）

>拦截器与过滤器的区别
 归属不同：Filter属于Servlet技术，Interceptor属于SpringMVC技术
 拦截器内容不同：Filter对所有访问进行增强（在Tomcat服务器进行配置），Interceptor仅针对SpringMVC的访问进行增强
## 8.1、拦截器

在Spring Boot中，拦截器分为两类：  
一种是对请求进来的url进行拦截，HandlerInterceptor接口
一种是对发送出去的请求进行拦截，ClientHttpRequestInterceptor

### 8.1.1、HandlerInterceptor接口

1、创建一个类，实现org.springframework.web.servlet.HandlerInterceptor接口

```java
/**
 * 登录检查
 * 1.配置到拦截器要拦截哪些请求
 * 2.把这些配置放在容器中
 */
public class LoginInterceptor implements HandlerInterceptor {
    /**
     * 目标方法执行之前
     * 登录检查写在这里，如果没有登录，就不执行目标方法
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        // 获取进过拦截器的路径
        String requestURI = request.getRequestURI();
        // 登录检查逻辑
        HttpSession session = request.getSession();
        Object loginUser = session.getAttribute("loginUser");
        if(loginUser !=null){
            // 放行
            return true;
        }
        // 拦截   就是未登录,自动跳转到登录页面，然后写拦截住的逻辑
        return false;
    }
 
    /**
     * 目标方法执行完成以后
     */
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        HandlerInterceptor.super.postHandle(request, response, handler, modelAndView);
    }
 
    /**
     * 页面渲染以后
     */
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        HandlerInterceptor.super.afterCompletion(request, response, handler, ex);
    }
}
```

2、将拦截器添加到容器当中

```java
@Configuration
//定制SpringMVC的一些功能都使用WebMvcConfigurer
public class AdminWebConfig implements WebMvcConfigurer {
 
    /**
     * 配置拦截器
     * @param registry 相当于拦截器的注册中心
     */
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
		// 下面这句代码相当于添加一个拦截器   添加的拦截器就是我们刚刚创建的
         registry.addInterceptor(new LoginInterceptor())
				//  addPathPatterns()配置我们要拦截哪些路径 addPathPatterns("/**")表示拦截所有请求，包括我们的静态资源
                 .addPathPatterns()
				// excludePathPatterns()表示我们要放行哪些（表示不用经过拦截器）
				// excludePathPatterns("/","/login")表示放行“/”与“/login”请求
				// 如果有静态资源的时候可以在这个地方放行
                 .excludePathPatterns("/","/login");
    }
}
```

### 8.1.2、ClientHttpRequestInterceptor接口
对于服务间调用，你可以使用ClientHttpRequestInterceptor接口来拦截和处理发送出去的请求。
这通常用于处理微服务之间的通信，例如添加认证信息、自定义请求头等

1、创建一个类，实现org.springframework.http.client.ClientHttpRequestInterceptor接口

```java
public class CustomHeaderInterceptor implements ClientHttpRequestInterceptor {    
	@Override    
	public ClientHttpResponse intercept(HttpRequest request, byte[] body, ClientHttpRequestExecution execution)  throws IOException {                       request.getHeaders().set("Custom-Header", "CustomHeaderValue");        
		return execution.execute(request, body);   
	}}
```

2、创建一个配置类，将拦截器添加到RestTemplate或WebClient中。这两个类都是Spring框架中用于发起HTTP请求的客户端
```java
@Configurationpublic 
class RestTemplateConfig {    
	@Bean    
	public RestTemplate restTemplate() { 
		RestTemplate restTemplate = new RestTemplate();        
		List<ClientHttpRequestInterceptor> interceptors = new ArrayList<>();        
		interceptors.add(new CustomHeaderInterceptor());        
		restTemplate.setInterceptors(interceptors);        
		return restTemplate;    
	}
}
```

### 8.1.3、两者区别

| 特征      | HandlerInterceptor                   | ClientHttpRequestInterceptor        |
|:--------|:-------------------------------------|:------------------------------------|
| 作用范围    | 拦截收到的Http请求                          | 拦截使用RestTemplate或WebClient发送的HTTP请求 |
| 使用场景    | 身份验证，授权，日志记录                         | 添加认证信息，自定义请求头                       |
| 需要实现的方法 | preHandle、postHandle、afterCompletion | intercept                           |  


## 8.2、过滤器
在 Spring Boot 中，过滤器（Filter）是用于在 Servlet 容器级别拦截和处理 HTTP 请求的组件。它们通常用于实现诸如身份验证、授权、日志记录、请求和响应的数据转换等功能。过滤器位于整个请求处理链的最前端，因此在请求到达 Spring 应用的任何其他组件之前，都会先经过过滤器处理

1、注册过滤器

注册过滤器：将过滤器作为一个 Bean 注册到 Spring 应用中。以下是一个简单的过滤器示例，用于记录每个请求的处理时间
```java
@WebFilter(urlPatterns = "/*")
public class RequestTimingFilter implements Filter {

	@Override    
	public void init(FilterConfig filterConfig) {}    
	
	@Override    
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)  throws IOException, ServletException {                       long startTime = System.currentTimeMillis();        
		try {            
			chain.doFilter(request, response);        
		} finally {            
			long endTime = System.currentTimeMillis();            
			long duration = endTime - startTime;            
			HttpServletRequest httpRequest = (HttpServletRequest) request;            
			System.out.println(String.format("%s %s took %d ms", httpRequest.getMethod(), httpRequest.getRequestURI(), duration));        
		}   
	}    
	
	@Override    
	public void destroy() {}
	
	}
```

2、要将此过滤器添加到 Spring Boot 应用中，请将其注册为 Bean

```java
@Configurationpublic 
class FilterConfig {    
	@Bean    
	public FilterRegistrationBean<RequestTimingFilter> requestTimingFilter() {        
		FilterRegistrationBean<RequestTimingFilter> registrationBean = new FilterRegistrationBean<>();        
		registrationBean.setFilter(new RequestTimingFilter());        
		registrationBean.addUrlPatterns("/*");        
		return registrationBean;    
	}
}
```

## 8.3、两者之间的关系

| 特征   | 过滤器                 | 拦截器                             |
|:-----|:--------------------|:--------------------------------|
| 处理层级 | servlet容器级别         | spring mvc层级                    |
| 使用场景 | 通用功能，如身份验证，授权，日志记录等 | 与spring框架相关的功能，如身份验证，请求参数处理     |
| 优势   | 处理任何web应用请求，更大的控制范围 | 集成于springmvc，可以访问spring上下文和其它组件 |  

![image.png|525](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202311061524568.png)


# 9、监听器

SpringBoot 的监听器多用于监听Web应用中某些对象，信息的创建、销毁、增加、修改、删除等动作的发生，然后做出响应的处理。的那个范围对象状态发生变化时，服务器自动调用监听器对象的方法，  
使用场景：

- 系统统计在线用户
- 系统加载时进行信息初始化
- 系统网站的访问量 
- 等.....

## 9.1、监听器分类

Spring的监听器也可以说是一种观察者模式，它能实现事件与事件监听者直接的解耦，在Spring中监听器的实现主要有一下重要组件：

- ApplicationListener：事件监听者，观察者；
- ApplicationEvent：Spring 事件，记录事件源、事件内容、时间等数据；
- @EventListener：除了实现ApplicationListener接口注册监听器，也可以使用注解的方式
- ApplicationEventPublisher：发布事件；

## 9.2、事件监听方式

### 9.2.1、实现ApplicationListener接口

```java
@Component
public class MyListener2 implements ApplicationListener<MyEvent>{
	Logger logger = Logger.getLogger(MyListener2.class);
	
	public void onApplicationEvent(MyEvent event){
		logger.info(String.format("%s监听到事件源：%s.", MyListener2.class.getName(), event.getSource()));
	}
}

```

### 9.2.2、通过@EventListener注解实现事件监听

```java
@Component
public class MyListener4{
	Logger logger = Logger.getLogger(MyListener4.class);
	
  // @EventListener 注解支持根据Event参数类型进行匹配
	@EventListener
	public void listener(MyEvent event){
		logger.info(String.format("%s监听到事件源：%s.", MyListener4.class.getName(), event.getSource()));
	}
}
```


# 10、原理相关知识

## 10.1、springmvc执行流程

1.客户端发送请求到DispatcherServlet。
2.DispatcherServlet根据请求的URL找到对应的HandlerMapping。
3.HandlerMapping返回对应的HandlerExecutionChain，其中包含处理请求的Controller以及拦截器。
4.DispatcherServlet根据返回的HandlerExecutionChain取出对应的Controller，并调用其处理方法。
5.Controller处理请求，并返回一个ModelAndView对象。
6.DispatcherServlet根据返回的ModelAndView对象，调用对应的ViewResolver进行视图解析。
7.ViewResolver返回对应的View对象。DispatcherServlet将Model传递给View，View对Model进行渲染。
8.DispatcherServlet将渲染后的视图返回给客户端

## 10.2、spring解决循环依赖问题

| **缓存**                | **说明**                                                |
| --------------------- | ----------------------------------------------------- |
| singletonObjects      | 第一级缓存，存放可用的成品Bean。                                    |
| earlySingletonObjects | 第二级缓存，存放半成品的Bean，半成品的Bean是已创建对象，但是未注入属性和初始化。用以解决循环依赖。 |
| singletonFactories    | 第三级缓存，存的是Bean工厂对象，用来生成半成品的Bean并放入到二级缓存中。用以解决循环依赖。     |
|                       |                                                       |

1. A 调用doCreateBean()创建Bean对象：由于还未创建，从第1级缓存singletonObjects查不到，此时只是一个半成品（提前暴露的对象），放入第3级缓存singletonFactories。
2. A在属性填充时发现自己需要B对象，但是在三级缓存中均未发现B，于是创建B的半成品，放入第3级缓存singletonFactories。
3. B在属性填充时发现自己需要A对象，从第1级缓存singletonObjects和第2级缓存earlySingletonObjects中未发现A，但是在第3级缓存singletonFactories中发现A，将A放入第2级缓存earlySingletonObjects，同时从第3级缓存singletonFactories删除。
4. 将A注入到对象B中。
5. B完成属性填充，执行初始化方法，将自己放入第1级缓存singletonObjects中（此时B是一个完整的对象），同时从第3级缓存singletonFactories和第2级缓存earlySingletonObjects中删除。
6. A得到“对象B的完整实例”，将B注入到A中。
7. A完成属性填充，执行初始化方法，并放入到第1级缓存singletonObjects中。
