---
title: 各框架下的SPI机制实现原理
categories:
  - coding
date: 2022/11/21
tags:
  - 技术文章
  - SPI
abbrlink: 12113
---
# 1、基本概念

## 1.1、概念

SPI （ Service Provider Interface），是一种服务发现机制。SPI 的本质是将**接口的实现类**的全限定名配置在文件中，并由服务加载器读取**配置文件**，加载对应接口的实现类。这样就可以在运行时，获取接口的实现类。通过这一特性，我们可以给很容易的通过 SPI 机制为程序提供拓展功能

## 1.2、使用场景
概括地说，适用于：调用者根据实际使用需要，启用、扩展、或者替换框架的实现策略。比较常见的例子：

- 数据库驱动加载接口实现类的加载：JDBC加载不同类型数据库的驱动
- 日志门面接口实现类加载：SLF4J加载不同提供商的日志实现类
- Spring：Spring中大量使用了SPI,比如：对servlet3.0规范对ServletContainerInitializer的实现、自动类型转换Type Conversion SPI(Converter SPI、Formatter SPI)等
- Dubbo：Dubbo中也大量使用SPI的方式实现框架的扩展, 不过它对Java提供的原生SPI做了封装，允许用户扩展实现Filter接口。

# 2、java SPI

## 2.1、基本概念

Java SPI 是“基于接口的编程＋策略模式＋配置文件”组合实现的动态加载机制。设计一个接口，将接口的实现类写在配置文件中，服务通过读取配置文件来发现实现类，进行加载实例化然后使用。

配置文件路径：classpath下的META-INF/services/
配置文件名：接口的全限定名
配置文件内容：接口的实现类的全限定名


## 2.2、使用

1、定义接口和实现类

```java
// 接口
public interface Superman {
    void introduce();
}

// 实现类1
public class IronMan implements Superman{
    @Override
    public void introduce() {
        System.out.println("我是钢铁侠！");
    }
}
// 实现类2
public class CaptainAmerica implements Superman {
    @Override
    public void introduce() {
        System.out.println("我是美国队长！");
    }
}
```

2、定义配置类

classpath下的META-INF/services/ 下创建文件：com.yc.service.spi.Superman（接口的全限定名），文件内容:

```
com.yc.service.spi.IronMan
com.yc.service.spi.CaptainAmerica
```


3、使用

```java
public static void main(String[] args) {
    ServiceLoader<Superman> serviceLoader = ServiceLoader.load(Superman.class);
    System.out.println("Java SPI:");
    serviceLoader.forEach(Superman::introduce);
}
```


看到这里，就应该了解，mysql-connector 的 jar 包中正是通过 SPI 的方式实现了 java 的 java.Driver.sql 接口，所以我们的服务可以在运行时获取到 mysql 的驱动类，从而连接 mysql 


## 2.3、原理

Java SPI 的实现原理基于 Java 类加载机制和反射机制。使用 ServiceLoader.load(Class< T > service) 方法加载服务时，会检查 META-INF/services 目录下是否存在以接口全限定名命名的文件。如果存在，则读取文件内容，获取实现该接口的类的全限定名，并通过 Class.forName() 方法加载对应的类。在加载类之后，ServiceLoader 会通过反射机制创建对应类的实例，并将其缓存起来。

> 这里涉及到一个懒加载迭代器的思想：当我们调用 ServiceLoader.load(Class< T > service) 方法时，并不会立即将所有实现了该接口的类都加载进来，而是返回一个懒加载迭代器。只有在使用迭代器遍历时，才会按需加载对应的类并创建其实例

## 2.4、优缺点

**优点：**  
核心思想：**解耦**，使得第三方服务模块的装配控制的逻辑与调用者的业务代码分离。可以根据实际业务情况进行使用或扩展。  

**缺点：**  
1、获取接口的实现类的方式不灵活
serviceLoader 只能通过 Iterator 形式遍历获取，不能根据参数获取指定的某个实现类。  
2、资源浪费
serviceLoader 只能通过遍历的方式将接口的实现类全部获取、加载并实例化一遍。如果不想用某些实现类，它也被加载并实例化，造成浪费。


# 3、Spring SPI

## 3.1、概念

与 JDK SPI 类似， 相对于 Java 的 SPI 的优势在于：  
Spring SPI 指定配置文件为 classpath 下的 META-INF/spring.factories，所有的拓展点配置放到一个文件中。  
配置文件内容为 key-value 类型，key 为接口的全限定名， value 为 实现类的全限定名，可以为多个。

> 值得一提的是，loadFactoryNames方法不仅加载当前资源目录META-INF下的spring.factories文件，还会导入项目中引入jar包的META-INF/spring.factories文件


## 3.2、spring SPI引入mybatis流程

mybatis不集成springBoot时使用方法：

```java
InputStream xml = Resources.getResourceAsStream("mybatis-config.xml");
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(xml);
SqlSession sqlSession = sqlSessionFactory.openSession();

PersonDao personDao = sqlSession.getMapper(PersonDao.class);
List<Person> all = personDao.findAll();
```

