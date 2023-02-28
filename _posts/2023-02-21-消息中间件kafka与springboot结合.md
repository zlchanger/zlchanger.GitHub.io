---
layout: post
title: springboot + kafka
date: 2023/02/21
categories: blog
tags: [消息中间件]
description: 实用消息中间件kafka

---

## 1、kafka

* kafka相较于其他消息中间件而言，吞吐量高（10万级）、可用性高（分布式部署）

* 常被应用于大数据平台（flink、spark等对接）

## 2、springboot结合
    
### 2.1、项目依赖

    <dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
    </dependency>

### 2.2、yml配置

    kafka:
        bootstrap-servers: ${ip}:${port} #kafka地址、集群多地址','分隔
        producer:
            retries: 0 # 重试次数
            acks: 1 # 应答级别:多少个分区副本备份完成时向生产者发送ack确认(可选0、1、all/-1)
            batch-size: 5120
            # 当生产端积累的消息达到batch-size或接收到消息linger.ms后,生产者就会将消息提交给kafka
            # linger.ms为0表示每接收到一条消息就提交给kafka,这时候batch-size其实就没用了
            properties:
                linger.ms: 0 # 提交延时
            buffer-memory: 5242880 # 生产端缓冲区大小
            key-serializer: org.apache.kafka.common.serialization.StringSerializer
            value-serializer: org.apache.kafka.common.serialization.StringSerializer
            # 自定义分区器
            # spring.kafka.producer.properties.partitioner.class=com.felix.kafka.producer.CustomizePartitioner
        consumer:
            enable-auto-commit: true # 是否自动提交offset
            auto-commit-interval: 1000 # 提交offset延时(接收到消息后多久提交offset)
            # 当kafka中没有初始offset或offset超出范围时将自动重置offset
            # earliest:重置为分区中最小的offset;
            # latest:重置为分区中最新的offset(消费分区中新产生的数据);
            # none:只要有一个分区不存在已提交的offset,就抛出异常;
            auto-offset-reset: latest
            properties:
                group.id: defaultConsumerGroup # 消费组id
                session.timeout.ms: 120000 # 消费会话超时时间(超过这个时间consumer没有发送心跳,就会触发rebalance操作)
                request.timeout.ms: 180000 # 消费请求超时时间
            key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
            value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
            max-poll-records: 50 # 批量消费每次最多消费多少条消息
        listener:
            missing-topics-fatal: false # 消费端监听的topic不存在时，项目启动会报错(关掉)
            type: batch # 设置批量消费

### 2.3、config配置

    @Configuration
    public class KafkaConfig {
    
        @Bean
        public NewTopic initialTopic() {
            // 目前是单节点
            return new NewTopic("xxx_topic", 8, (short) 1);
        }
    }

### 2.4、生产者（producer）配置

#### 2.4.1 简单生产者

    @RestController
    @RequestMapping("/kafka/producer")
    public class KafkaProducer {
        @Resource
        private KafkaTemplate<String, Object> kafkaTemplate;
        // 发送消息
        @GetMapping("/{message}")
        public String sendMessage(@PathVariable("message") String message) {
            kafkaTemplate.send("xxx_topic", sendResult);
            return "ok";
        }
    }

#### 2.4.2 带回调的生产者

    @RestController
    @RequestMapping("/kafka/producer")
    public class KafkaProducer {
        @Resource
        private KafkaTemplate<String, Object> kafkaTemplate;
        // 发送消息
        @GetMapping("/callback/{message}")
        public String sendMessageWithCallback(@PathVariable("message") String message) {
            kafkaTemplate.send("xxx_topic", sendResult).addCallback(success -> {
                // 消息发送到的topic
                String topic = success.getRecordMetadata().topic();
                // 消息发送到的分区
                int partition = success.getRecordMetadata().partition();
                // 消息在分区内的offset
                long offset = success.getRecordMetadata().offset();
                System.out.println("发送消息成功:" + topic + "-" + partition + "-" + offset);
            }, failure -> {
                System.out.println("发送消息失败:" + failure.getMessage());
            });
            return "ok";
        }
    }

