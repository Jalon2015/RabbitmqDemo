# 哈喽世界
## 介绍
> RabbitMQ是一个消息中间件，主要负责接收和转发消息

RabbitMQ类似邮局快递，使用者就是寄件人和收件人

## 知识点

### 生产者

发送消息的就是生产者

### 队列

存储消息的地方

### 消费者

接收消息的就是消费者

> PS：一般情况下，生产者、消费者、RabbitMQ服务都是部署在不同地方的

### 关系图

他们三者之间的关系如下图所示，其中P是生产者，C是消费者，中间红色是队列

![image-20210317122121868](https://i.loli.net/2021/03/17/JPfcIt9iZXvWBsq.png)

*(图片来自RabbitMQ官网)*

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
		// 声明队列：这里先声明队列名,其他的参数分别是是否持久化、是否独占排他性、是否自动删除、附加参数，详细介绍会在后面的章节写	
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