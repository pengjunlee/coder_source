---
title: SpringBoot框架整合之--KafkaTemplate
date: 2020-07-24 14:15:00
updated: 2020-07-24 14:15:00
tags: SpringBoot框架
categories: SpringBoot框架
keywords: Java, SpringBoot
type: 
description: SpringBoot中如何整合KafkaTemplate?
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img15.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img15.jpg
aside: true
toc: true
toc_number: true
auto_open: true
copyright: false
mathjax: false
katex: false
aplayer:
highlight_shrink: false
top: false
---
本文将对如何在Springboot项目中整合KafkaTemplate进行简单示例和介绍，项目的完整目录层次如下图所示。

<div align=center>

![整合KafkaTemplate示意图](http://pengjunlee.3vzhuji.net/static/springboot/04.png "整合KafkaTemplate示意图")
<div align=left>

# 添加依赖与配置

首先，需要在工程POM文件中引入相关依赖。
```Xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.2.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--引入 Junit 依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!--引入 Kafka 依赖-->
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>
```
然后，在核心配置文件 `application.yml` 中添加Kafka的生产者和消费者配置。
```Yml
	spring:
	  kafka:
	    # 指定 Kafka Borker 列表
	    bootstrap-servers: 172.16.250.233:9092,172.16.250.234:9092,172.16.250.237:9092
	    # 指定缺省 Topic
	    template:
	      default-topic: topic1
	 
	    ### Listener 配置参数 ###
	    listener:
	      # Listener 的AckMode,当enable.auto.commit的值设置为false时，该值会生效；为true时不会生效
	      ack-mode: COUNT_TIME
	      # 指定 Listener 容器中的线程数，用于提高并发量
	      concurrency: 5
	      # 轮询消费者时的超时时长（以毫秒为单位）
	      poll-timeout : 3000
	      # 当 ackMode为“COUNT”或“COUNT_TIME”时，偏移提交之间的记录数
	      ack-count: 2000
	      # 当ackMode为“TIME”或“COUNT_TIME”时，偏移提交之间的时间（以毫秒为单位）
	      ack-time: 60000
	 
	    ### Producer 配置参数 ###
	    producer:
	      # ClientID用于区分不同的生产者，该ID会在发出请求时传递给服务器，用于服务器端日志记录
	      client-id: producer1
	      # 若消息发送失败，重试发送多少次
	      retries: 3
	      # 每当多个记录被发送到同一分区时，生产者将尝试将记录一起批量处理为更少的请求；
	      # 这有助于提升客户端和服务器上的性能，此配置控制默认批量大小（以字节为单位），默认值为16384
	      batch-size: 16384
	      # 以逗号分隔的（主机：端口）列表，用于建立与Kafka集群的初始连接
	      bootstrap-servers: 172.16.250.233:9092,172.16.250.234:9092,172.16.250.237:9092
	      # procedure要求leader在考虑完成请求之前收到的确认数，用于控制发送记录在服务端的持久化，其值可以为如下：
	      # acks = 0 如果设置为零，则生产者将不会等待来自服务器的任何确认，该记录将立即添加到套接字缓冲区并视为已发送。在这种情况下，无法保证服务器已收到记录，并且重试配置将不会生效（因为客户端通常不会知道任何故障），为每条记录返回的偏移量始终设置为-1。
	      # acks = 1 这意味着leader会将记录写入其本地日志，但无需等待所有副本服务器的完全确认即可做出回应，在这种情况下，如果leader在确认记录后立即失败，但在将数据复制到所有的副本服务器之前，则记录将会丢失。
	      # acks = all 这意味着leader将等待完整的同步副本集以确认记录，这保证了只要至少一个同步副本服务器仍然存活，记录就不会丢失，这是最强有力的保证，这相当于acks = -1的设置。
	      # 可以设置的值为：all, -1, 0, 1
	      acks: 1
	      # 生产者可用于缓冲等待发送到服务器的记录的内存总字节数，默认值为33554432
	      buffer-memory: 33554432
	      # 生产者生成的所有数据的压缩类型，此配置接受标准压缩编解码器（'gzip'，'snappy'，'lz4'），
	      # 默认值为producer
	      compression-type: none
	      # Key的反序列化器类，实现类实现了接口org.apache.kafka.common.serialization.Deserializer
	      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
	      # 值的反序列化器类，实现类实现了接口org.apache.kafka.common.serialization.Deserializer
	      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
	 
	 
	    ### Consumer配置参数 ###
	    consumer:
	      # ClientID用于区分不同的消费者，该ID会在发出请求时传递给服务器，用于服务器端日志记录
	      client-id: client1
	      # 用于标识此消费者所属的组。
	      group-id: group1
	      # 一次调用poll()操作可返回的最大记录数，默认值为500
	      max-poll-records: 200
	      # 如果为true，则消费者消费消息的偏移量将在后台定期提交，默认值为true
	      enable-auto-commit: true
	      # 如果enable-auto-commit为true，消费者将消费消息的偏移量自动提交给Kafka的频率（以毫秒为单位），默认值为5000
	      auto-commit-interval: 5000
	      # 当Kafka中没有初始偏移量或者服务器上不存在当前偏移量时如何处理，默认值为latest
	      # 可选的值为latest, earliest, none
	      auto-offset-reset: earliest # 最早未被消费的offset
	      # 以逗号分隔的（主机：端口）列表，用于建立与Kafka集群的初始连接
	      bootstrap-servers: 172.16.250.233:9092,172.16.250.234:9092,172.16.250.237:9092
	      # 一次请求中服务器返回的最小数据量（以字节为单位），默认值为1，对应的kafka的参数为fetch.min.bytes
	      fetch-min-size:
	      # 如果队列中数据量少于 fetch-min-size ，服务器将阻塞的最长时间（以毫秒为单位），默认值为500
	      fetch-max-wait: 500
	      # 心跳与消费者协调员之间的预期时间（以毫秒为单位），默认值为3000
	      heartbeat-interval: 3000
	      # Key的反序列化器类，实现类实现了接口org.apache.kafka.common.serialization.Deserializer
	      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
	      # 值的反序列化器类，实现类实现了接口org.apache.kafka.common.serialization.Deserializer
	      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
```

# KafkaProducer（生产者）

在KafkaProducer（生产者）中使用到了KafkaTemplate来向Kafka集群发送消息。
```Java
	package com.pengjunlee.kafka;
	 
	import lombok.extern.java.Log;
	import org.apache.kafka.clients.producer.ProducerRecord;
	import org.apache.kafka.clients.producer.RecordMetadata;
	import org.springframework.kafka.core.KafkaTemplate;
	import org.springframework.kafka.support.SendResult;
	import org.springframework.stereotype.Component;
	import org.springframework.util.concurrent.ListenableFuture;
	 
	import javax.annotation.Resource;
	import java.util.concurrent.ExecutionException;
	import java.util.logging.Level;
	 
	/**
	 * @author pengjunlee
	 * @create 2020-01-06 17:11
	 */
	@Log(topic = "kafkaProducer")
	@Component
	public class KafkaProducer {
	 
	    @Resource
	    private KafkaTemplate<String, String> kafkaTemplate;
	 
	    // 异步发送消息
	    public RecordMetadata sendMessage(String topic, String message) {
	 
	        ListenableFuture<SendResult<String, String>> future = kafkaTemplate.send(topic, message);
	        return futureHandle(future);
	 
	    }
	 
	    // 异步发送消息
	    public RecordMetadata sendMsg(String topic, String msg) {
	 
	        //指定topic,key,value
	        ProducerRecord<String, String> record = new ProducerRecord<>(topic, msg, msg);
	        ListenableFuture<SendResult<String, String>> future = kafkaTemplate.send(record);
	        return futureHandle(future);
	    }
	 
	    // 同步发送消息
	    public RecordMetadata syncSendMsg(String topic, String msg) {
	        RecordMetadata recordMetadata = null;
	        try {
	            SendResult<String, String> sendResult = kafkaTemplate.send(topic, msg).get();
	            recordMetadata = sendResult.getRecordMetadata();
	        } catch (InterruptedException e) {
	            log.log(Level.SEVERE, "发送失败!");
	            log.severe(e.getMessage());
	        } catch (ExecutionException e) {
	            log.log(Level.SEVERE, "发送失败!");
	            log.severe(e.getMessage());
	        }
	        return recordMetadata;
	    }
	 
	    private RecordMetadata futureHandle(ListenableFuture<SendResult<String, String>> future) {
	        RecordMetadata recordMetadata = null;
	        try {
	            recordMetadata = future.get().getRecordMetadata();
	            log.info("发送成功!");
	            log.info("topic：" + recordMetadata.topic());
	            log.info("detail：" + recordMetadata.toString());
	        } catch (InterruptedException | ExecutionException e) {
	            log.log(Level.SEVERE, "发送失败!");
	            log.severe(e.getMessage());
	        }
	        return recordMetadata;
	    }
	}
```

# KafkaConsumer（消费者）
```Java
	package com.pengjunlee.kafka;
	 
	import lombok.extern.java.Log;
	import org.apache.kafka.clients.consumer.ConsumerRecord;
	import org.springframework.kafka.annotation.KafkaListener;
	import org.springframework.stereotype.Component;
	 
	import java.util.List;
	 
	/**
	 * @author pengjunlee
	 * @create 2020-01-07 14:38
	 */
	@Log(topic = "kafkaConsumer")
	@Component
	public class KafkaConsumer {
	 
	 
	    /**
	     * 有消息就读取,只读取消息value
	     */
	    @KafkaListener(topics = {"topic1"})
	    public void readMsg(String msg) {
	        log.info(msg);
	    }
	 
	    /**
	     * 有消息就读取,批量读取消息value
	     */
	    @KafkaListener(topics = "topic2")
	    public void receiveMsg(List<String> msgs) {
	        for (String msg : msgs) {
	            log.info(msg);
	        }
	    }
	 
	    /**
	     * 有消息就读取,读取消息topic，offset，key，value等信息
	     */
	    @KafkaListener(topics = "topic3")
	    public void listenMsg(ConsumerRecord<?, ?> msgs) {
	        System.out.println("收到消息,topic:" + msgs.topic() + "  offset:" + msgs.offset() + "  key:" + msgs.key() + "  value:" + msgs.value());
	    }
	}
```

# KafkaController

创建一个Controller用来模拟前端请求向Kafka发送消息。 
```Java
	package com.pengjunlee.controller;
	 
	import com.pengjunlee.kafka.KafkaProducer;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.web.bind.annotation.PathVariable;
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RequestParam;
	import org.springframework.web.bind.annotation.RestController;
	 
	/**
	 * @author pengjunlee
	 * @create 2020-01-07 14:41
	 */
	@RestController
	@RequestMapping("/msg")
	public class KafkaController {
	 
	    @Autowired
	    private KafkaProducer kafkaProducer;
	 
	 
	    // 模拟异步发送消息
	    @RequestMapping("/async/create/{id}")
	    public void createAsyncMsg(@PathVariable(name = "id") String topicId, @RequestParam(name = "msg", defaultValue = "") String msg) {
	        kafkaProducer.sendMsg("topic" + topicId, msg);
	    }
	 
	 
	    // 模拟同步发送消息，直接将发送结果返回给前端
	    @RequestMapping("/sync/create/{id}")
	    public String createSyncMsg(@PathVariable(name = "id") String topicId, @RequestParam(name = "msg", defaultValue = "") String msg) {
	        return kafkaProducer.syncSendMsg("topic" + topicId, msg).toString();
	 
	    }
	}
```

# 启动应用
```Java
	package com.pengjunlee;
	 
	import org.springframework.boot.SpringApplication;
	import org.springframework.boot.autoconfigure.SpringBootApplication;
	 
	@SpringBootApplication
	public class KafkaApplication {
	 
	    public static void main(String[] args) {
	 
	        SpringApplication.run(KafkaApplication.class, args);
	 
	    }
	 
	}
```

# 场景测试

## 发送异步消息
在浏览器中访问 `http://localhost:8080/msg/async/create/2?msg="hello kafka"` ，该请求会向 topic2 中异步发送一条内容为 “hello kafka” 的消息。

同时在程序的控制台打印如下内容：

<div align=center>

![整合KafkaTemplate示意图](http://pengjunlee.3vzhuji.net/static/springboot/05.png "整合KafkaTemplate示意图")
<div align=left>

## 发送同步消息
在浏览器中访问 `http://localhost:8080/msg/sync/create/3?msg="hello world"` ，该请求会向 topic3 中同步添加一条内容为 “hello world” 的消息，并将消息发送的结果输出到浏览器窗口。

<div align=center>

![整合KafkaTemplate示意图](http://pengjunlee.3vzhuji.net/static/springboot/06.png "整合KafkaTemplate示意图")
<div align=left>