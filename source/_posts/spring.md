---
title: spring
date: 2023/07/22
categories:
  - coding
tags:
  - spring
abbrlink: 9964
---
# 1、前置内容
## 1.1、EJB的问题
它是一个重量级的框架，体现在：

1. 运行环境苛刻：需要运行在EJB容器（weblogic  websphere），需要收费，并且容器代码闭源
2. 代码移植性差
## 1.2、什么是spring
spring是一个轻量级的javaEE解决方案，整合了众多优秀的设计模式

- 轻量级

> 1. 对于运行环境没有额外要求（tomcat jetty weblogic等都可以）
> 2. 代码一致性高（不需要实现额外接口）


- javaEE解决方案

> 包含了java web开发中 controller service dao层的解决方案


- 整合设计模式

> 工厂
> 代理
> 模板
> 策略

## 1.3、工厂设计模式
### 1.3.1、什么是工厂设计模式
```markdown
1. 概念：通过工厂类创建对象
2. 好处：解耦合
   问题：不利于代码维护
```
### 1.3.2、通用工厂的设计（简单工厂+反射）
```java
// 工厂类
public class BeanFactory{
    private static Properties env = new Properties();
    static {
        InputStream inputStream = BeanFactory.class.getResourceAsStream("/applicationContext.properties");
        try {
            env.load(inputStream);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    public static Object getBean(String key){
        Object ret = null;
        try{
           Class clazz = Class.forName(env.getProperty("userBean"));
           ret = clazz.newInstance();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return ret;
    }
}
```
# 2、spring IOC
## 2.1、第一个spring程序
### 2.1.1、核心API

- ApplicationContext

```markdown
1. 作用：屏蔽实现的差异

2. 好处：解耦合

3. 接口类型：
非web环境：ClassPathXmlApplicationContext
web环境：XmlWebApplicationContext（需要导入spring-webmvc）

4. 重量级资源
applicationContext工厂的对象占用大量内存
不会频繁的创建，一个应用程序只会创建一个工厂对象
所以applicationContext工厂一定是线程安全的
```
### 2.1.2、程序开发

1. 引入依赖

```xml
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.1.4.RELEASE</version>
        </dependency>
```

2. 创建配置文件

new - xmlConfiguration File - spring config
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="person" class="bean.Person"/>
</beans>
```

3. 通过工厂获取实例

```java
public class Demo {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("/applicationContext.xml");
        Person person = (Person)ctx.getBean("person");
        System.out.println(person);
    }
}
```

### 2.1.3、思考

1. 使用Spring工厂创建对象时，Spring会利用反射机制来实例化对象，并且会调用对象的构造方法来完成对象的初始化

2. 在开发中，并不是所有的对象都会交给Spring工厂来创建。通常来说，我们会将那些需要频繁创建和管理的对象交给Spring容器来管理和创建，而对于那些只需要创建一次或者创建次数很少的对象，我们可以直接使用new关键字来实例化对象。并且实体对象是不会交给spring创建的，它是由持久层框架进行创建（当我们调用 MyBatis 的查询方法时，MyBatis 会根据 SQL 语句和结果集映射关系来查询数据库，并将查询结果映射到实体对象中。在这个过程中，MyBatis 会利用 Java 的反射机制来动态创建实体对象，并调用实体对象的 setter 方法来设置属性的值）


## 2.2、spring与日志框架整合
spring与日志框架整合，日志框架就可以在控制台中，输出spring框架运行过程中的一些重要信息

![日志框架](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202307230929873.png)


- spring如何整合日志框架？

> spring 1,2,3 早期都是jcl
spring4.x开始使用slf4j，默认整合的日志框架logback 或 log4j2

当然spring5中我们也可以不采用默认，而去整合我们熟悉的log4j
1. pom

```xml
		<!--日志门面-->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>1.7.25</version>
        </dependency>
        <!--log4j-->
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.17</version>
        </dependency>
```

2. log4j.properties

```properties
### 配置根
log4j.rootLogger = debug, console

### 配置输出到控制台
log4j.appender.console = org.apache.log4j.ConsoleAppender
log4j.appender.console.Target = System.out
log4j.appender.console.layout = org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern =  %d{yyyy-MM-dd HH:mm:ss} %-5p %c{1}:%L - %m%n
```
## 2.3、注入
### 2.3.1、什么是注入？
通过spring工厂及配置文件，为所创建对象的成员变量赋值

### 2.3.2、为什么需要注入？
之前我们为成员变量赋值是通过set方法，但这种方式会产生耦合

### 2.3.3、如何进行注入？
#### 2.3.3.1、属性（field ）注入
所谓基于 field 注入，就是在bean的变量上使用注解进行依赖注入。本质上是通过反射的方式直接注入到field。这是我平常开发中看的最多也是最熟悉的一种方式，同时，也正是 Spring 团队所不推荐的方式（容易违背了单一职责原则：使用这种基于 field 注入的方式，添加依赖是很简单的，就算你的类中有十几个依赖你可能都觉得没有什么问题，普通的开发者很可能会无意识地给一个类添加很多的依赖）

```java
@Autowired
private Svc svc;
```

#### 2.3.3.2、setter注入
通过对应变量的`setXXX()`方法以及在方法上面使用注解，来完成依赖注入。比如：
```java
private Helper helper;

