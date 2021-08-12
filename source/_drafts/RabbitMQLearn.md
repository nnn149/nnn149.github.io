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
