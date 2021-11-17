# RabbitMq（一）

## 基本概念

### 什么是MQ？

MQ(message queue)，从字面意思上看，本质是个队列，FIFO 先入先出，只不过队列中存放的内容是 message 而已，还是一种跨进程的通信机制，用于上下游传递消息。在互联网架构中，MQ 是一种非常常见的上下游“逻辑解耦+物理解耦”的消息通信服务。使用了 MQ 之后，消息发送上游只需要依赖 MQ，不用依赖其他服务。

### 使用MQ的好处

- ##### 流量的削峰填谷

  使用队列作为缓冲，服务器在流量高峰期不容易被冲垮。

- ##### 应用解耦

  服务之间不直接依赖，以消息队列作为连接。

- ##### 异步处理

## MQ的分类

| 消息队列 | 吞吐量 |                             优点                             |                             缺点                             |
| :------: | :----: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| ActiveMQ |  W级   |                       高可用，可靠性强                       |               维护很少，很难应用在高吞吐量场景               |
|  Kafka   | 百W级  | kafka 是分布式的，一个数据多个副本，少数机器宕机，不会丢失数据，不会导致不可用 | Kafka 单机超过 64 个队列/分区，Load 会发生明显的飙高现象，队列越多，load 越高，发送消 息响应时间变长，使用短轮询方式，实时性取决于轮询间隔时间，消费失败不支持重试；支持消息顺序， 但是一台代理宕机后，就会产生消息乱序 |
| RocketMQ | 十万级 | 可用性非常高，分布式架构,消息可以做到 0 丢失,MQ 功能较为完善，还是分 布式的，扩展性好,支持 10 亿级别的消息堆积，不会因为堆积导致性能下降 | 支持的客户端语言不多，目前是 java 及 c++，其中 c++不成熟；社区活跃度一般,没有在MQ 核心中去实现 JMS 等接口,有些系统要迁移需要修改大量代码 |
| RabbitMQ |  万级  |   MQ 功能比较完备,健壮、稳定、易 用、跨平台、支持多种语言    |                      只适合于中小型公司                      |

## 核心概念

- ### 生产者

  产生数据发送消息的程序是生产者

- ### 交换机

  交换机是 RabbitMQ 非常重要的一个部件，一方面它接收来自生产者的消息，另一方面它将消息 推送到队列中。交换机必须确切知道如何处理它接收到的消息，是将这些消息推送到特定队列还是推 送到多个队列，亦或者是把消息丢弃，这个得有交换机类型决定

- ### 队列

  队列是 RabbitMQ 内部使用的一种数据结构，尽管消息流经 RabbitMQ 和应用程序，但它们只能存 储在队列中。队列仅受主机的内存和磁盘限制的约束，本质上是一个大的消息缓冲区。

- ### 消费者

  消费与接收具有相似的含义。消费者大多时候是一个等待接收消息的程序。请注意生产者，消费 者和消息中间件很多时候并不在同一机器上。同一个应用程序既可以是生产者又是可以是消费者。

## 工作模式

### HelloWorld模式