@Autowired
public void setHelper(Helper helper) {
    this.helper = helper;
}
```
#### 2.3.3.3、构造方法注入
将各个必需的依赖全部放在带有注解构造方法的参数中，并在构造方法中完成对应变量的初始化，这种方式，就是基于构造方法的注入。比如：
```java
private final Svc svc;

@Autowired
public HelpService(@Qualifier("svcB") Svc svc) {
    this.svc = svc;
}
```

## 2.4、spring对象的生命周期
### 2.4.1、spring bean的生命周期

![spring bean生命周期](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202307230927436.png)


1. 实例化（Instantiation）：当Spring容器接收到Bean的定义时，会使用反射机制创建一个Bean实例。
2. 属性赋值（Populate Bean）： Spring 将值和bean的引用注入到bean对应的属性中
3. 回调实现Aware接口的方法。BeanNameAware，BeanFactoryAware，ApplicationContextAware对应的方法。

> Spring的依赖注入的最大亮点就是你所有的Bean对Spring容器的存在是没有意识的。即你可以将你的容器替换成别的容器，例如Goggle Guice,这时Bean之间的耦合度很低。
> 但是在实际的项目中，我们不可避免的要用到Spring容器本身的功能资源，这时候Bean必须要意识到Spring容器的存在，才能调用Spring所提供的资源，这就是所谓的Spring Aware。其实Spring Aware本来就是Spring设计用来框架内部使用的，若使用了Spring Aware，你的Bean将会和Spring框架耦合。  

1. 初始化（Initialization）：分别调用（1）BeanPostProcessor的前置处理器，（2）InitialzingBean的afterPropertiesSet（）方法，（3）调用init初始化方法（4）调用BeanPostProcessor的后置处理器
2. 使用（In Use）：在初始化完成之后，Bean就可以被使用了。
3. 销毁（Destruction）：当Spring容器关闭时，会销毁所有的Bean。在销毁Bean之前，（1）如果实现DisposableBean接口，Spring将调用它的destory()接口方法（2）如果bean使用destroy-method声明了销毁方法，该方法也会被调用


# 3、spring AOP

## 3.1、spring动态代理
### 3.1.1、spring动态代理概念

1. 概念：通过代理类为目标类增加额外功能
2. 好处：利于目标类的维护
### 3.1.2、搭建开发环境

```xml
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aop</artifactId>
            <version>5.1.14.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjrt</artifactId>
            <version>1.8.8</version>
        </dependency>
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.8.3</version>
        </dependency>
```
### 3.1.3、spring动态代理开发步骤（MethodBeforeAdvice）

1. 创建目标对象

```java
public class UserServiceImpl implements UserService{
    public void login() {
        System.out.println("user login...");
    }
}

<bean id="userService" class="service.UserServiceImpl"></bean>
```

2. 额外功能

MethodBeforeAdvice接口


```java
public class Before implements MethodBeforeAdvice {
    // 参数： 目标方法 目标方法参数 目标对象
    public void before(Method method, Object[] objects, Object o) throws Throwable {
        System.out.println("--method before advice log--");
    }
}

<bean id="before" class="dynamic.Before"></bean>
```

3. 定义切入点

 切入点：额外功能加入的位置(方法)
```java
    <aop:config>
        <!--所有方法，都作为切入点，加入额外功能-->
        <aop:pointcut id="pc" expression="execution(* *(..))"/>
    </aop:config>
```

4. 组装（2 3步整合）

```java
    <aop:config>
        <!--所有方法，都作为切入点，加入额外功能-->
        <aop:pointcut id="pc" expression="execution(* *(..))"/>
        <!--组装：目的把切入点和额外功能进行整合-->
        <aop:advisor advice-ref="before" pointcut-ref="pc"/>
    </aop:config>
```

5. 测试

```java
    public static void main(String[] args) {
        //目的：获得spring工厂创建的动态代理对象并进行调用
        ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("/applicationContext.xml");
        /**
         * 1. spring的工厂通过原始对象的id值获得的是代理对象
         * 2. 获得代理对象后，可以通过声明接口类型，进行对象的存储
         */
        UserService userService = (UserService)ctx.getBean("userService");
        userService.login();
    }
