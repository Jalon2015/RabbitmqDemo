# RabbitMQ示例
- [RabbitMQ示例1：哈喽世界](https://github.com/Jalon2015/RabbitmqDemo/blob/master/RabbitMQ%E7%A4%BA%E4%BE%8B1%EF%BC%9A%E5%93%88%E5%96%BD%E4%B8%96%E7%95%8C.md)
- RabbitMQ示例2：工作队列
- RabbitMQ示例3：发布与订阅【fanout交换机】
- RabbitMQ示例4：路由【direct交换机】
- RabbitMQ示例5：主题【topic交换机】
- RabbitMQ示例6：远程过程调用RPC
# Pom.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.jalon</groupId>
    <artifactId>rabbit-demo</artifactId>
    <version>1.0.0</version>

    <properties>
        <java.version>1.8</java.version>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>
    <dependencies>
        <dependency>
            <groupId>com.rabbitmq</groupId>
            <artifactId>amqp-client</artifactId>
            <version>5.11.0</version>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>1.2.3</version>
        </dependency>
    </dependencies>
</project>
```
# 参考
[官网教程](https://www.rabbitmq.com/getstarted.html)