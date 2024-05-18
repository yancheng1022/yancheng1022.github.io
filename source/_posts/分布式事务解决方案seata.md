---
title: 分布式事务解决方案seata
date: 2023/08/01
updated: 2023/08/02
categories:
  - coding
tags:
  - seata
  - 分布式事务
  - 技术文章
abbrlink: 18601
---

# 1、概念

## 1.1、背景

在微服务架构下，由于数据库和应用服务的拆分，导致原本一个事务单元中的多个DML 操作，变成了跨进程或者跨数据库的多个事务单元的多个 DML 操作，而传统的数据库事务无法解决这类的问题，所以就引出了分布式事务的概念。分布式事务本质上要解决的就是跨网络节点的多个事务的数据一致性问题，业内常见的解决方法有两种

a. 强一致性，就是所有的事务参与者要么全部成功，要么全部失败，全局事务协调者需要知道每个事务参与者的执行状态，再根据状态来决定数据的提交或者回滚
b. 最终一致性，也叫弱一致性，也就是多个网络节点的数据允许出现不一致的情况，但是在最终的某个时间点会达成数据一致。

在分布式事务的实现上，对于强一致性，我们可以通过基于 XA 协议下的二阶段提交来实现，对于弱一致性，可以基于 TCC 事务模型、可靠性消息模型等方案来实现。


## 1.2、常见分布式事务解决方案

1、seata阿里分布式事务框架
2、消息队列
3、sage
4、XA

他们有一个共同的特点，都是两阶段（2PC），两阶段是指完成整个分布式事务，划分为两个步骤完成。
实际上这四种方案分别对应着分布式事务的四种模式：AT,TCC,Sage,XA

>三阶段提交协议3PC非常难实现，目前市面主流的分布式事务解决方案都是2PC协议

二阶段分别指准备和提交：
1、在准备阶段RM执行实际的业务操作，但不提交事务，资源锁定；
2、在提交阶段TM会接受RM在准备阶段的执行回复，只要有任一个RM执行失败，TM会通知所有RM执行回滚操 作，否则，TM将会通知所有RM提交该事务。提交阶段结束资源锁释放。


## 1.3、seata架构

TC (Transaction Coordinator) - 事务协调者，维护全局和分支事务的状态，驱动全局事务提交或回滚。
TM (Transaction Manager) - 事务管理器，定义全局事务的范围：开始全局事务、提交或回滚全局事务。
RM (Resource Manager) - 资源管理器，管理分支事务处理的资源，与TC交谈以注册分支事务和报告分支事务的状态，并驱动分支事务提交或回滚。

## 1.4、seata下的分布式事务解决方案

### 1.4.1、AT

AT模式（自动事务）：最终一致的分阶段事务模式，无业务侵入，也是Seata的默认模式

在一阶段，seata 会拦截"业务 SQL"，首先解析 SQL 语义，找到“业务 SQL"要更新的业务数据，在业务数据被更新前，将其保存成"before image”快照中，然后执行"业务 SQL"更新业务数据，在业务数据更新之后，再将其保存成"afer image”快照中（这两个快照保存在undo-log表中），同时在执行过程加行锁。以上操作全部在一个数据库事务内完成，这样保证了一阶段操作的原子性
在二阶段，如果所有分支事务成功，则将一阶段生成的快照数据和行锁删除；如果有分支失败，则所有分支事务进行还原，回滚方式是用before image生成逆向sql进行业务数据还原

### 1.4.2、TCC

TCC模式：最终一致的分阶段事务模式，有业务侵入
TCC 模式需要用户根据自己的业务场景实现, Try、Conirm 和 Cancel 三个操作（方法）;事务发起方在一阶段执行 Try 方法，在二阶段提交执行 Confm 方法，二阶段回滚执行 Cancel 方法。

### 1.4.3、XA

XA模式：强一致性分阶段事务模式，牺牲了一定的可用性，无业务侵入
一阶段：注册分支事务到TC， 执行分支业务但不提交，报告执行状态到TC
二阶段：TC检测各分支事务执行状态 a.如果都成功，通知所有RM提交事务 b.如果有失败，通知所有RM回滚事务

>XA模式的优点是什么？
事务的强一致性，满足ACID原则。
常用数据库都支持，实现简单，并且没有代码侵入
XA模式的缺点是什么？
因为一阶段需要锁定数据库资源，等待二阶段结束才释放，性能较差
依赖关系型数据库实现事务

### 1.4.4、SAGA
SAGA模式：长事务模式，有业务侵入


# 2、部署Seata的tc-server


