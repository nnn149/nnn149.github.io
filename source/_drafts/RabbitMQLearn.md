---
title: RabbitMQLearn
comments: true
tags:
  - Default
categories:
  - Default
date: 2021-08-11 17:13:16
updated:
---

[RabbitMQ](https://www.rabbitmq.com/#getstarted)

<!--more-->

## 安装

### docker

```bash
docker pull rabbitmq:management
# management带网页管理，15672端口。默认用户名密码guest。mq使用hostname作为database dir
docker run -d --hostname rabbitmq --name rabbitmq -p 15672:15672 -p 5672:5672 rabbitmq:management
#以后应该在rabbitmq.conf配置，docker的环境变量配置即将被废弃
```

## 概念

几个概念说明：

Broker：简单来说就是消息队列服务器实体。
　　Exchange：消息交换机，它指定消息按什么规则，路由到哪个队列。
　　Queue：消息队列载体，每个消息都会被投入到一个或多个队列。
　　Binding：绑定，它的作用就是把exchange和queue按照路由规则绑定起来。
　　Routing Key：路由关键字，exchange根据这个关键字进行消息投递。
　　vhost：虚拟主机，一个broker里可以开设多个vhost，用作不同用户的权限分离。
　　producer：消息生产者，就是投递消息的程序。
　　consumer：消息消费者，就是接受消息的程序。
　　channel：消息通道，在客户端的每个连接里，可建立多个channel，每个channel代表一个会话任务。

消息队列的使用过程大概如下：

　　（1）客户端连接到消息队列服务器，打开一个channel。
　　（2）客户端声明一个exchange，并设置相关属性。
　　（3）客户端声明一个queue，并设置相关属性。
　　（4）客户端使用routing key，在exchange和queue之间建立好绑定关系。
　　（5）客户端投递消息到exchange。

exchange接收到消息后，就根据消息的key和已经设置的binding，进行消息路由，将消息投递到一个或多个队列里。

exchange也有几个类型，完全根据key进行投递的叫做Direct交换机，例如，绑定时设置了routing key为”abc”，那么客户端提交的消息，只有设置了key为”abc”的才会投递到队列。对key进行模式匹配后进行投递的叫做Topic交换机，符号”#”匹配一个或多个词，符号”*”匹配正好一个词。例如”abc.#”匹配”abc.def.ghi”，”abc.*”只匹配”abc.def”。还有一种不需要key的，叫做Fanout交换机，它采取广播模式，一个消息进来时，投递到与该交换机绑定的所有队列。

RabbitMQ支持消息的持久化，也就是数据写在磁盘上，为了数据安全考虑，我想大多数用户都会选择持久化。消息队列持久化包括3个部分：
　　（1）exchange持久化，在声明时指定durable => 1
　　（2）queue持久化，在声明时指定durable => 1
　　（3）消息持久化，在投递时指定delivery_mode => 2（1是非持久化）

如果exchange和queue都是持久化的，那么它们之间的binding也是持久化的。如果exchange和queue两者之间有一个持久化，一个非持久化，就不允许建立绑定。

## 使用

### [Work queues](https://www.rabbitmq.com/tutorials/tutorial-two-python.html)

```java
//发送方（生产者）
public class NewTask {
    private final static String QUEUE_NAME = "task_queue";

    public static void main(String[] args) {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("192.168.2.211");
        try (Connection connection = factory.newConnection()) {
            Channel channel = connection.createChannel();
            //配置queue持久化配置
            boolean durable = true;
            channel.queueDeclare(QUEUE_NAME, durable, false, false, null);
            //从命令行读取要发送的信息
            String message = String.join(" ", args);
            //设置队列持久化后message还需要设置MessageProperties.TEXT_PLAIN，这样mq遇到异常关闭，message才有保存到磁盘
            channel.basicPublish("", QUEUE_NAME, MessageProperties.TEXT_PLAIN, message.getBytes());
            System.out.println("[x] Sent '" + message + "'");
        } catch (IOException | TimeoutException e) {
            e.printStackTrace();
        }
    }
}
```

```java
//接收方（消费者）
public class Worker {
    private final static String QUEUE_NAME = "task_queue";

    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("192.168.2.211");
        final Connection connection = factory.newConnection();
        final Channel channel = connection.createChannel();
        boolean durable = true;
        channel.queueDeclare(QUEUE_NAME, durable, false, false, null);
        System.out.println(" [*] Waiting for messages. To exit press CTRL+C");
        //设置预匹配计数为1，在worker处理并确认上一条消息前，mq不会向它发送新消息
        int prefetchCount = 1;
        channel.basicQos(prefetchCount);
        //消息传递的回调
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");
            System.out.println("[x] Received'" + message + "'");
            try {
                //模拟耗时操作
                doWork(message);
            }finally {
                System.out.println("[x] Done");
                //最后发送消息确认给mq
                channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
            }
        };
        //消息确认 消费者会返回确认信息给mq。若消费者在没有发送ack的情况下死亡或退出等，mq将会把消息重新排队。
        boolean autoAck = false;
        channel.basicConsume(QUEUE_NAME, autoAck, deliverCallback, consumerTag -> {
        });
    }
}
```

配置了queue和message的持久化，mq装在docker里面，并且run容器的时候指定了`--hostname rabbitmq`，生产者发送一条message，mq的网页端可以看到一条Ready的message，然后关闭mq的容器再启动，发现消息丢失。。

