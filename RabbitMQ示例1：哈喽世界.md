# 哈喽世界
## 目录

- 介绍：相关知识点，名词解释
- 为啥要用消息队列
- 安装RabbitMQ
- 简单DEMO

## 介绍

> RabbitMQ是一个消息中间件，主要负责接收和转发消息

RabbitMQ类似邮局快递，使用者就是寄件人和收件人

### 知识点

**1. 生产者**

> 发送消息的就是生产者，生产者只管发送消息，不管谁来取

**2. 队列**

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

**3. 消费者**

> 接收消息的就是消费者，消费者只管接收消息，不管谁发的

一般情况下，生产者、消费者、RabbitMQ服务都是部署在不同地方的，生产者和消费者都不需要知道对方是谁，只要能收发消息就行

**4.  关系图**

他们三者之间的关系如下图所示，其中P是生产者，C是消费者，中间红色是队列

![image-20210317122121868](https://i.loli.net/2021/03/17/JPfcIt9iZXvWBsq.png)

*(图片来自RabbitMQ官网)*



## 为啥要用消息队列

消息队列可以用来执行异步任务，使得主要的任务可以继续执行，不会被阻塞

比如下单系统，一般下单系统包含的业务逻辑有：下单、减库存、生成订单号、通知用户等，如果不用消息队列，那么整个下单的流程就会很长，如果是小系统，问题不大；但是提升几个数量级之后，就会感觉到系统卡顿；

这时，如果有消息队列，就可以先执行主要的任务，比如扣减库存、生成订单号；然后将该订单信息发送到消息队列中；让其他线程去消息队列中读取消息，并执行对应的任务；这样前面的主要任务就不会被阻塞，从而提升系统的响应性

## 安装Rabbit MQ

我这里安装的是[Windows版本](https://www.rabbitmq.com/install-windows.html#installer)

步骤分两步: 先安装依赖的Erlang语言，再下载安装包直接双击安装

RabbitMQ安装完成后，会自动启动，后续的命令操作可以通过PowerShell客户端来控制（因为没有配置全局命令，所以需在sbin目录下执行）

相关包及细节参考：https://www.rabbitmq.com/download.html

## 简单DEMO

下面的代码很简单，就是生产者发送消息，消费者读取消息

## 代码

依赖 pom.xml

```xml
<dependency>
    <groupId>com.rabbitmq</groupId>
    <artifactId>amqp-client</artifactId>
    <version>5.11.0</version>
</dependency>
```

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
            // 发送消息：将message发送到QUEUE_NAME队列
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

## RabbitMQ管理界面

下面是一个**管理**和**监控**RabbitMQ消息的一个后台界面

用起来很方便，界面如下所示：

![image-20210319181344775](https://i.loli.net/2021/03/19/SEY5xpdsZTeKO4m.png)

> PS：默认的用户名、密码都是`guest`

**参考链接**：[RabbitMQ 管理界面](https://www.rabbitmq.com/management.html)

## 参考

- [RabbitMQ官网教程：第一节](https://www.rabbitmq.com/tutorials/tutorial-one-java.html)

- [RabbitMQ 管理界面](https://www.rabbitmq.com/management.html)

- [AMQP协议介绍](https://www.rabbitmq.com/tutorials/amqp-concepts.html)

