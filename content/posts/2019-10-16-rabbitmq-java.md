---
title: RabbitMQ示例代码
date: 2019-10-16
---

## RabbitMQ

[官网文档](https://www.rabbitmq.com/api-guide.html) 涉及jar包`amqp-client-5.7.3.jar`

1. 发送

```java
package myrabbitmq;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

public class Send {

    private final static String QUEUE_NAME = "hello";
    private final static String IP = "localhost";
    public static void main(String[] args) {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost(IP);
        factory.setUsername("guest");
        factory.setPassword("guest");
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {
            channel.queueDeclare(QUEUE_NAME, false, false, false, null);
            String message = "Hello World!";
            channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
            System.out.println(" [x] Sent '" + message + "'");
        }catch (Exception e){
            e.printStackTrace();
        }
    }
}
```

1. 接收

```java
package myrabbitmq;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.DeliverCallback;

public class Recv {

    private final static String QUEUE_NAME = "test_queue_work";
    private final static String IP = "123.59.201.22";

    public static void main(String[] args) {
        try {

            ConnectionFactory factory = new ConnectionFactory();
            factory.setHost(IP);
            factory.setUsername("guest");
            factory.setPassword("guest");
            Connection connection = factory.newConnection();
            Channel channel = connection.createChannel();

            channel.queueDeclare(QUEUE_NAME, false, false, false, null);
            System.out.println();
            System.out.println(" [*] Waiting for messages. To exit press CTRL+C");
            DeliverCallback deliverCallback = (consumerTag, delivery) -> {
                String message = new String(delivery.getBody(), "UTF-8");
                System.out.println(" [x] Received '" + message + "'");
            };
            channel.basicConsume(QUEUE_NAME, true, deliverCallback, consumerTag -> { });
        }catch (Exception e){
            e.printStackTrace();
        }
    }
}
```
