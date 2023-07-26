---
title: RabbitMQ
date: 2023/07/24
categories:
  - coding
tags:
  - RabbitMQ
  - 消息队列
  - 编程基础
abbrlink: 33708
---
# 1、基本基础
## 1.1、mq概念
MQ全称Message Queue（消息队列），是在消息传输过程中保存消息的容器。多用于分布式系统之间进行通信
## 1.2、mq优缺点
**优势：**
（1）应用解耦:eg：订单系统不直接调用库存系统，库存系统宕机并不影响下单
（2）异步提速：eg：订单系统不需要同步调用库存系统，提升速度
（3）削峰填谷：eg：mq承载了5000请求，系统慢慢消费，就是削峰。但是因为消息积压，高峰过一段时间后消息才能被消费完，这就是填谷。  

**劣势：**
（1）系统可用性降低：一旦mq宕机，就可能对多个业务造成影响。如何保证mq高可用
（2）系统复杂度提高：如何保证消息不被重复消费？怎么处理丢失情况？怎么保证消息传递的顺序性？
（3）一致性问题：A系统给B,C,D系统发送数据，如果B，c处理成功，D系统失败。如何保证消息数据处理的一致性
## 1.3、mq产品选型

|指标 | **ActiveMQ** | **RabbitMQ** | **RocketM Q** | **Kafka** |
| --- | --- | --- | --- | --- |
| 数据量级（每秒） | 万级 | 万级 | 十万级 | 十万级 |
| 可靠性 | 低概率丢失 | 0丢失 | 0丢失 | 0丢失 |
| 可用性 | 主从架构 | 主从架构 | 分布式 | 分布式 |
| 时效性 | 毫秒 | 微秒 | 毫秒 | 毫秒 |

> 一个简单粗暴的判断方法：大量数据、日志采集用Kafka，需要高可靠性高并发性用RocketMQ，数据量不大的一般系统用RabbitMQ。

## 1.4、AMQP协议
 AMQP，即Advanced Message Queuing Protocol,一个提供统一消息服务的应用层标准高级消息队列协议，是**应用层协议**的一个开放标准,为面向消息的中间件设计。基于此协议的客户端与消息中间件可传递消息，并不受客户端/中间件同产品，不同的开发语言等条件的限制 1

## 1.5、rabbitMQ安装

1. 安装Eralng，下面链接已提供otp_win64_20.2.exe