```
### 3.1.4、思考

1. spring创建的动态代理类在哪里？

```java
spring框架在运行时，通过动态字节码技术，在JVM创建的，运行在JVM内部，等程序结束后，会和JVM一起消失

动态代理不需要定义类文件，都是JVM运行过程中动态创建的，所以不会造成静态代理类文件数量过多影响项目管理的问题

```

2. 动态字节码技术？

```java
通过第三方动态字节码框架（ASM, Javassist, cglib）直接在JVM生成字节码，进而创建对象，当虚拟机结束，动态字节码跟着消失
```
### 3.1.5、额外功能详解
MethodBeforeAdvice接口实现的的方法只能运行在目标方法执行之前
MethodInterceptor 前，后，抛出异常 都可以运行
所以我们实战中用到的更多的还是MethodInterceptor 
```java
public class Around  implements MethodInterceptor {
    //methodInvocation : 额外功能所增加给的那个原始方法
    public Object invoke(MethodInvocation methodInvocation) throws Throwable {
        System.out.println("--目标方法之前额外功能--");
        //原始方法执行
        Object proceed = null;
        try {
            proceed = methodInvocation.proceed();
        } catch (Throwable throwable){
            System.out.println("--目标方法抛出异常额外功能--");
            throwable.printStackTrace();
        }
        System.out.println("--目标方法之后额外功能--");
        return proceed;
    }
}
```
> MethodInterceptor可以影响原始方法的返回值（invoke方法不返回原始方法返回值）

### 3.1.6、切入点详解
#### 3.1.6.1、切入点表达式


```java
<aop:pointcut id="pc" expression="execution(* *(..))"/>

execution() ：切入点函数
* *(..)：切入点表达式

第一个* ：修饰符 返回值
第二个* ：方法名(包+类+方法 or 直接方法)
()：参数表 
..：对于参数没有要求
```

- 定义login方法且login方法有两个字符串类型的参数作为切入点

```java
* login(String,String)
```

- 定义login方法且login方法有第一个参数必须为String作为切入点

```java
* login(String,..)
```

- kaka包下UserServiceImpl类的login方法作为切入点（方法切入点表达式）

```java
* com.kaka.UserServiceImpl.login(..)
```

- kaka包及其子包下的所有类的方法作为切入点（包切入点表达式）

```java
* com.kaka..*.*()
    
    其中的..*代表当前包及其子包
    如果是.*代表当前包
```

- 任意包下UserServiceImpl类作为切入点（类切入点表达式）

```java
* *..UserServiceImpl.*(..)
  
    其中的*..代表层级为1级或多级的包
    如果是*.UserService 只能查找第一层级下的类
```
#### 3.1.6.2、切入点函数
切入点函数：用于执行切入点表达式

1. execution

```java
作用：最为重要的切入点函数，功能最全
	  执行方法切入点表达式 类切入点表达式 包切入点表达式
execution执行切入点表达式，书写麻烦
注意：其他的切入点函数简化execution书写复杂度，功能上完全一致

```

2. args

```java
作用：主要用于函数（方法）参数的匹配
方法参数必须是两个字符串类型的参数

args(String,String)
等同于 execution(* *(String,String))
```

3. within

```java
作用：主要用于进行类，包切入点表达式的匹配

within(*..UserServiceImpl)
等同于execution(* *..UserServiceImpl.*(..))
```

4. @annotation

```java
作用：@annotation表示标注了某个注解的所有方法

// 修饰的对象范围
@Target(ElementType.METHOD)
// 注解生命周期  SOURCE < CLASS < RUNTIME
@Retention(RetentionPolicy.RUNTIME)
public @interface Log {
}

public class UserServiceImpl implements UserService{
    @Log
    public void login() {
        System.out.println("user login...");
    }
}


<aop:pointcut id="pc" expression="@annotation(bean.Log)"/>
```
#### 3.1.6.3、切入点函数的逻辑运算

1. and与操作

```java
案例：login 同时 参数 两个字符串

execution(* login(..)) and args(String,String)
等同于 execution(* login(String,String))    
    
注意：与操作不能用于同种类型的切入点函数
```

2. or或操作

```java
案例：register方法和login方法作为切入点

execution(* login(..)) or execution(* register(..))
```
## 3.2、AOP编程
### 3.2.1、基本概念


```java
1. AOP：面向切面编程
   以切面为基本单位的程序开发，通过切面间彼此协同，相互调用，完成程序构建

2. OOP：面向对象编程
   以对象为基本单位的程序开发，通过对象间彼此协同，相互调用，完成程序构建
   