## 1.1、下载，解压

下载seata-server包，地址在[http](http://seata.io/zh-cn/blog/download.html)[://seata.io/zh-cn/blog/download](http://seata.io/zh-cn/blog/download.html)[.](http://seata.io/zh-cn/blog/download.html)[html](http://seata.io/zh-cn/blog/download.html) 

## 1.2、修改 conf/registry.conf 和file.conf配置

Seata连接到服务器的时候需要一些配置项，这时候有一个registry.conf文件可以指定注册中心和配置文件是什么。  

这里有很多可选性，比如file、nacos 、apollo、zk、consul


```properties

- serverAddr = “127.0.0.1:2181” ：zk 的地址
- cluster = “default” ：集群设置为默认 `default`
- session.timeout = 6000 ：会话的超时时间
- connect.timeout = 2000：连接的超时时间

registry {
  # file zk
  type = "zk"

 zk {
    cluster = "default"
    serverAddr = "127.0.0.1:2181"
    session.timeout = 6000
    connect.timeout = 2000
  }
  file {
    name = "file.conf"
  }
}

config {
  # file、nacos 、apollo、zk、consul
  type = "zk"

  zk {
    serverAddr = "127.0.0.1:2181"
    session.timeout = 6000
    connect.timeout = 2000
  }

  file {
    name = "file.conf"
  }
}
```



file.conf 配置

```properties

主要修改了`store.mode`为`db`,还有数据库相关的配置
store {
  ## store mode: file、db
  mode = "db"

  ## file store
  file {
    dir = "sessionStore"

    # branch session size , if exceeded first try compress lockkey, still exceeded throws exceptions
    max-branch-session-size = 16384
    # globe session size , if exceeded throws exceptions
    max-global-session-size = 512
    # file buffer size , if exceeded allocate new buffer
    file-write-buffer-cache-size = 16384
    # when recover batch read size
    session.reload.read_size = 100
    # async, sync
    flush-disk-mode = async
  }

  ## database store
  db {
    ## the implement of javax.sql.DataSource, such as DruidDataSource(druid)/BasicDataSource(dbcp) etc.
    datasource = "dbcp"
    ## mysql/oracle/h2/oceanbase etc.
    db-type = "mysql"
    driver-class-name = "com.mysql.jdbc.Driver"
    url = "jdbc:mysql://127.0.0.1:3306/seata"
    user = "root"
    password = "123456"
    min-conn = 1
    max-conn = 3
    global.table = "global_table"
    branch.table = "branch_table"
    lock-table = "lock_table"
    query-limit = 100
  }
}
```


修改 conf/nacos-config.txt配置为zk-config.properties

```
store.mode :存储模式 默认file 这里我修改为db 模式 ，并且需要三个表global_table、branch_table和lock_table
store.db.driver-class-name： 默认没有，会报错。添加了 com.mysql.jdbc.Driver
store.db.datasource=dbcp ：数据源 dbcp
store.db.db-type=mysql : 存储数据库的类型为mysql
store.db.url=jdbc:mysql://127.0.0.1:3306/seata?useUnicode=true : 修改为自己的数据库url、port、数据库名称
store.db.user=root :数据库的账号
store.db.password=123456 :数据库的密码
service.vgroup_mapping.order-service-seata-service-group=default
service.vgroup_mapping.account-service-seata-service-group=default
service.vgroup_mapping.storage-service-seata-service-group=default
service.vgroup_mapping.business-service-seata-service-group=default

```

## 1.4、创建数据库表

特别注意：tc服务在管理分布式事务时，需要记录事务相关数据到数据库中，你需要提前创建好这些表。
这些表主要记录全局事务(global_table)、分支事务(branch_table)、全局锁信息(lock_table)：

```mysql
SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- 分支事务表
-- ----------------------------
DROP TABLE IF EXISTS `branch_table`;
CREATE TABLE `branch_table`  (
  `branch_id` bigint(20) NOT NULL,
  `xid` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
  `transaction_id` bigint(20) NULL DEFAULT NULL,
  `resource_group_id` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `resource_id` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `branch_type` varchar(8) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `status` tinyint(4) NULL DEFAULT NULL,
  `client_id` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `application_data` varchar(2000) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `gmt_create` datetime(6) NULL DEFAULT NULL,
  `gmt_modified` datetime(6) NULL DEFAULT NULL,
  PRIMARY KEY (`branch_id`) USING BTREE,
  INDEX `idx_xid`(`xid`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;

-- ----------------------------
-- 全局事务表
-- ----------------------------
DROP TABLE IF EXISTS `global_table`;
CREATE TABLE `global_table`  (
  `xid` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
  `transaction_id` bigint(20) NULL DEFAULT NULL,
  `status` tinyint(4) NOT NULL,
  `application_id` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `transaction_service_group` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `transaction_name` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `timeout` int(11) NULL DEFAULT NULL,
  `begin_time` bigint(20) NULL DEFAULT NULL,
  `application_data` varchar(2000) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `gmt_create` datetime NULL DEFAULT NULL,
  `gmt_modified` datetime NULL DEFAULT NULL,
  PRIMARY KEY (`xid`) USING BTREE,
  INDEX `idx_gmt_modified_status`(`gmt_modified`, `status`) USING BTREE,
  INDEX `idx_transaction_id`(`transaction_id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;

SET FOREIGN_KEY_CHECKS = 1;

create table `lock_table` (
  `row_key` varchar(128) not null,
  `xid` varchar(96),
  `transaction_id` long ,
  `branch_id` long,
  `resource_id` varchar(256) ,
  `table_name` varchar(32) ,
  `pk` varchar(32) ,
  `gmt_create` datetime ,
  `gmt_modified` datetime,
  primary key(`row_key`)
);

```



## 1.5、启动TC服务

进入bin目录，运行其中的seata-server.bat即可：启动成功后，seata-server应该已经注册到nacos注册中心了。打开浏览器，访问nacos地址：http://localhost:8848，然后进入服务列表页面，可以看到seata-tc-server的信息：

# 3、微服务集成seata

## 3.1、引入依赖

首先，我们需要在微服务中引入seata依赖：

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
    <exclusions>
        <!--版本较低，1.3.0，因此排除-->
        <exclusion>
            <artifactId>seata-spring-boot-starter</artifactId>
            <groupId>io.seata</groupId>
        </exclusion>
    </exclusions>
</dependency>
<!--seata starter 采用1.4.2版本-->
<dependency>
    <groupId>io.seata</groupId>
    <artifactId>seata-spring-boot-starter</artifactId>
    <version>${seata.version}</version>
</dependency>
```


## 3.2、添加配置类

需要修改application.yml文件，添加一些配置：
DEFAULT_MODE：默认的事务模式 为AT_MODE

```java

@Configuration
public class SeataAutoConfig {


    @Autowired
    private DataSourceProperties dataSourceProperties;

    @Bean
    @Primary
    public DruidDataSource druidDataSource(){
        DruidDataSource druidDataSource = new DruidDataSource();
        druidDataSource.setUrl(dataSourceProperties.getUrl());
        druidDataSource.setUsername(dataSourceProperties.getUsername());
        druidDataSource.setPassword(dataSourceProperties.getPassword());
        druidDataSource.setDriverClassName(dataSourceProperties.getDriverClassName());
        druidDataSource.setInitialSize(0);
        druidDataSource.setMaxActive(180);
        druidDataSource.setMaxWait(60000);
        druidDataSource.setMinIdle(0);
        druidDataSource.setValidationQuery("Select 1 from DUAL");
        druidDataSource.setTestOnBorrow(false);
        druidDataSource.setTestOnReturn(false);
        druidDataSource.setTestWhileIdle(true);
        druidDataSource.setTimeBetweenEvictionRunsMillis(60000);
        druidDataSource.setMinEvictableIdleTimeMillis(25200000);
        druidDataSource.setRemoveAbandoned(true);
        druidDataSource.setRemoveAbandonedTimeout(1800);
        druidDataSource.setLogAbandoned(true);
        return druidDataSource;
    }

    @Bean
    public DataSourceProxy dataSourceProxy(DruidDataSource druidDataSource){
        return new DataSourceProxy(druidDataSource);
    }

    @Bean
    public SqlSessionFactory sqlSessionFactory(DataSourceProxy dataSourceProxy) throws Exception {
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
        factoryBean.setDataSource(dataSourceProxy);
        factoryBean.setMapperLocations(new PathMatchingResourcePatternResolver()
                .getResources("classpath*:/mapper/*.xml"));
        return factoryBean.getObject();
    }

    @Bean
    public GlobalTransactionScanner globalTransactionScanner(){
        return new GlobalTransactionScanner("account-gts-seata-example", "account-service-seata-service-group");
    }
}

```

## 3.3、业务代码增加@GlobalTransactional

```java
	@GlobalTransactional(timeoutMills = 300000, name = "dubbo-gts-seata-example")
    @Override
    public ObjectResponse handleBusiness(BusinessDTO businessDTO) {
    
    }
```