链接： [https://pan.baidu.com/s/1lmvCMPVAV1Ba9UogCdQpZg](https://pan.baidu.com/s/1lmvCMPVAV1Ba9UogCdQpZg)
提取码：x9m7

2. 安装rabbitmq，下面链接已提供rabbitmq-server-3.7.4.exe

链接： [https://pan.baidu.com/s/1CPfhg5X1e7UitpgMWIcAEg](https://pan.baidu.com/s/1CPfhg5X1e7UitpgMWIcAEg)
提取码：h4r3

3. 配置erlang环境变量

![配置erlang环境变量](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202307241037440.png)



4. 测试erlang

![测试erlang](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202307241037374.png)


5. 启动rabbitmq

![启动rabbitmq](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202307241037002.png)


6. 访问 [http://127.0.0.1:15672/](http://127.0.0.1:15672/)      guest guest


# 2、组件
## 2.1、RabbitMQ架构
![RabbitMQ架构](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202307241048618.png)


（1）broker（服务端）：接收客户端的连接，实现AMQP实体服务。
（2）Connection：连接，应用程序与Server的网络连接，TCP连接。
（3）Channel：信道，消息读写等操作在信道中进行。客户端可以建立多个信道，每个信道代表一个会话任务。
（4）Virtual Host：虚拟主机，用于逻辑隔离。一个虚拟主机里面可以有若干个Exchange和Queue，同一个虚拟主机里面不能有相同名称的Exchange或Queue。
（5）Exchange：交换器，接收消息，按照路由规则将消息路由到一个或者多个队列。如果路由不到，或者返回给生产者，或者直接丢弃。RabbitMQ常用的交换器常用类型有direct、topic、fanout、headers四种，后面详细介绍。
（6）Queue：消息队列，用来保存消息，供消费者消费
（7）Message：消息，应用程序和服务器之间传送的数据，消息可以非常简单，也可以很复杂。有Properties和Body组成。Properties为外包装，可以对消息进行修饰，比如消息的优先级、延迟等高级特性；Body就是消息体内容。
（8）Binding：绑定，交换器和消息队列之间的虚拟连接，绑定中可以包含一个或者多个RoutingKey。
（9）RoutingKey：路由键，生产者将消息发送给交换器的时候，会发送一个RoutingKey，用来指定路由规则，这样交换器就知道把消息发送到哪个队列。路由键通常为一个“.”分割的字符串，例如“com.rabbitmq”

## 2.2、4种Exchange交换机类型
### 2.2.1、Direct Exchange（直连交换机）

根据Routing Key(路由键)进行投递到不同队列。如果路由键不匹配，那么就不会发送到任何队列中去。

![直连交换机](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202307241551705.png)


### 2.2.2、Fanout Exchange（广播交换机）
该类型的交换机会将⼀条消息⼴播到绑定到该交换机的所有队列上，不论你设置的路由键是什么
> 如果想让多个消费者消费到数据必须不指定queues，指定交换机

```java
@RabbitListener(bindings = @QueueBinding(

        value = @Queue(), //注意这里不要定义队列名称,系统会随机产生

        exchange = @Exchange(value = "business_rrpc_exchange",type = ExchangeTypes.FANOUT)

))


```
### 2.2.3、Topic Exchange（主题交换机）

将路由键和某模式进行匹配。此时队列需要绑定要一个模式上。符号“#”匹配一个或多个词，符号“\*”匹配不多不少一个词。因此“abc.#”能够匹配到“abc.def.ghi”，但是“abc.\* ” 只会匹配到“abc.def”。

![主题交换机](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202307241552123.png)


### 2.2.4、Headers Exchanges（头交换机）

与routingKey无关，匹配机制是匹配消息头中的属性信息。在绑定消息队列与交换机之前声明一个map键值对，通过这个map对象实现消息队列和交换机的绑定。当消息发送到RabbitMQ时会取到该消息的headers与Exchange绑定时指定的键值对进行匹配；如果完全匹配则消息会路由到该队列，否则不会路由到该队列()

> 匹配规则x-match有下列两种类型：
x-match = all ：表示所有的键值对都匹配才能接受到消息
x-match = any ：表示只要有键值对匹配就能接受到消息

![image (6).png](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202307241557662.png)


## 2.3、工作模式
### 2.3.1、简单模式
一个生产者将消息发送到一个队列中，一个消费者从这个队列中获取消息并进行处理。这种模式仅适用于单个生产者和单个消费者的场景

![简单模式](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202307241557273.png)


> P代表生产者，C代表消费者，红色代表消息队列。P将消息发送到消息队列，C对消息进行处理


```java
// 1. 创建队列
@Bean
public Queue Queue() {
    return new Queue("hello");
}

// 2. 生产者
@RestController
public class Producer {
    @Autowired
    AmqpTemplate amqpTemplate;

    @RequestMapping("/send")
    public String send() {
        String content = "hello,rabbitmq";
        amqpTemplate.convertAndSend("queue1", content);
        return content;
    }
}

// 3. 消费者
@Component
public class Consumer {
    @RabbitListener(queues = "queue1")
    @RabbitHandler
    public void getMsg(String msg, Channel channel, Message message) throws IOException {
        System.out.println(msg);
    }
}

```


### 2.3.2、工作队列模式
一个生产者将消息发送到一个队列中，多个消费者从这个队列中获取消息并进行处理。这种模式可以提高消息的处理效率

> 对于任务过重或任务较多情况使用工作队列可以提高任务处理的速度

![工作队列模式](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202307241632591.png)


**实现方式**
多个消费者同时监听同一个队列,消息被消费，共同争抢当前的消息队列内容,谁先拿到谁负责消费消息
```java
// 增加一个消费者监听同一个队列
@Component
public class Consumer2 {
    @RabbitListener(queues = "queue1")
    @RabbitHandler
    public void getMsg(String msg, Channel channel, Message message) throws IOException {
        System.out.println("consumer2: "+msg);
    }
}
```
![image.png](https://cdn.nlark.com/yuque/0/2022/png/2996398/1661394067626-05e7a3e3-889c-4b32-aac4-b411287a4c20.png#averageHue=%2334322f&clientId=u1f2092d6-850c-4&from=paste&height=268&id=u834eba4b&originHeight=268&originWidth=276&originalType=binary&ratio=1&rotation=0&showTitle=false&size=7316&status=done&style=none&taskId=u30d0eca9-feeb-475a-8f5e-e2cbcd309fb&title=&width=276)

### 2.3.3、发布订阅模式
一个生产者将消息发送到一个交换机中，交换机将消息广播到所有绑定的队列中，多个消费者可以分别从这些队列中获取消息并进行处理。这种模式适用于需要将消息广播到多个消费者的场景

![发布订阅模式](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202307241633780.png)


1. **实现方式**

（1）创建两个队列和一个交换机，然后将队列绑定到交换机上 

```java
@Bean
public Queue queue1() {
    return new Queue("queue1", true);
}

@Bean
public Queue queue2() {
    return new Queue("queue2", true);
}

@Bean
FanoutExchange fanoutExchange1() {
    return new FanoutExchange("fanoutExchange1");
}

@Bean
Binding bindingExchangeA(Queue queue1, FanoutExchange fanoutExchange1) {
    return BindingBuilder.bind(queue1).to(fanoutExchange1);
}

@Bean
Binding bindingExchangeB(Queue queue2, FanoutExchange fanoutExchange1) {
    return BindingBuilder.bind(queue2).to(fanoutExchange1);
}
```

（2）生产者消费者

```java
// 生产者：第二个参数是队列名，设置为空
@RestController
public class Producer {
    @Autowired
    AmqpTemplate amqpTemplate;

    @RequestMapping("/send")
    public String send() {
        String content = "hello,rabbitmq";
        for (int i=0;i<5;i++){
            amqpTemplate.convertAndSend("fanoutExchange1","", content);
        }
        return content;
    }
}
// 消费者1
@Component
public class Consumer1 {
    @RabbitListener(queues = "queue1")
    @RabbitHandler
    public void getMsg(String msg, Channel channel, Message message) throws IOException {
        System.out.println("consumer1: "+msg);
    }
}
// 消费者2
@Component
public class Consumer2 {
    @RabbitListener(queues = "queue2")
    @RabbitHandler
    public void getMsg(String msg, Channel channel, Message message) throws IOException {
        System.out.println("consumer2: "+msg);
    }
}
```
![image.png](https://cdn.nlark.com/yuque/0/2022/png/2996398/1661413378872-162ba357-f8e8-44ca-858c-884732faf8fd.png#averageHue=%23353230&clientId=u1f2092d6-850c-4&from=paste&height=258&id=ub5bb0f8d&originHeight=258&originWidth=339&originalType=binary&ratio=1&rotation=0&showTitle=false&size=32505&status=done&style=none&taskId=u45d190ce-12a2-4723-afca-16e723e524b&title=&width=339)

### 2.3.4、路由模式
一个生产者将消息发送到一个交换机中，交换机根据消息的Routing Key将消息路由到对应的队列中，多个消费者可以从这些队列中获取消息并进行处理。这种模式适用于需要根据消息的路由键进行精确匹配的场景

![路由模式](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202307241647936.png)

**实现方式**
（1）交换机和队列根据指定路由规则绑定

```java
@Bean
public Queue queue1() {
    return new Queue("queue1", true);
}

@Bean
public Queue queue2() {
    return new Queue("queue2", true);
}

@Bean
DirectExchange directExchange1() {
    return new DirectExchange("directExchange1");
}

@Bean
Binding bindingExchangeMessage1(Queue queue1, DirectExchange directExchange1) {
    return BindingBuilder.bind(queue1).to(directExchange1).with("routingKey1");
}


@Bean
Binding bindingExchangeMessage2(Queue queue2, DirectExchange directExchange1) {
    return BindingBuilder.bind(queue2).to(directExchange1).with("routingKey2");
}
```

（2）生产者和消费者
```java
// consumer1
@Component
public class Consumer1 {
    @RabbitListener(queues = "queue1")
    @RabbitHandler
    public void getMsg(String msg, Channel channel, Message message) throws IOException {
        System.out.println("queue1 consumer1: "+msg);
    }
}
// consumer2
@Component
public class Consumer2 {
    @RabbitListener(queues = "queue2")
    @RabbitHandler
    public void getMsg(String msg, Channel channel, Message message) throws IOException {
        System.out.println("queue2 consumer2: "+msg);
    }
}
// producer
@RestController
public class Producer {
    @Autowired
    AmqpTemplate amqpTemplate;

    @RequestMapping("/send")
    public String send() {
        String content = "hello,rabbitmq";
        for (int i=0;i<5;i++){
            amqpTemplate.convertAndSend("fanoutExchange1","routingKey1", content);
        }
        return content;
    }
}
```

![image.png](https://cdn.nlark.com/yuque/0/2022/png/2996398/1661473083570-947e0de2-9c97-47fb-8fba-4e568d152f28.png#averageHue=%23353330&clientId=u1f2092d6-850c-4&from=paste&height=129&id=u8c943953&originHeight=129&originWidth=335&originalType=binary&ratio=1&rotation=0&showTitle=false&size=16674&status=done&style=none&taskId=uc8eec929-83ad-43c6-bda3-fb8dbe96e14&title=&width=335)


# 3、高级特性

## 3.1、保证消息传递的可靠性
RabbitMQ保证消息的可靠性主要分为两个部分：消息投递和消费者消息确认
（1）投递确认：**confirm确认模式**（producer——>exchange），**return退回模式**（exchange——>queue）
（2）消费者确认：ACK消息签收机制，表示消费者收到消息后的确认方式

### 3.1.1、confirm确认模式
消息从 producer 到 rabbitmq broker有一个 confirmCallback 确认模式。(无论成功失败都有返回)


### 3.1.2、return退回模式

消息从 exchange 到 queue 投递失败有一个 returnCallback 退回模式。（失败时才会有返回）


**实现confirm callback和return callback：**

1. 配置文件开启相关配置

```yml
spring:
  #配置rabbitMq 服务器
  rabbitmq:
    host: 127.0.0.1
    port: 5672
    username: yancey
    password: yancey
 
    # confirmCallback 确认模式
    # SIMPLE       禁用发布确认模式，是默认值
	# CORRELATED   发布消息成功到交换器或失败后 会触发回调方法
	# NONE         有两种效果，其一效果和CORRELATED值一样会触发回调方法，其二在发布消息成功后使用。rabbitTemplate调用waitForConfirms或waitForConfirmsOrDie方法等待broker节点返回 发送结果，根据返回结果来判定下一步的逻辑，要注意的点是waitForConfirmsOrDie方法如果 返回false则会关闭channel，则接下来无法发送消息到broker;
    publisher-confirm-type: correlated
 
    # returnCallback 退回模式
    publisher-returns: true
```

2. 编写配置类

```java
@Configuration
public class RabbitConfig {
 
    @Bean
    public RabbitTemplate createRabbitTemplate(ConnectionFactory connectionFactory) {
        RabbitTemplate rabbitTemplate = new RabbitTemplate();
        rabbitTemplate.setConnectionFactory(connectionFactory);
 
        //设置消息投递失败的策略，有两种策略：自动删除或返回到客户端。
        //我们既然要做可靠性，当然是设置为返回到客户端(true是返回客户端，false是自动删除)
        rabbitTemplate.setMandatory(true);
 
        rabbitTemplate.setConfirmCallback(new RabbitTemplate.ConfirmCallback() {
            @Override
            public void confirm(CorrelationData correlationData, boolean ack, String cause) {
                log.info("相关数据：" + correlationData);
                if (ack) {
                    log.info("投递成功,确认情况：" + ack);
                } else {
                    log.info("投递失败,确认情况：" + ack);
                    log.info("原因：" + cause);
                }
            }
        });
 
        rabbitTemplate.setReturnCallback(new RabbitTemplate.ReturnCallback() {
            @Override
            public void returnedMessage(Message message, int replyCode, String replyText, String exchange, String routingKey) {
               log.info("ReturnCallback:     " + "消息：" + message);
               log.info("ReturnCallback:     " + "回应码：" + replyCode);
               log.info("ReturnCallback:     " + "回应信息：" + replyText);
               log.info("ReturnCallback:     " + "交换机：" + exchange);
               log.info("ReturnCallback:     " + "路由键：" + routingKey);
            }
        });
 
        return rabbitTemplate;
    }
}
```


### 3.1.3、消费者端ack机制
消费者端消息接收确认采用的是ack模式。ACK机制是消费者从RabbitMQ收到消息并处理完成后，反馈给RabbitMQ，RabbitMQ收到反馈后才将此消息从队列中删除

1. rabbitmq默认的是自动ack，无需添加其他配置

若正常消费成功了，则会自动返回确认ack给队列，队列收到后即可将消息移除。
若消费过程中出现异常，则超过ack心跳时间，会触发重试消费

2. 手动ack

开启方式简单，只需要放开此配置即可
```java
acknowledge-mode: manual # 设置消费端手动 ack
```

消费者类
```java
@Component
public class Consumer {
    @RabbitListener(queues = "queue1")
    @RabbitHandler
    public void getMsg(String msg, Channel channel, Message message) throws IOException {
        try {
            int i = 10/0;
            // 消费成功后ack
            channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
        }catch (Exception e){
            // 消费失败后ack
            // 三个参数：
            // （1）delivery_tag：表示消息的唯一标识符
            // （2）multiple：表示是否将delivery_tag之前的所有未确认消息都拒绝。如果multiple为true，则RabbitMQ将拒绝所有未确认的消息，如果为false，则只拒绝指定的消息
            // （3）requeue：表示是否将被拒绝的消息重新放回队列中。如果requeue为true，则消息将返回到队列中以便重新处理，如果为false，则消息将被丢弃
            channel.basicNack(message.getMessageProperties().getDeliveryTag(),false,false);
		
            if (message.getMessageProperties().getRedelivered()) {//判断是否已经重试过
                log.error("消息已重复处理失败,拒绝再次接收...");
                channel.basicReject(message.getMessageProperties().getDeliveryTag(), false); // 拒绝消息
                // 重复消费失败的消息入库...
            } else {
                log.error("消息即将再次返回队列处理...");
                channel.basicNack(message.getMessageProperties().getDeliveryTag(), false, true);
            }
		
        }
    }
}
```


>requeue参数设置为true，可以将消息返回到队列中以便重新处理
>但是这样可能导致无限循环地处理同一个错误消息
>所以上面代码采用了折中方案：首次失败的消息通知队列重发，重复失败的消息落地进行后面的补偿机制



## 3.2、消息持久化机制
持久化是提高RabbitMQ 可靠性的基础，否则当 RabbitMQ 遇到异常时（如：重启、断电、停机等）数据将会丢失。主要从以下几个方面来保障消息的持久性：

1. Exchange 的持久化。通过定义时设置  durable 参数为  ture  来保证  Exchange  相关的元数据不丢失。
2. Queue 的持久化。也是通过定义时设置  durable 参数为  ture  来保证  Queue  相关的元数据不丢失。
3. 消息 的持久化。通过将消息的投递模式  (BasicProperties 中的 deliveryMode 属性 )  设置为 2 即可实现消息的持久化，保证消息自身不丢失。

>  注:Exchange 和 Queue 的持久化只能保证 Exchange  跟 Queue 在RabbitMQ重启之后仍然存在，如果消息没有设置持久化的话，仅设置 Exchange 和 Queue 的持久化，消息仍然会丢失，想要保证消息不丢失， 交换机，队列，消息 三者的持久化缺一不可

### 3.2.1、交换机持久化
在申明exchange的时候，有个参数：durable。当该参数为true，则对该exchange做持久化，重启rabbitmq服务器，该exchange不会消失。durable的默认值为true
```java
// autoDelete:当所有绑定队列都不在使用时，是否自动 删除
public DirectExchange(String name, boolean durable, boolean autoDelete) {
        super(name, durable, autoDelete);
}

```

### 3.2.2、队列持久化
申明队列时也有个参数：durable。当该参数为true，则对该queue做持久化，重启rabbitmq服务器，该queue不会消失。durable的默认值为true
```java
// durable: 是否做队列持久化
// exclusive: 是否排外。两个作用：（1）当连接关闭时connection.close()该队列是否会自动删除（2）对当前队列加锁，其他通道channel是不能访问的，用于一个队列只能有一个消费者来消费的场景
// autoDelete:当所有消费客户端连接断开后，是否自动删除 
public Queue(String name, boolean durable, boolean exclusive, boolean autoDelete) {
        this(name, durable, exclusive, autoDelete, (Map)null);
}
```

## 3.3、避免消息重复消费
### 3.3.1、消息重复发送的场景
消费者消费消息成功后，在给MQ发送消息确认的时候出现了网络异常(或者是服务中断)，MQ没有接收到确认，此时MQ不会将发送的消息删除，会继续给消费者投递之前的消息。这时候消费者就接收到了两条一样的消息

### 3.3.2、解决方案
保证消费者的幂等性（调用方，对一个系统进行重复调用（参数全部相同），不论重复调用多少次，这些调用对系统的影响都是相同的效果）

如何保证幂等性？
1. 使用代码的逻辑判断，判断消息状态是否已经被消费过了

> 使用数据库一个表来记录消息的状态（或者用redis来记录也可以）。每次消费之前，都查询判断消息的状态，是否已经被消费了。这个状态可以是id。例如，如果消息是订单，而且id是全局唯一的，那么只需要拿这个订单id来做判断即可。

2. 使用token，要申请，一次有效性。

> 在创建订单的场景下。首先，先生成一个token，返回给客户端存起来，同时也在后端存起来（redis）。当他创建订单的时候，带着这个token来请求后端，后端判断redis里是否存在，如果存在，则操作成功，同时删除token（删除了之后，就算他重复多次调用，前边的判断不成立，这样子就不能多次操作了）


## 3.4、避免消息积压问题
### 3.4.1、消息积压场景
消费者宕机/消费能力不足，没有人去消费了，但依旧源源不断生产消息，导致消息积压
### 3.4.2、解决方案

1. 上线更多的消费者（如：库存服务），进行正常消费
2. 上线专门的队列消费服务，将消息先批量取出来，记录数据库，离线慢慢处理。

## 3.5、保证消息的顺序性
### 3.5.1、消息顺序错乱场景

生产者向一个消息队列发送 创建学生信息 与 更新学生信息 两条消息。如果有两个消费者，可能同时一个消费者做创建学生的操作，另外一个消费者做更新学生的操作。那么就有可能发生，更新学生基本信息的操作早于创建学生基本信息的操作。这样的话更新就会失败。

### 3.5.2、解决方案
保证队列与消费者一对一
思路就是拆分队列，使得每个队列只有一个消费者，这样消费者一定是按照顺序消费的 