#### 2.4.3 带事务的生产者

    @RestController
    @RequestMapping("/kafka/producer")
    public class KafkaProducer {
        @Resource
        private KafkaTemplate<String, Object> kafkaTemplate;
        // 发送消息
        @GetMapping("/transaction/{message}")
        public String sendMessage(@PathVariable("message") String message) {
            kafkaTemplate.executeInTransaction(operations ->{
                operations.send("xxx_topic","transaction msg");
                throw new RuntimeException("fail");
            });
            return "ok";
        }
    }

### 2.5、消费者（consumer）配置

* 消费者监听可以指定监听的topic以及partition、offset，
比如我们可以监听topic1上的0分区所有消息，topic2的0分区所有消息，1分区offset=8开始的消息

* 或者当我们简单监听topic时，直接使用topics，
比如我们监听debezium_topic的所有分区，所有消息

* topics与topicPartitions不可同时使用


    @Component
    public class KafkaConsumer {
        //    @KafkaListener(id = "consumer", groupId = "xxx-group-id", topicPartitions = {
        //            @TopicPartition(topic = "topic1", partitions = {"0"}),
        //            @TopicPartition(topic = "topic2", partitions = "0",
        //                    partitionOffsets = @PartitionOffset(partition = "1", initialOffset = "8"))
        //    })
        @KafkaListener(id = "flink_batch_consumer", groupId = "flink-group", topics = "debezium_topic")
        public void onMessage(List<ConsumerRecord<?, ?>> records) {
            log.info(">>>批量消费一次，records.size()= {}", records.size());
            for (ConsumerRecord<?, ?> record : records) {
                log.info("+++++++++++++++++ r:{} +++++++++++++++++++", record);
            }
        }
    }

#### 2.5.1 消费者异常处理（ConsumerAwareListenerErrorHandler）

* 构建一个ConsumerAwareListenerErrorHandler的Bean


    @Bean
    public ConsumerAwareListenerErrorHandler errorHandler(){
        return (message,exception,consumer)->{
            log.error("消费异常：{}", message.getPayload());
            return null;
        };
    }

* @KafkaListener允许配置errorHandler来定义消费异常的处理


    @KafkaListener(topics = {"xxx_topic"},errorHandler = "errorHandler")


#### 2.5.2 消费者消息过滤（ConcurrentKafkaListenerContainerFactory）

* 定义一个 ConcurrentKafkaListenerContainerFactory 实现过滤规则，setRecordFilterStrategy中true为过滤 (ps: ConcurrentKafkaListenerContainerFactory不仅仅可用于消息过滤配置，比如setAutoStartup设置自启动等。。。)


    @Resource
    private ConsumerFactory consumerFactory;
    @Bean
    public ConcurrentKafkaListenerContainerFactory filterContainerFactory(){
        ConcurrentKafkaListenerContainerFactory factory=new ConcurrentKafkaListenerContainerFactory();
        factory.setConsumerFactory(consumerFactory);
        //被过滤器的消息将被丢弃
        factory.setAckDiscarded(true);
        //消息过滤策略
        factory.setRecordFilterStrategy(consumerRecord -> {
            ...
            // 返回true则被过滤
        });
        return factory;
    }
    
* @KafkaListener允许配置containerFactory来定义消息过滤

    
    @KafkaListener(topics = {"xxx_topic"},containerFactory = "filterContainerFactory")

### 2.6topic分区配置

kafka为topic提供了分区功能，一个topic可以划分多个分区，生产者发送消息时要将消息追加到topic的哪个分区由配置的分区策略提供。

#### 2.6.1 kafka默认分区策略

* 消息指定了分区时，则将消息追加到topic的指定分区

* 未指定分区且消息存在key值时，根据消息的key值进行hash,消息追加相同hash的分区

* 当未指定分区，且未配置消息的key值,由kafka 轮询分配一个分区

#### 2.6.2 kafka自定义分区策略

用户可以继承实现Partitioner接口，并在yml配置中将spring.kafka.producer.properties.partitioner.class指定为自定义分区类文件


### 2.7消息的转发

当使用场景为消费TopicA消息，再转发到TopicB，由监听TopicB的消费者消费，此时我们需要在消费的consumer上添加@SendTo("TopicB")注解





    