[![](https://pic.imgdb.cn/item/60e17ee65132923bf8178027)](https://pic.imgdb.cn/item/60e17ee65132923bf8178027.png)

导入jar包

```xml
<dependency>
        <groupId>com.rabbitmq</groupId>
        <artifactId>amqp-client</artifactId>
        <version>5.10.0</version>
    </dependency>

    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-simple</artifactId>
        <version>1.7.25</version>
    </dependency>
```

```java
//工具类
public class RabbitMqUtil {
    public static Channel getChannel(){
        //创建连接工厂
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("121.4.80.204");
        connectionFactory.setUsername("admin");
        connectionFactory.setPassword("admin");
        Channel channel = null;
        try {
            //创建连接
            Connection connection = connectionFactory.newConnection();
            //创建管道
            channel = connection.createChannel();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (TimeoutException e) {
            e.printStackTrace();
        }
        return channel;
    }
}
```

```java
//生产者
public class Producer {
    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMqUtil.getChannel();
        channel.queueDeclare("hello", false, false, false, null);
        channel.basicPublish("","hello",null,"hello,world!".getBytes());
    }
}
//消费者
public class Client {
    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMqUtil.getChannel();
        //收到消息回调
        DeliverCallback deliverCallback = (SconsumerTag, message)->{
            System.out.println(new String(message.getBody()));
        };
        //取消消息回调
        CancelCallback cancelCallback = (consumerTag) -> {
            System.out.println("消息接收中断");
        };
        channel.basicConsume("hello", true, deliverCallback,cancelCallback);
    }
}
```

### Work模式

[![](https://pic.imgdb.cn/item/60e17f0e5132923bf8184846)](https://pic.imgdb.cn/item/60e17f0e5132923bf8184846.png)

```java
//生产者
public class Producer {
    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMqUtil.getChannel();
        channel.queueDeclare("hello", false, false, false, null);
        channel.basicPublish("","hello",null,"hello,world!".getBytes());
    }
}
//消费者1
public class Client1 {
    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMqUtil.getChannel();
        
        //消费方设置不公平分发
        //channel.basicQos(1);
        
        //收到消息回调
        DeliverCallback deliverCallback = (SconsumerTag, message)->{
            System.out.println(new String(message.getBody()));
        };
        //取消消息回调
        CancelCallback cancelCallback = (consumerTag) -> {
            System.out.println("消息接收中断");
        };
        channel.basicConsume("hello", true, deliverCallback,cancelCallback);
    }
}
//消费者2
public class Client2{
    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMqUtil.getChannel();
        //收到消息回调
        DeliverCallback deliverCallback = (SconsumerTag, message)->{
            System.out.println(new String(message.getBody()));
        };
        //取消消息回调
        CancelCallback cancelCallback = (consumerTag) -> {
            System.out.println("消息接收中断");
        };
        channel.basicConsume("hello", true, deliverCallback,cancelCallback);
    }
}
```

### 发布订阅模式

[![](https://pic.imgdb.cn/item/60e17f145132923bf818627d)](https://pic.imgdb.cn/item/60e17f145132923bf818627d.png)

### 路由模式

[![](https://pic.imgdb.cn/item/60e17f1b5132923bf8188513)](https://pic.imgdb.cn/item/60e17f1b5132923bf8188513.png)

### 主题模式

[![](https://pic.imgdb.cn/item/60e17f1f5132923bf8189b4c)](https://pic.imgdb.cn/item/60e17f1f5132923bf8189b4c.png)

## 消息应答机制

- #### 自动应答

  消息发送后立即被认为已经传送成功，这种模式需要在高吞吐量和数据传输安全性方面做权 衡,因为这种模式如果消息在接收到之前，消费者那边出现连接或者 channel 关闭，那么消息就丢失 了,当然另一方面这种模式消费者那边可以传递过载的消息，没有对传递的消息数量进行限制，当然这样有可能使得消费者这边由于接收太多还来不及处理的消息，导致这些消息的积压，最终使 得内存耗尽，最终这些消费者线程被操作系统杀死，所以这种模式仅适用在消费者可以高效并以 某种速率能够处理这些消息的情况下使用。

- #### 手动应答

  方法：

  Channel.basicAck(用于肯定确认) RabbitMQ 已知道该消息并且成功的处理消息，可以将其丢弃了 

  Channel.basicNack(用于否定确认) 

  Channel.basicReject(用于否定确认) 与 Channel.basicNack 相比少一个参数 不处理该消息了直接拒绝，可以将其丢弃了

#### 代码实现：（只需要修改消费者）

```java
public class Client1 {
    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMqUtil.getChannel();
        //收到消息回调
        DeliverCallback deliverCallback = (SconsumerTag, message)->{
            System.out.println("开始处理消息");
            System.out.println(new String(message.getBody()));

            try {
                TimeUnit.SECONDS.sleep(6);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            //消息的序号 / 是否批量应答
            channel.basicAck(message.getEnvelope().getDeliveryTag(),false);
        };
        //取消消息回调
        CancelCallback cancelCallback = (consumerTag) -> {
            System.out.println("消息接收中断");
        };


        channel.basicConsume("hello", false, deliverCallback,cancelCallback);
    }
}
```

## 持久化

创建队列时，将队列设置为持久化

```java
channel.queueDeclare("hello", true, false, false, null);
```

## 发布确认

```java
//队列必须持久化
channel.queueDeclare("hello", true, false, false, null);
//队列中的消息必须持久化
channel.basicPublish("","hello",MessageProperties.PERSISTENT_TEXT_PLAIN,next.getBytes());
//开启发布确认
channel.confirmSelect();
```

- ### 单个发布确认

  ```java
  //每一条后
  channel.waitForConfirms();
  ```

- ### 批量发布确认

- ### 异步发布确认

  ```java
  public class Producer1 {
      public static void main(String[] args) throws IOException, TimeoutException, InterruptedException {
          Channel channel = RabbitMqUtil.getChannel();
          channel.queueDeclare("hello", true, false, false, null);
          //开启发布确认
          channel.confirmSelect();
          long start = System.currentTimeMillis();
          ConcurrentHashMap<Long, String> longStringConcurrentHashMap = new ConcurrentHashMap<>();
  
          ConfirmCallback ackCallback = (deliveryTag, multiple) -> {
              System.out.println(deliveryTag);
              longStringConcurrentHashMap.remove(deliveryTag);
          };
          ConfirmCallback nockCallback = (deliveryTag, multiple) -> {
              System.out.println("未收到消息"+deliveryTag);
          };
          channel.addConfirmListener(ackCallback,null);
          for (int i = 1;i <= 1000;i++){
              String s = String.valueOf(i);
              System.out.println("发布消息"+ s);
              TimeUnit.MILLISECONDS.sleep(10);
              longStringConcurrentHashMap.put(channel.getNextPublishSeqNo(),s);
              channel.basicPublish("","hello", null,s.getBytes());
  
          }
          long end = System.currentTimeMillis();
  
          System.out.println(end - start);
          channel.close();
  
      }
  }
  ```

  