mybatis在不集成springboot时，需要mybatis-config.xml配置文件，而且需要手动调用SqlSessionFactoryBuilder()的build()方法解析配置文件并生成SqlSessionFactory。

springBoot自动配置如何实现注入：

1、在springboot中，mybatis引用的jar包

```xml
 <dependency>
     <groupId>org.mybatis.spring.boot</groupId>
     <artifactId>mybatis-spring-boot-starter</artifactId>
     <version>2.1.3</version>
</dependency>

```

2、导入后发现这个依赖其实就是帮助我们导入了mybatis需要的依赖，其中和自动配置相关最重要的一个就是mybatis-spring-boot-autoconfigure

```xml
<dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-jdbc</artifactId>
    </dependency>
    <dependency>
      <groupId>org.mybatis.spring.boot</groupId>
      <artifactId>mybatis-spring-boot-autoconfigure</artifactId>
    </dependency>
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
    </dependency>
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis-spring</artifactId>
    </dependency>
  </dependencies>

```

3、通过spring SPI加载了自动配置类 MybatisAutoConfiguration

![image.png](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202403082339214.png)

4、通过@Condition，确保有sqlSessionFactory类和spring容器中有数据源实例

```java
@org.springframework.context.annotation.Configuration
@ConditionalOnClass({ SqlSessionFactory.class, SqlSessionFactoryBean.class })
@ConditionalOnSingleCandidate(DataSource.class)
@EnableConfigurationProperties(MybatisProperties.class)
@AutoConfigureAfter({ DataSourceAutoConfiguration.class, MybatisLanguageDriverAutoConfiguration.class })
public class MybatisAutoConfiguration implements InitializingBean {

  @Bean
  @ConditionalOnMissingBean
  public SqlSessionFactory sqlSessionFactory(DataSource dataSource) throws Exception {
    SqlSessionFactoryBean factory = new SqlSessionFactoryBean();
    factory.setDataSource(dataSource);
    factory.setVfs(SpringBootVFS.class);
    if (StringUtils.hasText(this.properties.getConfigLocation())) {
      factory.setConfigLocation(this.resourceLoader.getResource(this.properties.getConfigLocation()));
    }
    applyConfiguration(factory);
	
    // ... 省略部分代码   

	// 通过SqlSessionFactoryBean 注入 SqlSessionFactory
    return factory.getObject();
  }

  @Bean
  @ConditionalOnMissingBean
  public SqlSessionTemplate sqlSessionTemplate(SqlSessionFactory sqlSessionFactory) {
    ExecutorType executorType = this.properties.getExecutorType();
    // ...注入SqlSessionTemplate
    if (executorType != null) {
      return new SqlSessionTemplate(sqlSessionFactory, executorType);
    } else {
      return new SqlSessionTemplate(sqlSessionFactory);
    }
  }

```

5、到这里也就知道了MyBatis自动配置其实就是替我们完成了SqlSessionFactory和SqlSessionTempate的创建, 省去了自己导入相关依赖和配置相关Bean的麻烦.

# 4、Dubbo SPI
## 4.1、基本概念

dubbo的 Filter、Protocol、Cluster、LoadBalance 等都是通过 SPI 的方式进行拓展加载的。但它没有使用jdk的spi机制，而是自己实现了一套ExetensionLoader

相Dubbo的SPI机制可直接通过Key获取我们想要的对象实例，比原生的Java SPI更有优势，除此之外，Dubbo在设计中还用到了大量的全局缓存，提高了我们实例化对象的效率，同时还支持通过注解默认实现，AOP,IOC等功能

## 4.2、demo

1、测试方法

```java
@Slf4j
public class Main {
    public static void main(String[] args) {
        log.info("这是Dubbo的SPI机制");
        Animal cat = ExtensionLoader.getExtensionLoader(Animal.class)
                .getExtension("cat");
        Animal dog = ExtensionLoader.getExtensionLoader(Animal.class)
                .getExtension("dog");
        cat.call();
        dog.call();
 
    }
}
 
```

2、配置文件

```
cat=cuit.epoch.pymjl.spi.impl.Cat
dog=cuit.epoch.pymjl.spi.impl.Dog
```

# 5、JDK SPI 、spring spi、dubbo spi 的比较

JDK SPI: 每个扩展点单独一个文件;只能通过ServiceLoader的迭代器遍历获取扩展实现类，同时也会加载所有实现类，这样就会把用不到的类也会进行实例化，浪费一定资源；是java自带实现，不需要引入第三方依赖可直接使用．

spring spi: spring指定配置文件为classpath下的META-INF/spring.factories, 所有的扩展点配置都放到这个文件中．配置文件内容为key=value的形式，key为接口的全限定名，value为实现类的全限定名，可以为多个．

dubbo spi:每个扩展点一个配置文件，文件名为接口的全限定名．支持别名的概念，可以通过别名获取某个具体扩展点的实现．配置文件的内容为key=value形式，key是别名，value是实现类的全限定名．