3. POP：面向过程编程
   以过程为基本单位的程序开发，通过过程间彼此协同，相互调用，完成程序构建
 
```
12.2、切面名词解释

> 切面 = 切入点 + 额外功能

多个额外功能相同的方法所代表的点连起来就是一个面

![aop切面](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202307230928946.png)


### 3.2.2、AOP底层实现原理
#### 3.2.2.1、核心问题


> 1. aop如何创建动态代理类（动态字节码技术）
> 2. 如何实现通过原始对象的id值，获得的是代理对象


### 3.2.3、动态代理类的创建
#### 3.2.3.1、JDK动态代理

- Proxy.newProxyInstancec：

- 编码实现

```java
public class TestJDKProxy {
    /**
     1. 借⽤类加载器  TestJDKProxy 或 UserServiceImpl 都可以
     2. JDK8.x 前必须加 final
     final UserService userService = new UserServiceImpl();
     */
    public static void main(String[] args) {
        // 1. 创建原始对象
        UserService userService = new UserServiceImpl();

        // 2. JDK 动态代理
        InvocationHandler handler = new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                System.out.println("---- proxy log ----");
                // 原始方法运行
                Object ret = method.invoke(userService, args);
                return ret;
            }
        };
        UserService userServiceProxy = (UserService) Proxy.
                newProxyInstance(TestJDKProxy.class.getClassLoader(),
                                userService.getClass().getInterfaces(),
                                handler);
        userServiceProxy.login("zhenyu", "123456");

        userServiceProxy.register(new User());
    }
}
```
#### 3.2.3.2、CGLib动态代理

- CGlib 创建动态代理的原理：通过父子继承关系创建代理对象，原始类作为父类，代理类作为子类，这样既可以保证 2 者方法⼀致，同时在代理类中可以提供新的实现（额外功能+原始方法）

> 在原始类没有实现接口的情况下cglib是一种很好的实现方式

- cglib原理

```java
public class TestCglib {
    public static void main(String[] args) {
        // 1. 创建原始对象
        UserService userService = new UserService();

        /*
         2. 通过 cglib 方式创建动态代理对象
         对比 jdk 动态代理 ---> Proxy.newProxyInstance(classLoader, interface, invocationHandler);

         Enhancer.setClassLoader()
         Enhancer.setSuperClass()
         Enhancer.setCallBack() ---> MethodInterceptor(cglib)
         Enhancer.createProxy() ---> 创建代理对象
         */
        Enhancer enhancer = new Enhancer();

        enhancer.setClassLoader(TestCglib.class.getClassLoader());
        enhancer.setSuperclass(userService.getClass());

        MethodInterceptor interceptor = new MethodInterceptor() {
            @Override
            public Object intercept(Object o, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
                System.out.println("--- cglib log ----");
                Object ret = method.invoke(userService, args); // 执行原始方法
                return ret;
            }
        };

        enhancer.setCallback(interceptor);
        UserService userServiceProxy = (UserService) enhancer.create();
        userServiceProxy.login("zhenyu", "123456");
        userServiceProxy.register(new User());
    }
}
```
 总结：

1. JDK 动态代理
Proxy.newProxyInstance：通过接口创建代理的实现类
2. Cglib 动态代理
Enhancer：通过继承父类创建的代理类

### 3.2.4、基于注解的AOP编程
#### 3.2.4.1、开发步骤

1. 原始功能

包含原始功能和实现类等

2. 额外功能+切入点+组装切面

```java
/*
    1. 额外功能
        public class MyAround implements MethodInterceptor {
            public Object invoke(MethodInvocation invocation) {
                Object ret = invocation.invoke();
                return ret;
            }
        }
        <bean id="around" class="com.yusael.dynamic.Around"/>

    2. 切入点
        <aop:config>
            <aop:pointcut id="pc" expression="execution(* login(..)))"/>
            <aop:advisor advice-ref="around" pointcut-ref="pc"/>
        </aop:config>
 */

@Aspect
public class MyAspect {
    @Around("execution(* login(..))")
    public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("---- aspect log ----");
        Object ret = joinPoint.proceed();
        return ret;
    }
}

```
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean id="userService" class="com.yusael.aspect.UserServiceImpl"/>
    <!--
        切面:
            1. 额外功能
            2. 切入点
            3. 组装切面
    -->
    <bean id="around" class="com.yusael.aspect.MyAspect"/>
    <!--告知 Spring 基于注解进行 AOP 编程-->
    <aop:aspectj-autoproxy/>

</beans>

```

3. 切入点复用

