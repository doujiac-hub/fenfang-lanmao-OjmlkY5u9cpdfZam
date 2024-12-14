[合集 \- 消息队列(1\)](https://github.com)1\.RabbitMQ快速入门 整合 SpringBoot12\-13收起
# RabbitMQ快速入门 整合 SpringBoot


## 概述


1. ```
大多应用中，可通过消息服务中间件来提升系统异步通信、扩展解耦能力、流量削峰

```
2. ```
消息服务中两个重要概念：
消息代理（`message broker`）和目的地（`destination`）
当消息发送者发送消息以后，将由消息代理接管，消息代理保证消息传递到指定目的地。

```
3. ```
消息队列主要有两种形式的目的地
1.     队列（`queue`）：点对点消息通信（`point-to-point`）
2.     主题（`topic`）：发布（`publish`）/订阅（`subscribe`）消息通信

```


## RabbitMQ 架构图


![rabbitmq架构](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412131721524.svg)


## 概念


#### 生产者 Producer


​ 生产者是消息的发送方，它将消息发送到 RabbitMQ 的交换器中。


#### ✨消息 Message


* 消息\=消息头\+消息体，根据routekey发送到指定的交换机 Exchange
* 消息头：含有各种属性 **routing\-key**（路由键）、priority（优先级）、delivery\-mode（指出该消息可能需要持久性存储）等。


#### ✨消息代理 Broker


* 消息传递的中间件服务器，负责接收、存储和转发消息，作用类似邮局🏣
* 消息存储\+消息路由
* Broker \= VHost1\+Vhost2\+Vhost3\+.....
* Virtual Host \= Exchange \+ Queue \+Binding


#### 虚拟主机 Virtual Host


* 逻辑分组机制，将不同的用户、队列、交换器等资源隔离开来
* Virtual 即 VHost
* 默认目录 /


#### ✨交换机 Exchange


* 绑定 routekey接收消息并发送到符合routekey 的 队列
* 常用三种类型
	+ ✨**dirct**：**Direct Exchange（直连交换器）** 【**单播**】完全匹配**路由键**的队列
		- ![image-20241207003055398](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412131719657.png)
	+ ✨**fanout**：**Fanout Exchange（扇出交换器）**【**广播**】消息分发所有绑定队列上，不处理路由键
		- ![image-20241207003049286](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412131719560.png)
	+ ✨**topic**：**Topic Exchange（主题交换器）**【**模式匹配**】
		- `#`:配置0个或者多个单词
		- `*`：匹配一个单词
		- ![image-20241207003038106](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412131720772.png)
	+ headers：很少使用
	+ system：很少使用


#### ✨队列 Queue


* 存储消息的容器，FIFO
* 缓冲消息\+持久化


#### 绑定 Binding


* 用于消息队列和交换器之间的关联。
* 一个绑定就是基于路由键将交换器和消息队列连接起来的路由规则，所以可以将交换器理解成一个由绑定构成的路由表。
* Exchange 和Queue的绑定可以是多对多的关系。


#### 连接 Connection


* 网络连接，比如一个TCP连接


#### 信道 Channel


* 信道，多路复用连接中的一条**独立的双向数据流通道**。信道是建立在真实的TCP连接内的虚拟连接，AMQP 命令都是通过信道发出去的，不管是**发布消息**、**订阅队列**还是**接收消息**，这些动作都是通过信道完成。因为对于操作系统来说建立和销毁 TCP 都是非常昂贵的开销，所以引入了信道的概念，以复用一条 TCP 连接。


#### 消费者 Consumer


​ 消费者是消息的接收方，它从 RabbitMQ 的队列中获取消息并进行处理。


## Docker 安装 RMQ



```
docker run -d --restart=always --name rabbitmq -p 5671:5671 -p 5672:5672 -p 4369:43699-p25672:25672-p 15671:15671 -p 15672:15672 rabbitmq:management

```

## 后台页面收发消息


打开 localhost:15672


### 登录页面



```
登录的用户密码：guest/guest

```

![登录页面](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412131720654.png)


### 登录后台首页


![首页](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412131720356.png)


### 交换机 Exchange页面


![交换机页面](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412131720609.png)


五种交换机类型：**direct**、**fanout**、headers、**topic**、x\-local\-random


![新增交换机](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412131720740.png)


### 队列页面


![队列页面](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412131720697.png)


### 绑定：交换机根据路由键绑定到对应的队列



> **Virtual Host**【Exchange \-\-\> binding(**route\-key**) 】\-\-\> Queue(**route\-key**)
> 
> 
> 默认的虚拟主机的路径是 "/"，即**根目录**


**交换机和队列绑定**


![交换机绑定队列](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412131720696.png)


**队列和交换机绑定关系**


![队列和交换机绑定关系](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412131720966.png)


## SpringBoot 整合


### 配置pom 文件



```

<dependency>
    <groupId>org.springframework.bootgroupId>
    <artifactId>spring-boot-starter-amqpartifactId>
dependency>

```

### 配置application.yaml



```
server:
  port: 8081
spring:
  application:
    name: rabbitmq-demo
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
    virtualHost: /
#    publisher-confirm-type: CORRELATED
#    publisher-returns: true
#    listener:
#      simple:
#        acknowledge-mode: manual #默认情况下消息消费者是自动确认消息的，如果要手动确认消息则需要修改确认模式为manual
#        prefetch: 1 # 消费者每次从队列获取的消息数量。此属性当不设置时为：轮询分发，设置为1为：公平分发

```

### 测试类中\-创建交换机



```
@Slf4j
@SpringBootTest
class RabbitmqDemoApplicationTests {

    @Autowired
    AmqpAdmin amqpAdmin;

    @Test
    public void createExchange() {
        DirectExchange directExchange = new DirectExchange("hello-java-exchange", true, false);
        amqpAdmin.declareExchange(directExchange);
        log.info("Exchange[hello-java-exchange] 创建完成");
    }
}


```

**成功创建**![image-20241206170654862](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412131720837.png)


### 创建队列



```
@Test
public void createQueue() {
    Queue queue = new Queue("hello-java-queue", true, false, false);
    amqpAdmin.declareQueue(queue);
    log.info("Queue[hello-java-queue] 创建完成");
}

```

执行后成功创建


![image-20241206170811552](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412131720822.png)


### 创建绑定



```
@Test
public void Binding() {
    Binding binding = new Binding("hello-java-queue",
                                  Binding.DestinationType.QUEUE,
                                  "hello-java-exchange",
                                  "hello-java", null);
    amqpAdmin.declareBinding(binding);
    log.info("Binding[hello-java-binding] 创建完成");
}

```

**直连交换机**【`hello-java-exchang`e】和队列【`hello-java-queue`】用 `routingkey` 【`hello-java`】绑定


**队列绑定**


![image-20241206170917982](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412131720396.png)


**交换机绑定**


![image-20241206171001858](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412131720598.png)


### 发送消息【JSON消息转换器】


配置 `RabbitConfig` 序列化 `json`



> 根据源码 `RabbitAutoConfiguration` 创建`@Bean RabbitTemplate` 中的消息转换器属性 `MessageConverter messageConverter = new SimpleMessageConverter();`
> 
> 
> 说明了`RabbitMQ` 自动配置过程中，创建工具类【`RabbitTemplate`】，其中默认的消息转换器是 【`SimpleMessageConverter`】，我们来看下【`SimpleMessageConverter`】源码是如何收发消息的


**`SimpleMessageConverter`创建消息**



```
// 创建消息,默认使用序列化 Serializable类型发送，发送的消息实体需要实现序列化
protected Message createMessage(Object object, MessageProperties messageProperties) throws MessageConversionException {
    if (object instanceof byte[] bytes) {
        messageProperties.setContentType("application/octet-stream");
    } else if (object instanceof String) {
        try {
            bytes = ((String)object).getBytes("UTF-8");
        } catch (UnsupportedEncodingException e) {
            throw new MessageConversionException("failed to convert to Message content", e);
        }
        messageProperties.setContentType("text/plain");
        messageProperties.setContentEncoding("UTF-8");
    } else if (object instanceof Serializable) {
        try {
            bytes = SerializationUtils.serialize(object);
        } catch (IllegalArgumentException e) {
            throw new MessageConversionException("failed to convert to serialized Message content", e);
        }
        messageProperties.setContentType("application/x-java-serialized-object");
    }
    if (bytes != null) {
        messageProperties.setContentLength((long)bytes.length);
        return new Message(bytes, messageProperties);
    } else {
        String var10002 = this.getClass().getSimpleName();
        throw new IllegalArgumentException(var10002 + " only supports String, byte[] and Serializable payloads, received: " + object.getClass().getName());
    }
}

```

**`SimpleMessageConverter`消费消息**



```
public Object fromMessage(Message message) throws MessageConversionException {
        Object content = null;
        MessageProperties properties = message.getMessageProperties();
        if (properties != null) {
            String contentType = properties.getContentType();
            if (contentType != null && contentType.startsWith("text")) {
                String encoding = properties.getContentEncoding();
                if (encoding == null) {
                    encoding = this.defaultCharset;
                }

                try {
                    content = new String(message.getBody(), encoding);
                } catch (UnsupportedEncodingException e) {
                    throw new MessageConversionException("failed to convert text-based Message content", e);
                }
            } else if (contentType != null && contentType.equals("application/x-java-serialized-object")) {
                try {
                    content = SerializationUtils.deserialize(this.createObjectInputStream(new ByteArrayInputStream(message.getBody())));
                } catch (IllegalArgumentException | IllegalStateException | IOException e) {
                    throw new MessageConversionException("failed to convert serialized Message content", e);
                }
            }
        }

        if (content == null) {
            content = message.getBody();
        }

        return content;
    }

```

**自定义消息类型转器 `MessageConverter`**



> `MessageConverter` 的层次结构


![image-20241207013346381](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412131720416.png)


**自定义消息类型转换器**



```
import org.springframework.amqp.support.converter.Jackson2JsonMessageConverter;
import org.springframework.amqp.support.converter.MessageConverter;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class RabbitMQConfig {

    @Bean
    public MessageConverter jsonMessageConverter() {
        return new Jackson2JsonMessageConverter();
    }

}

```

创建数据：**订单退出原因实体对象** 注意需要序列化 `Serializable`



```
@ToString
@Data
@Accessors(chain = true)
//@TableName("oms_order_return_reason")
public class OrderReturnReasonEntity implements Serializable {
	private static final long serialVersionUID = 1L;

	private Long id;
	/**
	 * 退货原因名
	 */
	private String name;
	/**
	 * 排序
	 */
	private Integer sort;
	/**
	 * 启用状态
	 */
	private Integer status;
	/**
	 * create_time
	 */
	private Date createTime;

}

```

测试类中发送消息



```
@Autowired
RabbitTemplate rabbitTemplate;

@Test
public void sendMessage() {
    OrderReturnReasonEntity data = new OrderReturnReasonEntity();
    data.setId(1L)
            .setCreateTime(new Date())
            .setName("测试");
    rabbitTemplate.convertAndSend("hello-java-exchange", "hello-java", data);
    log.info("发送消息: {}", data);
}

```

**队列收到消息**


![image-20241206192848893](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412131720069.png)


**收到消息对象**


![image-20241206193039689](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412131720465.png)



```
{"id":1,"name":"测试","sort":null,"status":null,"createTime":1733484472414}

```

### 接收信息


在**启动类**上添加 `@EnableRabbit` 开启 RabbitMQ



```
@EnableRabbit
@SpringBootApplication
public class RabbitmqDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(RabbitmqDemoApplication.class, args);
    }
}

```

在需要接收消息的地方添加方法 `@RabbitListerner` 、`@RabbitHandler`



> @`RabbitListerner` ：用在类和方法上并绑定对应的队列
> 
> 
> @`RabbitHandler`：用在方法上，可以接收**不同的类型的数据**



```
@RabbitListener(queues = {"hello-java-queue"})
@Component
@Slf4j
public class OrderMQHandler {
    @RabbitHandler
    public void receiveOrderReturnReason(Message message, OrderReturnReasonEntity content, Channel channel) {
        //消息体
        byte[] body = message.getBody();
        //消息头配置
        MessageProperties messageProperties = message.getMessageProperties();
        log.info("消息体内容：{}", content);
    }

    @RabbitHandler
    public void receiverOrder(OrderEntity content) {
        log.info("接收消息=>Order：{}", content);
    }
}

```

成功收到`OrderReturnReasonEntity`对象数据



```
2024-12-06T22:46:37.495+08:00  INFO 15808 --- [ntContainer#0-3] c.s.rabbitmqdemo.handler.OrderMQHandler  : 消息体内容：OrderReturnReasonEntity(id=1, name=测试-0, sort=null, status=null, createTime=Fri Dec 06 22:46:37 CST 2024)

```

成功收到`OrderEntity`对象数据



```
2024-12-06T22:46:37.522+08:00  INFO 15808 --- [ntContainer#0-3] c.s.rabbitmqdemo.handler.OrderMQHandler  : 接收消息=>Order：OrderEntity(id=1, memberId=null, orderSn=null, couponId=null, createTime=Fri Dec 06 22:46:37 CST 2024, memberUsername=测试-1, totalAmount=null, payAmount=null, freightAmount=null, promotionAmount=null, integrationAmount=null, couponAmount=null, discountAmount=null, payType=null, sourceType=null, status=null, deliveryCompany=null, deliverySn=null, autoConfirmDay=null, integration=null, growth=null, billType=null, billHeader=null, billContent=null, billReceiverPhone=null, billReceiverEmail=null, receiverName=null, receiverPhone=null, receiverPostCode=null, receiverProvince=null, receiverCity=null, receiverRegion=null, receiverDetailAddress=null, note=null, confirmStatus=null, deleteStatus=null, useIntegration=null, paymentTime=null, deliveryTime=null, receiveTime=null, commentTime=null, modifyTime=null)

```

  * [RabbitMQ快速入门 整合 SpringBoot](#rabbitmq%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8-%E6%95%B4%E5%90%88-springboot)
* [概述](#%E6%A6%82%E8%BF%B0)
* [RabbitMQ 架构图](#rabbitmq-%E6%9E%B6%E6%9E%84%E5%9B%BE)
* [概念](#%E6%A6%82%E5%BF%B5)
* [生产者 Producer](#%E7%94%9F%E4%BA%A7%E8%80%85-producer)
* [✨消息 Message](#%E6%B6%88%E6%81%AF-message)
* [✨消息代理 Broker](#%E6%B6%88%E6%81%AF%E4%BB%A3%E7%90%86-broker)
* [虚拟主机 Virtual Host](#%E8%99%9A%E6%8B%9F%E4%B8%BB%E6%9C%BA-virtual-host)
* [✨交换机 Exchange](#%E4%BA%A4%E6%8D%A2%E6%9C%BA-exchange)
* [✨队列 Queue](#%E9%98%9F%E5%88%97-queue)
* [绑定 Binding](#%E7%BB%91%E5%AE%9A-binding)
* [连接 Connection](#%E8%BF%9E%E6%8E%A5-connection)
* [信道 Channel](#%E4%BF%A1%E9%81%93-channel)
* [消费者 Consumer](#%E6%B6%88%E8%B4%B9%E8%80%85-consumer)
* [Docker 安装 RMQ](#docker-%E5%AE%89%E8%A3%85-rmq)
* [后台页面收发消息](#%E5%90%8E%E5%8F%B0%E9%A1%B5%E9%9D%A2%E6%94%B6%E5%8F%91%E6%B6%88%E6%81%AF)
* [登录页面](#%E7%99%BB%E5%BD%95%E9%A1%B5%E9%9D%A2):[豆荚加速器官网PodHub](https://doujiaa.com)
* [登录后台首页](#%E7%99%BB%E5%BD%95%E5%90%8E%E5%8F%B0%E9%A6%96%E9%A1%B5)
* [交换机 Exchange页面](#%E4%BA%A4%E6%8D%A2%E6%9C%BA-exchange%E9%A1%B5%E9%9D%A2)
* [队列页面](#%E9%98%9F%E5%88%97%E9%A1%B5%E9%9D%A2)
* [绑定：交换机根据路由键绑定到对应的队列](#%E7%BB%91%E5%AE%9A%E4%BA%A4%E6%8D%A2%E6%9C%BA%E6%A0%B9%E6%8D%AE%E8%B7%AF%E7%94%B1%E9%94%AE%E7%BB%91%E5%AE%9A%E5%88%B0%E5%AF%B9%E5%BA%94%E7%9A%84%E9%98%9F%E5%88%97)
* [SpringBoot 整合](#springboot-%E6%95%B4%E5%90%88)
* [配置pom 文件](#%E9%85%8D%E7%BD%AEpom-%E6%96%87%E4%BB%B6)
* [配置application.yaml](#%E9%85%8D%E7%BD%AEapplicationyaml)
* [测试类中\-创建交换机](#%E6%B5%8B%E8%AF%95%E7%B1%BB%E4%B8%AD-%E5%88%9B%E5%BB%BA%E4%BA%A4%E6%8D%A2%E6%9C%BA)
* [创建队列](#%E5%88%9B%E5%BB%BA%E9%98%9F%E5%88%97)
* [创建绑定](#%E5%88%9B%E5%BB%BA%E7%BB%91%E5%AE%9A)
* [发送消息【JSON消息转换器】](#%E5%8F%91%E9%80%81%E6%B6%88%E6%81%AFjson%E6%B6%88%E6%81%AF%E8%BD%AC%E6%8D%A2%E5%99%A8)
* [接收信息](#%E6%8E%A5%E6%94%B6%E4%BF%A1%E6%81%AF)

   \_\_EOF\_\_

       - **本文作者：** [Stormling](https://github.com)
 - **本文链接：** [https://github.com/stormling2022/p/18605383](https://github.com)
 - **关于博主：** 评论和私信会在第一时间回复。或者[直接私信](https://github.com)我。
 - **版权声明：** 本博客所有文章除特别声明外，均采用 [BY\-NC\-SA](https://github.com "BY-NC-SA") 许可协议。转载请注明出处！
 - **声援博主：** 如果您觉得文章对您有帮助，可以点击文章右下角**【[推荐](javascript:void(0);)】**一下。
     
