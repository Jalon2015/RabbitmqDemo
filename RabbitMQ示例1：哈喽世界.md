# 哈喽世界
## 介绍
> RabbitMQ是一个消息中间件，主要负责接收和转发消息

RabbitMQ类似邮局快递，使用者就是寄件人和收件人

## 知识点

### 生产者

> 发送消息的就是生产者

### 队列

> 存储消息的地方

声明队列的代码如下所示

```java
queueDeclare(String queue, boolean durable, boolean exclusive, boolean autoDelete,
                             Map<String, Object> arguments)
```

参数解释：（这里只是做概念上的简单解释，具体案例可以通过修改代码来实现）

- queue：队列名
- durable：是否持久化，指的是消息服务重启时，是否恢复队列
- exlusive：是否具有排他性，指的是队列是否只能被一个连接所使用
- autoDelete：是否自动删除，指的是当最后一个订阅队列的消费者退出时，是否删除队列（注：如果队列中消息为空，是不会删除队列的）

### 消费者

> 接收消息的就是消费者

一般情况下，生产者、消费者、RabbitMQ服务都是部署在不同地方的

### 关系图

他们三者之间的关系如下图所示，其中P是生产者，C是消费者，中间红色是队列

![image-20210317122121868](https://i.loli.net/2021/03/17/JPfcIt9iZXvWBsq.png)

*(图片来自RabbitMQ官网)*

### RabbitMQ管理界面

这是一个**管理**和**监控**RabbitMQ消息的一个后台界面

用起来很方便，界面如下所示：

![image-20210319181344775](https://i.loli.net/2021/03/19/SEY5xpdsZTeKO4m.png)

> PS：默认的用户名、密码都是`guest`

**参考连接**：[RabbitMQ 管理界面](https://www.rabbitmq.com/management.html)

## 代码

消费者`Recv.java`

```java

public class Recv {

    private final static String QUEUE_NAME = "hello";

    public static void main(String[] args) throws IOException, TimeoutException {
        // 创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        // 设置RabbitMQ服务器地址（默认也是localhsot)
        factory.setHost("localhost");
        // 创建连接
        Connection connection = factory.newConnection();
        // 创建频道
        Channel channel = connection.createChannel();
		// 声明队列：参数依次是队列名、是否持久化、是否独占排他性、是否自动删除、附加参数
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        System.out.println("waiting for  messages, To exit press CTRL+C");
        // 消息回调函数：消息的接收和处理
        DeliverCallback callback = (s, delivery) -> {
            String message = new String(delivery.getBody(), "utf-8");
            System.out.println("received:" + message);
        };
        // 消费消息，即接收消息
        channel.basicConsume(QUEUE_NAME, true, callback, consumerTag -> {
        });
    }

}

```

生产者`Send.java`，代码基本和消费者一致，不同的是生产者发送消息

```java

public class Send {

    private final static String QUEUE_NAME = "hello";

    public static void main(String[] args){
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        try{
            Connection connection = factory.newConnection();
            Channel channel = connection.createChannel();
            channel.queueDeclare(QUEUE_NAME, false, false, false, null);
            String message = "Hello World";
            // 发送消息：参数后面详细介绍，这里要知道的是消息会被发送到队列
            channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
            System.out.println("send:" + message);
        }catch (Exception e){
            e.printStackTrace();
        }
    }

}

```

## 演示效果

- 先启动消费者`Recv.java`，然后启动生产者`Send.java`

- 可以看到控制台输出如下所示：生产者发送了`Hello World`，生产者接收到`Hello World`

  <img src="https://i.loli.net/2021/03/17/b234vhalo81XrGC.png" alt="image-20210317141347742"  />

  <img src="https://i.loli.net/2021/03/17/y531EsfrPAkjpiD.png" alt="image-20210317141259615"  />



## 参考

- [RabbitMQ官网教程：第一节](https://www.rabbitmq.com/tutorials/tutorial-one-java.html)

- [RabbitMQ 管理界面](https://www.rabbitmq.com/management.html)

- [AMQP协议介绍](https://www.rabbitmq.com/tutorials/amqp-concepts.html)