切入点复用：在切面类中定义⼀个函数，上面用 @Pointcut 注解。
	通过这种方式定义切入点表达式，后续更加有利于切入点复用
```java
@Aspect
public class MyAspect {

    @Pointcut("execution(* login(..))")
    public void myPoincut() {}

    @Around(value = "myPoincut()")
    public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("---- aspect log ----");
        Object ret = joinPoint.proceed();
        return ret;
    }
    @Around(value = "myPoincut()")
    public Object around1(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("---- aspect transaction ----");
        Object ret = joinPoint.proceed();
        return ret;
    }
}
```
#### 3.2.4.2、动态代理创建方式

AOP 底层实现 2 种代理创建方式：

1. JDK：通过 **实现接口，做新的实现类** 创建代理对象
2. Cglib：通过 **继承父类，做新的子类** 创建代理对象

**默认情况 AOP 编程 底层应用 JDK动态代理创建方式**
```java
<!-- 动态代理模式切换为cglib -->
<aop:aspectj-autoproxy proxy-target-class="true"/>
```

# 4、spring 持久层
## 4.1、spring与mybatis整合

1. 引入依赖

```xml
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-jdbc</artifactId>
  <version>5.2.6.RELEASE</version>
</dependency>

<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis-spring</artifactId>
  <version>2.0.4</version>
</dependency>

<dependency>
  <groupId>com.alibaba</groupId>
  <artifactId>druid</artifactId>
  <version>1.1.12</version>
</dependency>

<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <version>5.1.43</version>
</dependency>

<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis</artifactId>
  <version>3.5.4</version>
</dependency>
```

