# RabbitMQ（二）

## 死信队列

[![](https://pic.imgdb.cn/item/60e2c4c65132923bf85b5507.jpg)](https://pic.imgdb.cn/item/60e2c4c65132923bf85b5507.jpg)

代码实现：

```java
public class Producer {
    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMqUtil.getChannel();

        channel.exchangeDeclare("direct", BuiltinExchangeType.DIRECT);
        channel.exchangeDeclare("dead", BuiltinExchangeType.DIRECT);

        HashMap<String, Object> stringObjectHashMap = new HashMap<>();
        stringObjectHashMap.put("x-dead-letter-exchange","dead");
        stringObjectHashMap.put("x-dead-letter-routing-key","dead.queue1");
        stringObjectHashMap.put("x-message-ttl",1000000);
        stringObjectHashMap.put("x-max-length",3);

        channel.queueDeclare("direct.queue1",true,false,false,stringObjectHashMap);
        channel.queueDeclare("direct.queue2",true,false,false,stringObjectHashMap);
        channel.queueDeclare("dead.queue1",true,false,false,null);

        channel.queueBind("direct.queue1","direct","direct.queue1");
        channel.queueBind("direct.queue2","direct","direct.queue2");
        channel.queueBind("dead.queue1","dead","dead.queue1");
        Scanner scanner = new Scanner(System.in);
        while (true){
            String next = scanner.next();
            if (next.equals("q!")){
                break;
            }else {
                channel.basicPublish("direct","direct.queue1", MessageProperties.PERSISTENT_TEXT_PLAIN,next.getBytes());
            }
        }
        channel.close();

    }
}


public class Client1 {
    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMqUtil.getChannel();

        DeliverCallback deliverCallback = (SconsumerTag, message)->{
            System.out.println("开始处理消息");
            System.out.println(new String(message.getBody()));

            try {
                TimeUnit.SECONDS.sleep(2);
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


        channel.basicConsume("direct.queue1", false, deliverCallback,cancelCallback);
    }
}


public class DeadClient1 {
    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMqUtil.getChannel();

        DeliverCallback deliverCallback = (SconsumerTag, message)->{
            System.out.println("开始处理消息");
            System.out.println(new String(message.getBody()));

            try {
                TimeUnit.SECONDS.sleep(2);
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


        channel.basicConsume("dead.queue1", false, deliverCallback,cancelCallback);
    }
}

```

## 延迟队列

[![](https://pic.imgdb.cn/item/60e2c5445132923bf85dee81.jpg)](https://pic.imgdb.cn/item/60e2c5445132923bf85dee81.jpg)

代码略

## 整合SpringBoot