2. spring配置文件

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd">
        <!--连接池-->
        <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
            <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
            <property name="url" value="jdbc:mysql://localhost:3306/yus?useSSL=false"/>
            <property name="username" value="root"/>
            <property name="password" value="1234"/>
        </bean>

        <!--创建SqlSessionFactory SqlSessionFactoryBean-->
        <bean id="sqlSessionFactoryBean" class="org.mybatis.spring.SqlSessionFactoryBean">
            <property name="dataSource" ref="dataSource"/>
            <property name="typeAliasesPackage" value="com.yusael.entity"/>
            <property name="mapperLocations">
                <list>
                    <value>classpath:com.yusael.dao/*Mapper.xml</value>
                </list>
            </property>
        </bean>

        <!--创建DAO对象 MapperScannerConfigure-->
        <bean id="scanner" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
            <property name="sqlSessionFactoryBeanName" value="sqlSessionFactoryBean"/>
            <property name="basePackage" value="com.yusael.dao"/>
        </bean>
</beans>
```

3. 测试

```java
@Test
public void test() {
    ApplicationContext ctx = new ClassPathXmlApplicationContext("/applicationContext.xml");
    UserDAO userDAO = (UserDAO) ctx.getBean("userDAO");

    User user = new User();
    user.setName("xiaojr");
    user.setPassword("999999");

    userDAO.save(user);
}
```
## 4.2、关于事务提交的疑问

**问题**：Spring 与 Myabatis 整合后，为什么 DAO 不提交事务，但是数据能够插入数据库中？

> Mybatis 提供的连接池对象 —> 创建 Connection
> Connection.setAutoCommit(false) 手工的控制了事务，操作完成后，需要手工提交。
> Druid（C3P0、DBCP）作为连接池 —> 创建 Connection
> Connection.setAutoCommit(true) 默认值为 true，保持自动控制事务，一条 sql 自动提交。


**答案**：因为 Spring 与 Mybatis 整合时，引入了外部连接池对象，保持自动的事务提交这个机制Connection.setAutoCommit(true)，不需要手工进行事务的操作，也能进行事务的提交。

**注意**：实战中，还是会手工控制事务（多条SQL一起成功，一起失败）后续 Spring 通过 事务控制 解决这个问题

## 4.3、spring事务处理
### 4.3.1、事务基本概念

1. 什么是事务？

事务时保证业务操作完整性的一种**数据库机制**

2. 事务的四大特性？

A 原子性	C 一致性 I 隔离性	D 持久性

3. 如何控制事务？

**JDBC：**
Connection.setAutoCommit(false);
Connection.commit();
Connection.rollback();
**Mybatis：**
Mybatis 自动开启事务
sqlSession.commit();，底层还是调用的 Connection
sqlSession.rollback();，底层还是调用的 Connection
> 结论：控制事务的底层，都是通过 Connection 对象完成的

### 4.3.2、spring控制事务的开发
spring是通过aop的方式进行事务开发
**增强功能：**原理也是在方法执行前关闭自动提交，spring帮我们封装成datasourcetransactionmanager，不需要自己去写增强方法

**切入点：**@Transactional （1）作用于类上（2）作用域方法。<

**组装切面：**tx:annotation-driven transaction-manager="dataSourceTransactionManager"/> 自动扫描所有的@Transactional注解

1. 搭建开发环境

```xml
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-tx</artifactId>
	<version>5.2.6.RELEASE</version>
</dependency>
```

2. 编码

```xml
<bean id="userService" class="com.yusael.service.UserServiceImpl">
	<property name="userDAO" ref="userDAO"/>
</bean>

<!--DataSourceTransactionManager-->
<bean id="dataSourceTransactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
	<property name="dataSource" ref="dataSource"/>
</bean>

@Transactional
public class UserServiceImpl implements UserService {
    private UserDAO userDAO;

<bean id="userService" class="com.yusael.service.UserServiceImpl">
	<property name="userDAO" ref="userDAO"/>
</bean>

<!--DataSourceTransactionManager-->
<bean id="dataSourceTransactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
	<property name="dataSource" ref="dataSource"/>
</bean>

@Transactional
public class UserServiceImpl implements UserService {
    private UserDAO userDAO;

<tx:annotation-driven transaction-manager="dataSourceTransactionManager"/>
```
进行动态代理底层实现的切换，默认 false 是 JDK，true 是 Cglib
```xml
<tx:annotation-driven transaction-manager="dataSourceTransactionManager" proxy-target-class="true"/>
```
### 4.3.3、事务的属性
5个：隔离属性，传播属性，只读属性，超时属性，异常属性
```markdown
@Transactional（isolation=, propagation=, readOnly=,timeout=,rollbackFor=,noRollbackFor=）
```
#### 4.3.3.1、隔离属性

##### 4.3.3.1.1、并发问题&解决方案？
**（1）脏读**：一个事务，读取了另一个事务中没有提交的数据，会在本事务中产生数据不一样的现象
> eg：账户1000，A取300还没提交，B读取到还700，然后取200，A回滚，B提交后账户只剩500了

```java
// 隔离级别：读已提交
@Transaction(isolation=Isolation.READ_COMMITTED)
```

**（2）不可重复读**：一个事务中，多次读取相同的数据，但是读取结果不一样，会在本事务中产生数据不一样的现象
> eg：账户1000，A查询到1000(事务还没提交)，B取200，A再查就剩800了
> 注意：1.不是脏读 2.在一个事务中

```java
// 隔离级别：可重复读
@Transaction(isolation=Isolation.REPEATABLE_READ)
```
> 本质：一把行锁（对数据库表的某一行加锁）



**（3）幻读**：一个事务中，多次对**整表**进行**查询统计**，但是**结果不一样**，会在本事务中产生数据不一致的问题
> 查询表中数据行数count，一个事务两次查询中间，其它用户添加并提交了数据，再读取行数count就会不一致

```java
// 隔离级别：序列化
@Transaction(isolation=Isolation.SERIALIZABLE)
```

> 本质：表锁（对数据库某个表加锁）

##### 4.3.3.1.2、安全与效率对比：

- 并发安全：SERIALIZABLE > READ_ONLY > READ_COMMITTED
- 运行效率：READ_COMMITTED > READ_ONLY > SERIALIZABLE

##### 4.3.3.1.3、数据库默认隔离属性
mysql：可重复读（REPEATABLE_READ）
Oracle：读已提交（READ_COMMITTED）

#### 4.3.3.2、传播属性
##### 4.3.3.2.1、基本概念
传播属性：描述了事务解决 嵌套 问题 的特征
**事务的嵌套**：指的是一个大的事务中，包含了若干个小的事务。
**事务嵌套产生的问题**： 大事务中融入了很多小的事务，他们彼此影响，最终就导致外部大的事务丧失了事务的原子性（一旦外部的事务出现问题，内部已提交的事务无法一起回滚）

##### 4.3.3.2.2、传播属性的值及用法
| 传播属性的值 | 外部不存在事务 | 外部存在事务 | 用法 | 备注 |
| --- | --- | --- | --- | --- |
| REQUIRED | 开启新的事务 | 融合到外部事务中 | @Transactional(propagation = Propagation.REQUIRED) | 增、删、改方法（保证了肯定会有事务的存在） |
| REQUIRES_NEW | 开启新的事务 | 挂起外部事务，创建新的事务 | @Transactional(propagation = Propagation.REQUIRES_NEW) | 日志记录方法中（保证外部事务异常时能正常的记录日志） |
| SUPPORTS | 不开启事务 | 融合到外部事务中 | @Transactional(propagation = Propagation.SUPPORTS) | 查询方法 |
| NOT_SUPPORTED | 不开启事务 | 挂起外部事务| @Transactional(propagation = Propagation.NOT_SUPPORTED) | 极其不常用 |
| NEVER | 不开启事务 | 抛出异常 | @Transactional(propagation = Propagation.NEVER) | 极其不常用 |
| MANDATORY（强制的） | 抛出异常 | 融合到外部事物中 | @Transactional(propagation = Propagation.MANDATORY) | 极其不常用 |

Spring 中**传播属性的默认值**是：REQUIRED

推荐传播属性的使用方式：

- 增删改 方法：使用默认值 REQUIRED
- 查询 方法：显示指定传播属性的值为 SUPPORTS


#### 4.2.2.3、只读属性
针对于 **只进行查询操作的业务方法**，可以加入只读属性，提高运行效率。
默认值：false
```java
@Transactional(readOnly = true)
```
#### 4.2.2.4、超时属性
指定了事务等待的最长时间。

1. 为什么事务会进行等待？

当前事务访问数据时，有可能访问的数据被别的事务进行加锁的处理，那么此时本事务就必须进行等待。

2. 等待时间，单位是 秒
3. 如何使用：@Transactional(timeout = 2)
4. 超时属性的默认值：-1

-1 表示超时属性由对应的数据库来指定（一般不会主动指定，-1 即可）

#### 4.2.2.5、异常属性
Spring 事务处理过程中：

- 默认对于 RuntimeException 及其子类，采用 **回滚** 的策略。
- 默认对于 对于其他类型的异常，采用 **提交** 的策略（例如IO异常、网络异常等，可以在程序中使用try-catch块来处理这些异常，或者在方法签名中使用throws关键字将异常抛出给调用者处理。如果Spring默认采用回滚策略，会导致这些异常被回滚，可能会带来意想不到的后果，例如文件未能正确关闭、网络连接未能正确关闭等）

```java
@Transactional(rollbackFor = java.lang.Exception.class, xxx, xxx)

@Transactional(noRollbackFor = java.lang.RuntimeException, xxx, xxx)
```


# 5、spring mvc
spring mvc相关内容请参考spring mvc的笔记，这里只讨论spring mvc和spring 容器的关系 

## 5.1、父子容器关系
1. Spring框架的核心是Spring容器（BeanFactory，ApplicationContext ）。Spring MVC是Spring框架中的一个模块，它提供了一种基于MVC模式的Web应用程序开发方式。 Spring MVC框架的核心是Spring MVC容器（WebApplicationContext）
2. Spring MVC容器继承了Spring容器的所有功能，并且提供了一些额外的功能，如处理HTTP请求和响应、支持多种视图技术等
3. Spring 容器是父容器，SpringMVC 是子容器，子容器可以访问父容器的 Bean，但是父容器不能访问子容器的 Bean（参考类继承的访问权限）
4. 调用子容器的getBean方法获取bean的时候，会沿着当前容器开始向上面的容器进行查找，直到找到对应的bean为止


## 5.2、为什么要划分父子容器
1. 分离关注点：Spring MVC负责处理Web请求和响应，与业务逻辑无关，因此需要将Spring MVC容器与Spring容器分离，以便更好地分离关注点，简化应用程序的开发和维护（单一职责原则）
2. 性能优化：Spring MVC容器的初始化速度比Spring容器快，因为它只需要初始化与Web请求相关的对象。而Spring容器需要初始化整个应用程序中的所有对象，这可能会影响应用程序的性能

## 5.3、springBoot舍弃了父子容器的概念

SpringBoot只有一个容器。
Spring Boot框架采用了“约定优于配置”的设计理念，旨在简化Spring应用程序的开发和部署


# 6、注解编程
## 6.1、spring基础注解（spring2.X）
### 6.1.1、对象创建相关
#### 6.1.1.1、@Component

1. 作用：替换原有Spring配置文件中的 <bean> 标签
- id 属性：在 @Component 中提供了默认的设置方式：首单词首字母小写（UserDAO --> userDAO）
- class 属性：通过反射获得的 class 的内容
2. 细节：如何显式指定工厂创建对象的 id 值

```java
@Component("u")
```
#### 6.1.1.2、@Repository、@Service、@Contoller
@Repository、@Service、@Controller 都是 @Component 的 **衍生注解**。
本质上这些衍生注解就是 @Component，通过源码可以看见他们都使用了 @Component；
它们的存在是为了：**更加准确的表达一个类型的作用**

#### 6.1.1.3、@Scope
作用：控制简单对象创建次数
注意：不添加 @Scope，Spring 提供默认值 singleton
```java
// 创建单例对象
@Component
@Scope("singleton")
public class Customer {}

// 创建多例对象
@Component
@Scope("prototype")
public class Customer {}
```
#### 6.1.1.4、@Lazy
作用：延迟创建单实例对象
注意：一旦使用 @Lazy 注解后，Spring 会在使用这个对象的时候，才创建这个对象
```java
@Component
@Lazy
public class Account {
    public Account() {
        System.out.println("Account.Account");
    }
}
```
#### 6.1.1.5、@PostConstruct、@PreDestroy
初始化相关方法： @PostConstruct
```java
InitializingBean
<bean init-method=""/>
```
销毁方法：@PreDestory
```java
DisposableBean
<bean destory-method=""/>
```
### 6.1.2、注入相关注解
#### 6.1.2.1、@Autowired（用户自定义类型）

1. @Autowired 注解 **基于类型进行注入** [推荐]：
- 注入对象的类型，必须与目标成员变量类型相同或者是其子类（实现类）

```java
@Autowired
private UserDAO userDAO;
```

2. @Autowired、@Qualifier 注解联合实现 **基于名字进行注入** [了解]
- 注入对象的 id 值，必须与 @Qualifier 注解中设置的名字相同

```java
@Autowired
@Qualifier("userDAOImpl")
private UserDAO userDAO;
```

3. @Autowired 注解放置位置：
- 放置在对应成员变量的 set 方法上，调用 set 方法赋值（在 set 里写的代码会被执行）
- **直接放置在成员变量上**，Spring 通过反射直接对成员变量进行赋值

4. JSR提供的@Resource注解

> JavaEE 规范中类似功能的注解：
> - JSR250 提供的 @Resource(name="xxx") **基于名字进行注入**
等价于 @Autowired 与 @Qualifier 联合实现的效果
注意：@Resource 注解如果名字没有配对成功，会继续 **按照类型进行注入**

#### 6.1.2.2、@value、@PropertySource（JDK 类型）

1. @value 注解的基本使用（xml配置）：

```java
1. 设置xxx.properties 
   id = 10
   name = suns
2. Spring的工厂读取这个配置文件 
   <context:property-placeholder location=""/>
3. 代码中进行注入
   属性 @Value("${key}")

```

2. 使用 @PropertySource 取代 xml配置

```java
@Configuration
@PropertySource("classpath:/init.properties")
public class AppConfig1 {

	@Value("${id}")
	private Integer id;
	@Value("${name}")
	private String name;
	
	@Bean
	public Customer customer() {
		Customer customer = new Customer();
		customer.setId(id);
		customer.setName(name);
		return customer;
	}
}

```

3. @value 注解使用细节：
- @Value 注解不能应用在静态成员变量上，如果使用，获取的值为 null
- @Value 注解 + Properties 这种方式，不能注入集合类型
Spring 提供新的配置形式 YAML(YML) (更多的用于SpringBoot中)
## 
## 6.2、spring的高级注解（spring3.X及以上）
### 6.2.1、@Configuration（配置bean）

1. Spring 在 3.x 提供的新的注解@Configuration，用于替换 XML 配置文件
2. 使用了 @Configuration 后，用 AnnotationConfigApplicationContext 创建工厂：

```xml
方法1: 指定配置bean的Class
ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);

方法2: 指定配置bean所在的路径(某个包及其子包)
ApplicationContext ctx = new AnnotationConfigApplicationContext("com.yusael");
```
### 6.2.2、@Bean
@Bean 注解在配置 bean 中进行使用，等同于 XML 配置文件中的 <bean 标签
> **简单对象**：直接能够通过 new 方式创建的对象
> - User、UserService、UserDAO
> 
**复杂对象**：不能通过 new 的方式直接创建的对象
> - Connection、SqlSessionFactory

```java
@Configuration
public class AppConfig{
  // 简单对象
	@Bean
  public Customer customer() {
    Customer customer = new Customer();
    return customer;
  }

  // 复杂对象
  @Bean
  public Connection conn1() {
    Connection conn = null;
    try {
      ConnectionFactoryBean factoryBean = new ConnectionFactoryBean();
      conn = factoryBean.getObject();
    } catch (Exception e) {
      e.printStackTrace();
    }
    return conn;
  }
}
```
### 6.2.3、@ComponentScan
@ComponentScan 注解在配置 bean 中进行使用，等同于 XML 配置文件中的 <context:component-scan> 标签
目的：进行相关注解的扫描（@Component、@Value、@Autowired …)
```java
@Configuration
@ComponentScan(basePackages = "com.yusael.scan",
               excludeFilters = {@ComponentScan.Filter(type= FilterType.ANNOTATION, value={Service.class}),
                                 @ComponentScan.Filter(type= FilterType.ASPECTJ, pattern = "*..User1")})
public class AppConfig2 {
}
```

