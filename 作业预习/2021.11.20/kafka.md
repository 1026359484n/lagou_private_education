# **Kafka**

1. 集群模式
2. 零拷贝、批量机制
3. 一些优缺点及应用场景

## 1.  **Kafka**架构与实战

### **1.1** 概念和基本架构

#### **1.1.1 Kafka**介绍

Kafka是最初由Linkedin公司开发，是⼀个分布式、分区的、多副本的、多⽣产者、多订阅者，基于zookeeper协调的分布式⽇志系统（也可以当做MQ系统），常⻅可以⽤于web/nginx⽇志、访问⽇志，消息服务等等，Linkedin于2010年贡献给了Apache基⾦会并成为顶级开源项⽬。主要应⽤场景是：⽇志收集系统和消息系统。

Kafka主要设计⽬标如下：

- 以时间复杂度为O(1)的⽅式提供消息持久化能⼒，即使对TB级以上数据也能保证常数时间的访问性能。

- ⾼吞吐率。即使在⾮常廉价的商⽤机器上也能做到单机⽀持每秒100K条消息的传输。

- ⽀持Kafka Server间的消息分区，及分布式消费，同时保证每个partition内的消息顺序传输。

- 同时⽀持离线数据处理和实时数据处理。

- ⽀持在线⽔平扩展

  ![image-20211117005220330](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.11.20/image-20211117005220330-7081544.png)

有两种主要的消息传递模式：点对点传递模式、发布**-**订阅模式。⼤部分的消息系统选⽤发布-订阅模式。**Kafka**就是⼀种发布**-**订阅模式。

对于消息中间件，消息分推拉两种模式。Kafka只有消息的拉取，没有推送，可以通过轮询实现消息的推送。

1. Kafka在⼀个或多个可以跨越多个数据中⼼的服务器上作为集群运⾏。
2. Kafka集群中按照主题分类管理，⼀个主题可以有多个分区，⼀个分区可以有多个副本分区。
3. 每个记录由⼀个键，⼀个值和⼀个时间戳组成。

Kafka具有四个核⼼API：

1. Producer API：允许应⽤程序将记录流发布到⼀个或多个Kafka主题。
2. Consumer API：允许应⽤程序订阅⼀个或多个主题并处理为其⽣成的记录流。
3. Streams API：允许应⽤程序充当流处理器，使⽤⼀个或多个主题的输⼊流，并⽣成⼀个或多个输出主题的输出流，从⽽有效地将输⼊流转换为输出流。
4. Connector API：允许构建和运⾏将Kafka主题连接到现有应⽤程序或数据系统的可重⽤⽣产者或使⽤者。例如，关系数据库的连接器可能会捕获对表的所有更改。

#### **1.1.2 Kafka**优势

1. ⾼吞吐量：单机每秒处理⼏⼗上百万的消息量。即使存储了许多TB的消息，它也保持稳定的性能。
1. ⾼性能：单节点⽀持上千个客户端，并保证零停机和零数据丢失。
3. 持久化数据存储：将消息持久化到磁盘。通过将数据持久化到硬盘以及replication防⽌数据丢失。
   1. 零拷贝： https://cloud.tencent.com/developer/article/1421266
   2.  顺序读，顺序写
   3. 利⽤Linux的⻚缓存

4. 分布式系统，易于向外扩展。所有的Producer、Broker和Consumer都会有多个，均为分布式的。⽆需停机即可扩展机器。多个Producer、Consumer可能是不同的应⽤。
5. 可靠性 - Kafka是分布式，分区，复制和容错的。
6. 客户端状态维护：消息被处理的状态是在Consumer端维护，⽽不是由server端维护。当失败时能⾃动平衡。
7. ⽀持online和offline的场景。
8. ⽀持多种客户端语⾔。Kafka⽀持Java、.NET、PHP、Python等多种语⾔。

#### **1.1.3 Kafka**应⽤场景

⽇志收集：⼀个公司可以⽤Kafka可以收集各种服务的Log，通过Kafka以统⼀接⼝服务的⽅式开放给各种Consumer；

消息系统：解耦⽣产者和消费者、缓存消息等；

⽤户活动跟踪：Kafka经常被⽤来记录Web⽤户或者App⽤户的各种活动，如浏览⽹⻚、搜索、点击等活动，这些活动信息被各个服务器发布到Kafka的Topic中，然后消费者通过订阅这些Topic来做实时的监控分析，亦可保存到数据库；

运营指标：Kafka也经常⽤来记录运营监控数据。包括收集各种分布式应⽤的数据，⽣产各种操作的集中反馈，⽐如报警和报告；

流式处理：⽐如Spark Streaming和Storm。

#### **1.1.4** 基本架构

**消息和批次**

Kafka的数据单元称为消息。可以把消息看成是数据库⾥的⼀个“数据⾏”或⼀条“记录”。消息由字节数组组成。

消息有键，键也是⼀个字节数组。当消息以⼀种可控的⽅式写⼊不同的分区时，会⽤到键。

为了提⾼效率，消息被分批写⼊Kafka。批次就是⼀组消息，这些消息属于同⼀个主题和分区。

把消息分成批次可以减少⽹络开销。批次越⼤，单位时间内处理的消息就越多，单个消息的传输时间就越长。批次数据会被压缩，这样可以提升数据的传输和存储能⼒，但是需要更多的计算处理。

**模式**

消息模式（schema）有许多可⽤的选项，以便于理解。如JSON和XML，但是它们缺乏强类型处理能⼒。Kafka的许多开发者喜欢使⽤Apache Avro。Avro提供了⼀种紧凑的序列化格式，模式和消息体分开。当模式发⽣变化时，不需要重新⽣成代码，它还⽀持强类型和模式进化，其版本既向前兼容，也向后兼容。

数据格式的⼀致性对Kafka很重要，因为它消除了消息读写操作之间的耦合性。

**主题和分区**

Kafka的消息通过主题进⾏分类。主题可⽐是数据库的表或者⽂件系统⾥的⽂件夹。主题可以被分为若⼲分区，⼀个主题通过分区分布于Kafka集群中，提供了横向扩展的能⼒。

![image-20211117005806197](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.11.20/image-20211117005806197-7081888.png)

⽣产者和消费者

⽣产者创建消息。消费者消费消息。

⼀个消息被发布到⼀个特定的主题上。

⽣产者在默认情况下把消息均衡地分布到主题的所有分区上：

1. 直接指定消息的分区
2. 根据消息的key散列取模得出分区
3. 轮询指定分区。

消费者通过偏移量来区分已经读过的消息，从⽽消费消息。

消费者是消费组的⼀部分。消费组保证每个分区只能被⼀个消费者使⽤，避免重复消费。

![image-20211117005854423](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.11.20/image-20211117005854423-7081936.png)

**broker**和集群⼀个独⽴的Kafka服务器称为broker。broker接收来⾃⽣产者的消息，为消息设置偏移量，并提交消息到磁盘保存。broker为消费者提供服务，对读取分区的请求做出响应，返回已经提交到磁盘上的消息。单个**broker**可以轻松处理数千个分区以及每秒百万级的消息量。

![image-20211117005930197](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.11.20/image-20211117005930197-7081972.png)

每个集群都有⼀个broker是集群控制器（⾃动从集群的活跃成员中选举出来）。

控制器负责管理⼯作：

- 将分区分配给broker
- 监控broker

集群中⼀个分区属于⼀个**broker**，该broker称为分区⾸领。

⼀个分区可以分配给多个**broker**，此时会发⽣分区复制。

分区的复制提供了消息冗余，⾼可⽤。副本分区不负责处理消息的读写。

#### **1.1.5** 核⼼概念

##### **1.1.5.1 Producer**

⽣产者创建消息。

该⻆⾊将消息发布到Kafka的topic中。broker接收到⽣产者发送的消息后，broker将该消息追加到当前⽤于追加数据的 segment ⽂件中。

⼀般情况下，⼀个消息会被发布到⼀个特定的主题上。

1. 默认情况下通过轮询把消息均衡地分布到主题的所有分区上。
2. 在某些情况下，⽣产者会把消息直接写到指定的分区。这通常是通过消息键和分区器来实现的，分区器为键⽣成⼀个散列值，并将其映射到指定的分区上。这样可以保证包含同⼀个键的消息会被写到同⼀个分区上。
3. ⽣产者也可以使⽤⾃定义的分区器，根据不同的业务规则将消息映射到分区。

##### **1.1.5.2 Consumer**

消费者读取消息。

1. 消费者订阅⼀个或多个主题，并按照消息⽣成的顺序读取它们。

2. 消费者通过检查消息的偏移量来区分已经读取过的消息。偏移量是另⼀种元数据，它是⼀个不断递增的整数值，在创建消息时，Kafka 会把它添加到消息⾥。在给定的分区⾥，每个消息的偏移量都是唯⼀的。消费者把每个分区最后读取的消息偏移量保存在Zookeeper 或**Kafka** 上，如果消费者关闭或重启，它的读取状态不会丢失。

3. 消费者是消费组的⼀部分。群组保证每个分区只能被⼀个消费者使⽤。

4. 如果⼀个消费者失效，消费组⾥的其他消费者可以接管失效消费者的⼯作，再平衡，分区重新分配。

   ![image-20211117010104017](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.11.20/image-20211117010104017-7082066.png)

##### **1.1.5.3 Broker**

⼀个独⽴的Kafka 服务器被称为broker。

broker 为消费者提供服务，对读取分区的请求作出响应，返回已经提交到磁盘上的消息。

1. 如果某topic有N个partition，集群有N个broker，那么每个broker存储该topic的⼀个partition。
2. 如果某topic有N个partition，集群有(N+M)个broker，那么其中有N个broker存储该topic的⼀个partition，剩下的M个broker不存储该topic的partition数据。
3. 如果某topic有N个partition，集群中broker数⽬少于N个，那么⼀个broker存储该topic的⼀个或多个partition。在实际⽣产环境中，尽量避免这种情况的发⽣，这种情况容易导致Kafka集群数据不均衡。

broker 是集群的组成部分。每个集群都有⼀个broker 同时充当了集群控制器的⻆⾊（⾃动从集群的活跃成员中选举出来）。

控制器负责管理⼯作，包括将分区分配给broker 和监控broker。

在集群中，⼀个分区从属于⼀个broker，该broker 被称为分区的⾸领。

![image-20211117010226709](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.11.20/image-20211117010226709-7082148.png)

##### **1.1.5.4 Topic**

每条发布到Kafka集群的消息都有⼀个类别，这个类别被称为Topic。

物理上不同Topic的消息分开存储。

主题就好⽐数据库的表，尤其是分库分表之后的逻辑表。

##### **1.1.5.5 Partition**

1. 主题可以被分为若⼲个分区，⼀个分区就是⼀个提交⽇志。

2. 消息以追加的⽅式写⼊分区，然后以先⼊先出的顺序读取。

3. ⽆法在整个主题范围内保证消息的顺序，但可以保证消息在单个分区内的顺序。

4. Kafka 通过分区来实现数据冗余和伸缩性。

5. 在需要严格保证消息的消费顺序的场景下，需要将partition数⽬设为1。

   ![image-20211117010446186](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.11.20/image-20211117010446186-7082288.png)

##### **1.1.5.6 Replicas**

Kafka 使⽤主题来组织数据，每个主题被分为若⼲个分区，每个分区有多个副本。那些副本被保存在broker 上，

每个broker 可以保存成百上千个属于不同主题和分区的副本。

副本有以下两种类型：

- ⾸领副本：每个分区都有⼀个⾸领副本。为了保证⼀致性，所有⽣产者请求和消费者请求都会经过这个副本。
- 跟随者副本：⾸领以外的副本都是跟随者副本。跟随者副本不处理来⾃客户端的请求，它们唯⼀的任务就是从⾸领那⾥复制消息，保持与⾸领⼀致的状态。如果⾸领发⽣崩溃，其中的⼀个跟随者会被提升为新⾸领。

##### **1.1.5.7 Offset**

⽣产者**Offset**

消息写⼊的时候，每⼀个分区都有⼀个offset，这个offset就是⽣产者的offset，同时也是这个分区的最新最⼤的offset。

有些时候没有指定某⼀个分区的offset，这个⼯作kafka帮我们完成。

![image-20211117010545147](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.11.20/image-20211117010545147-7082347.png)

消费者**Offset**

![image-20211117010610728](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.11.20/image-20211117010610728-7082372.png)

这是某⼀个分区的offset情况，⽣产者写⼊的offset是最新最⼤的值是12，⽽当Consumer A进⾏消费时，从0开始消费，⼀直消费到了9，消费者的offset就记录在9，Consumer B就纪录在了11。等下⼀次他们再来消费时，他们可以选择接着上⼀次的位置消费，当然也可以选择从头消费，或者跳到最近的记录并从“现在”开始消费。

##### **1.1.5.8** 副本

Kafka通过副本保证⾼可⽤。副本分为⾸领副本(Leader)和跟随者副本(Follower)。

跟随者副本包括同步副本和不同步副本，在发⽣⾸领副本切换的时候，只有同步副本可以切换为⾸领副本。

###### **1.1.5.8.1 AR**

分区中的所有副本统称为**AR**（Assigned Repllicas）。

**AR=ISR+OSR**

###### **1.1.5.8.2 ISR**

所有与leader副本保持⼀定程度同步的副本（包括Leader）组成**ISR**（In-Sync Replicas），ISR集合是AR集合中的⼀个⼦集。消息会先发送到leader副本，然后follower副本才能从leader副本中拉取消息进⾏同步，同步期间内follower副本相对于leader副本⽽⾔会有⼀定程度的滞后。前⾯所说的“⼀定程度”是指可以忍受的滞后范围，这个范围可以通过参数进⾏配置。

###### **1.1.5.8.3 OSR**

与leader副本同步滞后过多的副本（不包括leader）副本，组成**OSR**(Out-Sync Relipcas)。在正常情况下，所有的follower副本都应该与leader副本保持⼀定程度的同步，即AR=ISR,OSR集合为空。

###### **1.1.5.8.4 HW**

HW是High Watermak的缩写， 俗称⾼⽔位，它表示了⼀个特定消息的偏移量（offset），消费之只能拉取到这个offset之前的消息。

###### **1.1.5.8.5 LEO**

LEO是Log End Offset的缩写，它表示了当前⽇志⽂件中下⼀条待写⼊消息的offset。

### **1.2 Kafka**开发实战

#### **1.2.1** 消息的发送与接收

![image-20211117011036335](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.11.20/image-20211117011036335-7082639.png)

⽣产者主要的对象有： KafkaProducer ， ProducerRecord 。

其中 KafkaProducer 是⽤于发送消息的类， ProducerRecord 类⽤于封装Kafka的消息。

KafkaProducer 的创建需要指定的参数和含义：

![image-20211117011122039](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.11.20/image-20211117011122039-7082685.png)

其他参数可以从 org.apache.kafka.clients.producer.ProducerConfig 中找到。

消费者⽣产消息后，需要broker端的确认，可以同步确认，也可以异步确认。

同步确认效率低，异步确认效率⾼，但是需要设置回调对象。

⽣产者：

```java
package com.lagou.kafka.demo.producer;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;
import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.TimeoutException;
public class MyProducer1 {
  public static void main(String[] args) throws InterruptedException,
  ExecutionException, TimeoutException {
    Map<String, Object> configs = new HashMap<>();
    // 设置连接Kafka的初始连接⽤到的服务器地址
    // 如果是集群，则可以通过此初始连接发现集群中的其他broker
    configs.put("bootstrap.servers", "node1:9092");
    // 设置key的序列化器
    configs.put("key.serializer",
                "org.apache.kafka.common.serialization.IntegerSerializer");
    // 设置value的序列化器
    configs.put("value.serializer",
                "org.apache.kafka.common.serialization.StringSerializer");
    configs.put("acks", "1");
    KafkaProducer<Integer, String> producer = new KafkaProducer<Integer, String> (configs);
    // ⽤于封装Producer的消息
    ProducerRecord<Integer, String> record = new ProducerRecord<Integer, String>(
      "topic_1", // 主题名称
      0, // 分区编号，现在只有⼀个分区，所以是0 
      0, // 数字作为key
      "message 0" // 字符串作为value
    );
    // 发送消息，同步等待消息的确认
    producer.send(record).get(3_000, TimeUnit.MILLISECONDS);
    // 关闭⽣产者
    producer.close();
  }
}
```

⽣产者2：

```java
package com.lagou.kafka.demo.producer;
import org.apache.kafka.clients.producer.Callback;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.clients.producer.RecordMetadata;
import java.util.HashMap;
import java.util.Map;

public class MyProducer2 {
  public static void main(String[] args) {
    Map<String, Object> configs = new HashMap<>();
    configs.put("bootstrap.servers", "node1:9092");
    configs.put("key.serializer","org.apache.kafka.common.serialization.IntegerSerializer");
    configs.put("value.serializer","org.apache.kafka.common.serialization.StringSerializer");
    KafkaProducer<Integer, String> producer = new KafkaProducer<Integer, String> (configs);
    ProducerRecord<Integer, String> record = new ProducerRecord<Integer, String>("topic_1", 0,1,"lagou message 2");
    // 使⽤回调异步等待消息的确认
    producer.send(record, new Callback() {
      @Override
      public void onCompletion(RecordMetadata metadata, Exception exception) {
        if (exception == null) {
          System.out.println( "主题：" + metadata.topic() + "\n" 
                             + "分区：" + metadata.partition() + "\n" 
                             + "偏移量：" + metadata.offset() + "\n" 
                             + "序列化的key字节：" + metadata.serializedKeySize() + "\n" 
                             + "序列化的value字节：" + metadata.serializedValueSize() + "\n" 
                             + "时间戳：" + metadata.timestamp());
        } else {
          System.out.println("有异常：" + exception.getMessage());
        }
      }
    });
    // 关闭连接
    producer.close();
  }
}
```

⽣产者3：

```java
package com.lagou.kafka.demo.producer;
import org.apache.kafka.clients.producer.Callback;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.clients.producer.RecordMetadata; 
import java.util.HashMap;
import java.util.Map;

public class MyProducer3 {
  public static void main(String[] args) {
    Map<String, Object> configs = new HashMap<>();
    configs.put("bootstrap.servers", "node1:9092");
    configs.put("key.serializer","org.apache.kafka.common.serialization.IntegerSerializer");
    configs.put("value.serializer","org.apache.kafka.common.serialization.StringSerializer");
    KafkaProducer<Integer, String> producer = new KafkaProducer<Integer, String> (configs);
    for (int i = 100; i < 200; i++) {
      ProducerRecord<Integer, String> record = new ProducerRecord<Integer,String>("topic_1", 0,i,"lagou message " + i);
      // 使⽤回调异步等待消息的确认
      producer.send(record, new Callback() {
        @Override
        public void onCompletion(RecordMetadata metadata, Exception exception) {
          if (exception == null) {
            System.out.println( "主题：" + metadata.topic() + "\n" 
                               + "分区：" + metadata.partition() + "\n" 
                               + "偏移量：" + metadata.offset() + "\n" 
                               + "序列化的key字节：" +metadata.serializedKeySize() + "\n" 
                               + "序列化的value字节：" + metadata.serializedValueSize() + "\n" 
                               + "时间戳：" + metadata.timestamp());
          } else {
            System.out.println("有异常：" + exception.getMessage());
          }
        }
      });
    }
    // 关闭连接
    producer.close();
  }
} 
```



消息消费流程：

消费者：

```java
package com.lagou.kafka.demo.consumer;
import org.apache.kafka.clients.consumer.ConsumerRebalanceListener;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.common.TopicPartition;
import java.lang.reflect.Array;
import java.util.*;
import java.util.regex.Pattern;
public class MyConsumer1 {
	public static void main(String[] args) {
		Map<String, Object> configs = new HashMap<>();
    // 指定bootstrap.servers属性作为初始化连接Kafka的服务器。
    // 如果是集群，则会基于此初始化连接发现集群中的其他服务器。
    configs.put("bootstrap.servers", "node1:9092");
    // key的反序列化器
    configs.put("key.deserializer",
                "org.apache.kafka.common.serialization.IntegerDeserializer");
    // value的反序列化器
    configs.put("value.deserializer",
                "org.apache.kafka.common.serialization.StringDeserializer");
    configs.put("group.id", "consumer.demo");
    // 创建消费者对象
    KafkaConsumer<Integer, String> consumer = new KafkaConsumer<Integer, String>(configs);
    // final Pattern pattern = Pattern.compile("topic_\\d");
    final Pattern pattern = Pattern.compile("topic_[0-9]");
    // 消费者订阅主题或分区
    // consumer.subscribe(pattern);
    // consumer.subscribe(pattern, new ConsumerRebalanceListener() {
    final List<String> topics = Arrays.asList("topic_1");
    consumer.subscribe(topics, new ConsumerRebalanceListener() {
      @Override
      public void onPartitionsRevoked(Collection<TopicPartition> partitions) {
      	partitions.forEach(tp -> {
      		System.out.println("剥夺的分区：" + tp.partition());
       	});
      }
      @Override
      public void onPartitionsAssigned(Collection<TopicPartition> partitions) {
      	partitions.forEach(tp -> {
          System.out.println(tp.partition());
        });
      }
    });
    // 拉取订阅主题的消息
    final ConsumerRecords<Integer, String> records = consumer.poll(3_000);
    // 获取topic_1主题的消息
    final Iterable<ConsumerRecord<Integer, String>> topic1Iterable =
    records.records("topic_1");
    // 遍历topic_1主题的消息
    topic1Iterable.forEach(record -> {
      System.out.println("========================================");
      System.out.println("消息头字段：" +
      Arrays.toString(record.headers().toArray()));
      System.out.println("消息的key：" + record.key());
      System.out.println("消息的偏移量：" + record.offset());
      System.out.println("消息的分区号：" + record.partition());
      System.out.println("消息的序列化key字节数：" + record.serializedKeySize());
      System.out.println("消息的序列化value字节数：" +
      record.serializedValueSize());
      System.out.println("消息的时间戳：" + record.timestamp());
      System.out.println("消息的时间戳类型：" + record.timestampType());
      System.out.println("消息的主题：" + record.topic());
      System.out.println("消息的值：" + record.value());
    });
    // 关闭消费者
    consumer.close();
  }
}
```





#### **1.2.2 SpringBoot Kafka**

1. pom.xml⽂件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
https://maven.apache.org/xsd/maven-4.0.0.xsd"> 
  <modelVersion>4.0.0</modelVersion> 
  <parent> 
    <groupId>org.springframework.boot</groupId> 
    <artifactId>spring-boot-starter-parent</artifactId> 
    <version>2.2.8.RELEASE</version> 
    <relativePath/> <!-- lookup parent from repository -->
  </parent> 
  <groupId>com.lagou.kafka.demo</groupId> 
  <artifactId>demo-02-springboot</artifactId> 
  <version>0.0.1-SNAPSHOT</version> 
  <name>demo-02-springboot</name> 
  <description>Demo project for Spring Boot</description> 
  <properties> 
    <java.version>1.8</java.version>
  </properties> 
  <dependencies> 
    <dependency> 
      <groupId>org.springframework.boot</groupId> 
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency> 
    <dependency> 
      <groupId>org.springframework.kafka</groupId> 
      <artifactId>spring-kafka</artifactId>
    </dependency> 
    <dependency> 
      <groupId>org.springframework.boot</groupId> 
      <artifactId>spring-boot-starter-test</artifactId> 
      <scope>test</scope> 
      <exclusions> 
        <exclusion> 
          <groupId>org.junit.vintage</groupId> 
          <artifactId>junit-vintage-engine</artifactId>
        </exclusion>
      </exclusions>
    </dependency> 
    <dependency> 
      <groupId>org.springframework.kafka</groupId> 
      <artifactId>spring-kafka-test</artifactId> 
      <scope>test</scope> 
    </dependency>
  </dependencies> 
  <build> 
    <plugins> 
      <plugin> 
        <groupId>org.springframework.boot</groupId> 
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>
</project>
```

2. application.properties

   ```properties
   spring.application.name=springboot-kafka-02
   server.port=8080
   # ⽤于建⽴初始连接的broker地址
   spring.kafka.bootstrap-servers=node1:9092
   # producer⽤到的key和value的序列化类
   spring.kafka.producer.key
   serializer=org.apache.kafka.common.serialization.IntegerSerializer
   spring.kafka.producer.value
   serializer=org.apache.kafka.common.serialization.StringSerializer
   # 默认的批处理记录数
   spring.kafka.producer.batch-size=16384
   # 32MB的总发送缓存
   spring.kafka.producer.buffer-memory=33554432
   # consumer⽤到的key和value的反序列化类
   spring.kafka.consumer.key.deserializer=org.apache.kafka.common.serialization.IntegerDeserializer
   spring.kafka.consumer.value.deserializer=org.apache.kafka.common.serialization.StringDeserializer
   # consumer的消费组id
   spring.kafka.consumer.group-id=spring-kafka-02-consumer
   # 是否⾃动提交消费者偏移量
   spring.kafka.consumer.enable-auto-commit=true
   # 每隔100ms向broker提交⼀次偏移量
   spring.kafka.consumer.auto-commit-interval=100
   # 如果该消费者的偏移量不存在，则⾃动设置为最早的偏移量
   spring.kafka.consumer.auto-offset-reset=earliest
   ```
   
   
   
3. Demo02SpringbootApplication.java

   ```java
   package com.lagou.kafka.demo;
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   
   @SpringBootApplication
   public class Demo02SpringbootApplication {
     public static void main(String[] args) {
       SpringApplication.run(Demo02SpringbootApplication.class, args);
     }
   } 
   ```
   
4. KafkaConfig.java

   ```java
   package com.lagou.kafka.demo.config;
   import org.apache.kafka.clients.admin.NewTopic;
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   @Configuration
   public class KafkaConfig {
   @Bean
   public NewTopic topic1() {
   return new NewTopic("ntp-01", 5, (short) 1);
    }
   @Bean
   public NewTopic topic2() {
   return new NewTopic("ntp-02", 3, (short) 1);
    }
   } 
   
   
   
   
   ```
   
   
   
5. KafkaSyncProducerController.java

   ```java
   package com.lagou.kafka.demo.controller;
   import org.apache.kafka.clients.producer.ProducerRecord;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.kafka.core.KafkaTemplate;
   import org.springframework.kafka.support.SendResult;
   import org.springframework.util.concurrent.ListenableFuture; 
   import org.springframework.web.bind.annotation.PathVariable;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.RestController;
   import java.util.concurrent.ExecutionException;
   @RestController
   public class KafkaSyncProducerController {
     @Autowired
     private KafkaTemplate template;
     @RequestMapping("send/sync/{message}")
     public String sendSync(@PathVariable String message) {
       ListenableFuture future = template.send(
         new ProducerRecord<Integer, String>(
           "topic-spring-02", 
           0,
           1,
           message));
       try {
         // 同步等待broker的响应
         Object o = future.get();
         SendResult<Integer, String> result = (SendResult<Integer, String>) o;
         System.out.println(result.getRecordMetadata().topic()
                            + result.getRecordMetadata().partition()
                            + result.getRecordMetadata().offset());
       } catch (InterruptedException e) {
         e.printStackTrace();
       } catch (ExecutionException e) {
         e.printStackTrace();
       }
       return "success";
     }
   } 
   
   ```
   
   
   
6. KafkaAsyncProducerController

   ```java
   package com.lagou.kafka.demo.controller;
   import org.apache.kafka.clients.producer.ProducerRecord;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.kafka.core.KafkaTemplate;
   import org.springframework.kafka.support.SendResult; 
   import org.springframework.util.concurrent.ListenableFuture;
   import org.springframework.util.concurrent.ListenableFutureCallback;
   import org.springframework.web.bind.annotation.PathVariable;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.RestController;
   @RestController
   public class KafkaAsyncProducerController {
   @Autowired
   private KafkaTemplate<Integer, String> template;
   @RequestMapping("send/async/{message}")
   public String asyncSend(@PathVariable String message) {
   ProducerRecord<Integer, String> record = new ProducerRecord<Integer, String>(
   "topic-spring-02", 
   0,
   3,
   message
    );
   ListenableFuture<SendResult<Integer, String>> future = template.send(record);
   // 添加回调，异步等待响应
   future.addCallback(new ListenableFutureCallback<SendResult<Integer, String>>
   () {
   @Override
   public void onFailure(Throwable throwable) {
   System.out.println("发送失败: " + throwable.getMessage());
    }
   @Override
   public void onSuccess(SendResult<Integer, String> result) {
   System.out.println("发送成功：" +
   result.getRecordMetadata().topic() + "\t"
   \+ result.getRecordMetadata().partition() + "\t"
   \+ result.getRecordMetadata().offset());
    }
    });
   return "success";
    }
   } 
   
   
   ```
   
7. MyConsumer.java

   ```java
   package com.lagou.kafka.demo.consumer;
   import org.apache.kafka.clients.consumer.ConsumerRecord; 
   import org.springframework.kafka.annotation.KafkaListener;
   import org.springframework.stereotype.Component;
   import java.util.Optional;
   @Component
   public class MyConsumer {
   @KafkaListener(topics = "topic-spring-02")
   public void onMessage(ConsumerRecord<Integer, String> record) {
   Optional<ConsumerRecord<Integer, String>> optional =
   Optional.ofNullable(record);
   if (optional.isPresent()) {
   System.out.println(
   record.topic() + "\t"
   \+ record.partition() + "\t"
   \+ record.offset() + "\t"
   \+ record.key() + "\t"
   \+ record.value());
    }
    }
   } 
   ```
   
   

### **1.3** 服务端参数配置

$KAFKA_HOME/config/server.properties⽂件中的配置。

#### **1.3.1 zookeeper.connect**

该参数⽤于配置Kafka要连接的Zookeeper/集群的地址。

它的值是⼀个字符串，使⽤逗号分隔Zookeeper的多个地址。Zookeeper的单个地址是 host:port 形式的，可以在最后添加Kafka在Zookeeper中的根节点路径。

如：

```properties
zookeeper.connect=node2:2181,node3:2181,node4:2181/myKafka
```

#### **1.3.2 listeners**

⽤于指定当前Broker向外发布服务的地址和端⼝。

与 advertised.listeners 配合，⽤于做内外⽹隔离。

内外⽹隔离配置：

**listener.security.protocol.map**

监听器名称和安全协议的映射配置。

⽐如，可以将内外⽹隔离，即使它们都使⽤SSL。

listener.security.protocol.map=INTERNAL:SSL,EXTERNAL:SSL

每个监听器的名称只能在map中出现⼀次。

**inter.broker.listener.name**

⽤于配置broker之间通信使⽤的监听器名称，该名称必须在advertised.listeners列表中。

inter.broker.listener.name=EXTERNAL

**listeners**

⽤于配置broker监听的URI以及监听器名称列表，使⽤逗号隔开多个URI及监听器名称。如果监听器名称代表的不是安全协议，必须配置listener.security.protocol.map。

每个监听器必须使⽤不同的⽹络端⼝。

**advertised.listeners**

需要将该地址发布到zookeeper供客户端使⽤，如果客户端使⽤的地址与listeners配置不同。

可以在zookeeper的 get /myKafka/brokers/ids/<broker.id> 中找到。

在IaaS环境，该条⽬的⽹络接⼝得与broker绑定的⽹络接⼝不同。

如果不设置此条⽬，就使⽤listeners的配置。跟listeners不同，该条⽬不能使⽤0.0.0.0⽹络端⼝。

advertised.listeners的地址必须是listeners中配置的或配置的⼀部分。

典型配置：

#### **1.3.3 broker.id**

该属性⽤于唯⼀标记⼀个Kafka的Broker，它的值是⼀个任意integer值。

当Kafka以分布式集群运⾏的时候，尤为重要。

最好该值跟该Broker所在的物理主机有关的，如主机名为 host1.lagou.com ，则 broker.id=1 ，如果主机名

为 192.168.100.101 ，则 broker.id=101 等等。

#### **1.3.4 log.dir**

通过该属性的值，指定Kafka在磁盘上保存消息的⽇志⽚段的⽬录。

它是⼀组⽤逗号分隔的本地⽂件系统路径。

如果指定了多个路径，那么broker 会根据“最少使⽤”原则，把同⼀个分区的⽇志⽚段保存到同⼀个路径下。

broker 会往拥有最少数⽬分区的路径新增分区，⽽不是往拥有最⼩磁盘空间的路径新增分区。

## 2. **Kafka**⾼级特性解析

### **2.1** ⽣产者

#### **2.1.1** 消息发送

##### **2.1.1.1** 数据⽣产流程解析

1. Producer创建时，会创建⼀个Sender线程并设置为守护线程。
2. ⽣产消息时，内部其实是异步流程；⽣产的消息先经过拦截器->序列化器->分区器，然后将消息缓存在缓冲区（该缓冲区也是在Producer创建时创建）。
3. 批次发送的条件为：缓冲区数据⼤⼩达到batch.size或者linger.ms达到上限，哪个先达到就算哪个。
4. 批次发送后，发往指定分区，然后落盘到broker；如果⽣产者配置了retrires参数⼤于0并且失败原因允许重试，那么客户端内部会对该消息进⾏重试。
5. 落盘到broker成功，返回⽣产元数据给⽣产者。
6. 元数据返回有两种⽅式：⼀种是通过阻塞直接返回，另⼀种是通过回调返回。

##### **2.1.1.2** 必要参数配置

###### **2.1.1.2.1 broker**配置

1. 配置条⽬的使⽤⽅式：

   ![image-20211117020739008](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.11.20/image-20211117020739008.png)

2. 配置参数：

   ![image-20211117020718909](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.11.20/image-20211117020718909-7086041.png)

##### **2.1.1.3** 序列化器

由于Kafka中的数据都是字节数组，在将消息发送到Kafka之前需要先将数据序列化为字节数组。

序列化器的作⽤就是⽤于序列化要发送的消息的。

Kafka使⽤ org.apache.kafka.common.serialization.Serializer 接⼝⽤于定义序列化器，将泛型指定类型的数据转换为字节数组。

```java
package org.apache.kafka.common.serialization;
import java.io.Closeable;
import java.util.Map;
/**
\* 将对象转换为byte数组的接⼝
*
\* 该接⼝的实现类需要提供⽆参构造器
\* @param <T> 从哪个类型转换
*/
public interface Serializer<T> extends Closeable {
/**
\* 类的配置信息
\* @param configs key/value pairs
\* @param isKey key的序列化还是value的序列化
*/
void configure(Map<String, ?> configs, boolean isKey);
系统提供了该接⼝的⼦接⼝以及实现类：
org.apache.kafka.common.serialization.ByteArraySerializer
org.apache.kafka.common.serialization.ByteBufferSerializer
/**
\* 将对象转换为字节数组
*
\* @param topic 主题名称
\* @param data 需要转换的对象
\* @return 序列化的字节数组
*/
byte[] serialize(String topic, T data);
/**
\* 关闭序列化器
\* 该⽅法需要提供幂等性，因为可能调⽤多次。
*/
@Override
void close();
}
```



org.apache.kafka.common.serialization.BytesSerializer

org.apache.kafka.common.serialization.DoubleSerializer

org.apache.kafka.common.serialization.FloatSerializer

org.apache.kafka.common.serialization.IntegerSerializer

org.apache.kafka.common.serialization.StringSerializer

org.apache.kafka.common.serialization.LongSerializer

org.apache.kafka.common.serialization.ShortSerializer

**2.1.1.3.1** ⾃定义序列化器

数据的序列化⼀般⽣产中使⽤avro。

⾃定义序列化器需要实现org.apache.kafka.common.serialization.Serializer<T>接⼝，并实现其中的

serialize ⽅法。

案例：

实体类：

```java
package com.lagou.kafka.demo.entity;
public class User {
private Integer userId;
private String username;
public Integer getUserId() {
return userId;
 }
public void setUserId(Integer userId) {
this.userId = userId;
 }
public String getUsername() {
return username;
 }
public void setUsername(String username) {
this.username = username;
 }
}
```

序列化类：

```java
package com.lagou.kafka.demo.serializer;
import com.lagou.kafka.demo.entity.User;
import org.apache.kafka.common.errors.SerializationException;
import org.apache.kafka.common.serialization.Serializer;
import java.io.UnsupportedEncodingException;
import java.nio.Buffer;
import java.nio.ByteBuffer;
import java.util.Map;
public class UserSerializer implements Serializer<User> {
@Override
public void configure(Map<String, ?> configs, boolean isKey) {
// do nothing
 }
@Override
public byte[] serialize(String topic, User data) {
try {
// 如果数据是null，则返回null
if (data == null) return null;
Integer userId = data.getUserId();
String username = data.getUsername();
int length = 0;
byte[] bytes = null;
if (null != username) {
bytes = username.getBytes("utf-8");
length = bytes.length;
 }
ByteBuffer buffer = ByteBuffer.allocate(4 + 4 + length);
buffer.putInt(userId);
buffer.putInt(length);
buffer.put(bytes);
return buffer.array();
 } catch (UnsupportedEncodingException e) {
throw new SerializationException("序列化数据异常");
 }
 }
@Override
public void close() {
// do nothing
 }
}
```

⽣产者：

```java
package com.lagou.kafka.demo.producer;
import com.lagou.kafka.demo.entity.User;
import com.lagou.kafka.demo.serializer.UserSerializer;
import org.apache.kafka.clients.producer.*;
import org.apache.kafka.common.serialization.StringSerializer;
import java.util.HashMap;
import java.util.Map;
public class MyProducer {
public static void main(String[] args) {
Map<String, Object> configs = new HashMap<>();
configs.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "node1:9092");
configs.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG,
StringSerializer.class);
// 设置⾃定义的序列化类
configs.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,
UserSerializer.class);
KafkaProducer<String, User> producer = new KafkaProducer<String, User> 
(configs);
User user = new User();
user.setUserId(1001);
user.setUsername("张三");
ProducerRecord<String, User> record = new ProducerRecord<>(
"tp_user_01", 
0,
user.getUsername(),
user
 );
producer.send(record, (metadata, exception) -> {
if (exception == null) {
System.out.println("消息发送成功：" 
\+ metadata.topic() + "\t"
\+ metadata.partition() + "\t"
\+ metadata.offset());
 } else {
System.out.println("消息发送异常");
 }
 });
// 关闭⽣产者
producer.close();
 }
}
```

##### **2.1.1.4** 分区器

默认（DefaultPartitioner）分区计算：

1. 如果record提供了分区号，则使⽤record提供的分区号
2. 如果record没有提供分区号，则使⽤key的序列化后的值的hash值对分区数量取模
3.  如果record没有提供分区号，也没有提供key，则使⽤轮询的⽅式分配分区号。
   1. 会⾸先在可⽤的分区中分配分区号
   2. 如果没有可⽤的分区，则在该主题所有分区中分配分区号。

如果要⾃定义分区器，则需要

1. ⾸先开发Partitioner接⼝的实现类
2. 在KafkaProducer中进⾏设置：configs.put("partitioner.class", "xxx.xx.Xxx.class")

位于 org.apache.kafka.clients.producer 中的分区器接⼝：

```java
package org.apache.kafka.clients.producer;
import org.apache.kafka.common.Configurable;
import org.apache.kafka.common.Cluster; 
import java.io.Closeable;

/**
 * 分区器接⼝
 */

public interface Partitioner extends Configurable, Closeable {

  /**
    * 为指定的消息记录计算分区值
    *
    * @param topic 主题名称
    * @param key 根据该key的值进⾏分区计算，如果没有则为null。
    * @param keyBytes key的序列化字节数组，根据该数组进⾏分区计算。如果没有key，则为null
    * @param value 根据value值进⾏分区计算，如果没有，则为null
    * @param valueBytes value的序列化字节数组，根据此值进⾏分区计算。如果没有，则为null
    * @param cluster 当前集群的元数据
    */
  public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster);
  /**
  * 关闭分区器的时候调⽤该⽅法
  */
  public void close();
} 
```



包 org.apache.kafka.clients.producer.internals 中分区器的默认实现：

```java
package org.apache.kafka.clients.producer.internals;

import java.util.List;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.ConcurrentMap;
import java.util.concurrent.ThreadLocalRandom;
import java.util.concurrent.atomic.AtomicInteger;
import org.apache.kafka.clients.producer.Partitioner;
import org.apache.kafka.common.Cluster;
import org.apache.kafka.common.PartitionInfo;
import org.apache.kafka.common.utils.Utils;
/**
\* 默认的分区策略：
*
\* 如果在记录中指定了分区，则使⽤指定的分区
\* 如果没有指定分区，但是有key的值，则使⽤key值的散列值计算分区
* 如果没有指定分区也没有key的值，则使⽤轮询的⽅式选择⼀个分区
*/
public class DefaultPartitioner implements Partitioner {
private final ConcurrentMap<String, AtomicInteger> topicCounterMap = new
ConcurrentHashMap<>();
public void configure(Map<String, ?> configs) {}
/**
\* 为指定的消息记录计算分区值
*
\* @param topic 主题名称
\* @param key 根据该key的值进⾏分区计算，如果没有则为null。
\* @param keyBytes key的序列化字节数组，根据该数组进⾏分区计算。如果没有key，则为null
\* @param value 根据value值进⾏分区计算，如果没有，则为null
\* @param valueBytes value的序列化字节数组，根据此值进⾏分区计算。如果没有，则为null
\* @param cluster 当前集群的元数据
*/
public int partition(String topic, Object key, byte[] keyBytes, Object value,
byte[] valueBytes, Cluster cluster) {
// 获取指定主题的所有分区信息
List<PartitionInfo> partitions = cluster.partitionsForTopic(topic);
// 分区的数量
int numPartitions = partitions.size();
// 如果没有提供key
if (keyBytes == null) {
int nextValue = nextValue(topic);
List<PartitionInfo> availablePartitions =
cluster.availablePartitionsForTopic(topic);
if (availablePartitions.size() > 0) {
int part = Utils.toPositive(nextValue) % availablePartitions.size();
return availablePartitions.get(part).partition();
 } else {
// no partitions are available, give a non-available partition
return Utils.toPositive(nextValue) % numPartitions;
 }
 } else {
// hash the keyBytes to choose a partition
// 如果有，就计算keyBytes的哈希值，然后对当前主题的个数取模
return Utils.toPositive(Utils.murmur2(keyBytes)) % numPartitions;
 }
 }
private int nextValue(String topic) {
AtomicInteger counter = topicCounterMap.get(topic);
if (null == counter) {
counter = new AtomicInteger(ThreadLocalRandom.current().nextInt());
AtomicInteger currentCounter = topicCounterMap.putIfAbsent(topic,
counter);
if (currentCounter != null) {
counter = currentCounter;
 }
 }
return counter.getAndIncrement();
 }
public void close() {}
}
```

可以实现Partitioner接⼝⾃定义分区器：

然后在⽣产者中配置：





**2.1.1.5** 拦截器

Producer拦截器（interceptor）和Consumer端Interceptor是在Kafka 0.10版本被引⼊的，主要⽤于实现Client端的定制化控制逻辑。

对于Producer⽽⾔，Interceptor使得⽤户在消息发送前以及Producer回调逻辑前有机会对消息做⼀些定制化需求，⽐如修改消息等。同时，Producer允许⽤户指定多个Interceptor按序作⽤于同⼀条消息从⽽形成⼀个拦截链(interceptor chain)。Intercetpor的实现接⼝是org.apache.kafka.clients.producer.ProducerInterceptor，其定义的⽅法包括：

- onSend(ProducerRecord)：该⽅法封装进KafkaProducer.send⽅法中，即运⾏在⽤户主线程中。Producer确保在消息被序列化以计算分区前调⽤该⽅法。⽤户可以在该⽅法中对消息做任何操作，但最好保证不要修改消息所属的topic和分区，否则会影响⽬标分区的计算。
- onAcknowledgement(RecordMetadata, Exception)：该⽅法会在消息被应答之前或消息发送失败时调⽤，并且通常都是在Producer回调逻辑触发之前。onAcknowledgement运⾏在Producer的IO线程中，因此不要在该⽅法中放⼊很重的逻辑，否则会拖慢Producer的消息发送效率。
- close：关闭Interceptor，主要⽤于执⾏⼀些资源清理⼯作。

如前所述，Interceptor可能被运⾏在多个线程中，因此在具体实现时⽤户需要⾃⾏确保线程安全。另外倘若指定了多个Interceptor，则Producer将按照指定顺序调⽤它们，并仅仅是捕获每个Interceptor可能抛出的异常记录到错误⽇志中⽽⾮在向上传递。这在使⽤过程中要特别留意。

⾃定义拦截器：

1. 实现ProducerInterceptor接⼝
2. 在KafkaProducer的设置中设置⾃定义的拦截器

案例：

1. 消息实体类：

   ```
   package com.lagou.kafka.demo.entity;
   
   public class User {
   
   private Integer userId;
   
   private String username;
   
   public Integer getUserId() {
   
   return userId;
   
    }
   
   public void setUserId(Integer userId) {
   
   this.userId = userId;
   
    }
   
   public String getUsername() {
   
   return username;
   
    }
   
   public void setUsername(String username) {
   
   this.username = username;
   
    }
   
   } 
   ```

   

2. ⾃定义序列化器

   ```
   package com.lagou.kafka.demo.serializer;
   
   import com.lagou.kafka.demo.entity.User;
   
   import org.apache.kafka.common.errors.SerializationException;
   
   import org.apache.kafka.common.serialization.Serializer;
   
   import java.io.UnsupportedEncodingException;
   
   import java.nio.Buffer;
   
   import java.nio.ByteBuffer;
   
   import java.util.Map;
   
   public class UserSerializer implements Serializer<User> {
   
   @Override
   
   public void configure(Map<String, ?> configs, boolean isKey) {
   
   // do nothing
   
   
   
    }
   
   @Override
   
   public byte[] serialize(String topic, User data) {
   
   try {
   
   // 如果数据是null，则返回null
   
   if (data == null) return null;
   
   Integer userId = data.getUserId();
   
   String username = data.getUsername();
   
   int length = 0;
   
   byte[] bytes = null;
   
   if (null != username) {
   
   bytes = username.getBytes("utf-8");
   
   length = bytes.length;
   
    }
   
   ByteBuffer buffer = ByteBuffer.allocate(4 + 4 + length);
   
   buffer.putInt(userId);
   
   buffer.putInt(length);
   
   buffer.put(bytes);
   
   return buffer.array();
   
    } catch (UnsupportedEncodingException e) {
   
   throw new SerializationException("序列化数据异常");
   
    }
   
    }
   
   @Override
   
   public void close() {
   
   // do nothing
   
    }
   
   }
   ```

3. ⾃定义分区器

   ```
   package com.lagou.kafka.demo.partitioner;
   
   import org.apache.kafka.clients.producer.Partitioner;
   
   import org.apache.kafka.common.Cluster;
   
   import java.util.Map;
   
   public class MyPartitioner implements Partitioner {
   
   @Override
   
   public int partition(String topic, Object key, byte[] keyBytes, Object value,
   
   byte[] valueBytes, Cluster cluster) {
   
   return 2; 
   
   
   
    }
   
   @Override
   
   public void close() {
   
    }
   
   @Override
   
   public void configure(Map<String, ?> configs) {
   
    }
   
   }
   ```

4. ⾃定义拦截器**1**

   ```
   4. package com.lagou.kafka.demo.interceptor;
   
   import org.apache.kafka.clients.producer.ProducerInterceptor;
   
   import org.apache.kafka.clients.producer.ProducerRecord;
   
   import org.apache.kafka.clients.producer.RecordMetadata;
   
   import org.apache.kafka.common.header.Headers;
   
   import org.slf4j.Logger;
   
   import org.slf4j.LoggerFactory;
   
   import java.util.Map;
   
   public class InterceptorOne<KEY, VALUE> implements ProducerInterceptor<KEY, VALUE> {
   
   private static final Logger LOGGER =
   
   LoggerFactory.getLogger(InterceptorOne.class);
   
   @Override
   
   public ProducerRecord<KEY, VALUE> onSend(ProducerRecord<KEY, VALUE> record) {
   
   System.out.println("拦截器1---go");
   
   // 此处根据业务需要对相关的数据作修改
   
   String topic = record.topic();
   
   Integer partition = record.partition();
   
   Long timestamp = record.timestamp();
   
   KEY key = record.key();
   
   VALUE value = record.value();
   
   Headers headers = record.headers();
   
   // 添加消息头
   
   headers.add("interceptor", "interceptorOne".getBytes());
   
   ProducerRecord<KEY, VALUE> newRecord = new ProducerRecord<KEY, VALUE>(
   
   topic,
   
   partition,
   
   timestamp,
   
   key, 
   
   5. ⾃定义拦截器**2**
   
   value,
   
   headers
   
    );
   
   return newRecord;
   
    }
   
   @Override
   
   public void onAcknowledgement(RecordMetadata metadata, Exception exception) {
   
   System.out.println("拦截器1---back");
   
   if (exception != null) {
   
   // 如果发⽣异常，记录⽇志中
   
   LOGGER.error(exception.getMessage());
   
    }
   
    }
   
   @Override
   
   public void close() {
   
    }
   
   @Override
   
   public void configure(Map<String, ?> configs) {
   
    }
   
   }
   ```

   

5. 自定义拦截器**2**

   ```
   package com.lagou.kafka.demo.interceptor;
   
   import org.apache.kafka.clients.producer.ProducerInterceptor;
   
   import org.apache.kafka.clients.producer.ProducerRecord;
   
   import org.apache.kafka.clients.producer.RecordMetadata;
   
   import org.apache.kafka.common.header.Headers;
   
   import org.slf4j.Logger;
   
   import org.slf4j.LoggerFactory;
   
   import java.util.Map;
   
   public class InterceptorTwo<KEY, VALUE> implements ProducerInterceptor<KEY, VALUE> {
   
   private static final Logger LOGGER =
   
   LoggerFactory.getLogger(InterceptorTwo.class);
   
   @Override
   
   public ProducerRecord<KEY, VALUE> onSend(ProducerRecord<KEY, VALUE> record) {
   
   System.out.println("拦截器2---go");
   
   // 此处根据业务需要对相关的数据作修改
   
   6. ⾃定义拦截器**3**
   
   String topic = record.topic();
   
   Integer partition = record.partition();
   
   Long timestamp = record.timestamp();
   
   KEY key = record.key();
   
   VALUE value = record.value();
   
   Headers headers = record.headers();
   
   // 添加消息头
   
   headers.add("interceptor", "interceptorTwo".getBytes());
   
   ProducerRecord<KEY, VALUE> newRecord = new ProducerRecord<KEY, VALUE>(
   
   topic,
   
   partition,
   
   timestamp,
   
   key,
   
   value,
   
   headers
   
    );
   
   return newRecord;
   
    }
   
   @Override
   
   public void onAcknowledgement(RecordMetadata metadata, Exception exception) {
   
   System.out.println("拦截器2---back");
   
   if (exception != null) {
   
   // 如果发⽣异常，记录⽇志中
   
   LOGGER.error(exception.getMessage());
   
    }
   
    }
   
   @Override
   
   public void close() {
   
    }
   
   @Override
   
   public void configure(Map<String, ?> configs) {
   
    }
   
   }
   ```

   

6. 自定义拦截器**3**

   ```
   package com.lagou.kafka.demo.interceptor;
   
   import org.apache.kafka.clients.producer.ProducerInterceptor;
   
   import org.apache.kafka.clients.producer.ProducerRecord;
   
   import org.apache.kafka.clients.producer.RecordMetadata;
   
   import org.apache.kafka.common.header.Headers; 
   
   import org.slf4j.Logger;
   
   import org.slf4j.LoggerFactory;
   
   import java.util.Map;
   
   public class InterceptorThree<KEY, VALUE> implements ProducerInterceptor<KEY, VALUE> 
   
   {
   
   private static final Logger LOGGER =
   
   LoggerFactory.getLogger(InterceptorThree.class);
   
   @Override
   
   public ProducerRecord<KEY, VALUE> onSend(ProducerRecord<KEY, VALUE> record) {
   
   System.out.println("拦截器3---go");
   
   // 此处根据业务需要对相关的数据作修改
   
   String topic = record.topic();
   
   Integer partition = record.partition();
   
   Long timestamp = record.timestamp();
   
   KEY key = record.key();
   
   VALUE value = record.value();
   
   Headers headers = record.headers();
   
   // 添加消息头
   
   headers.add("interceptor", "interceptorThree".getBytes());
   
   ProducerRecord<KEY, VALUE> newRecord = new ProducerRecord<KEY, VALUE>(
   
   topic,
   
   partition,
   
   timestamp,
   
   key,
   
   value,
   
   headers
   
    );
   
   return newRecord;
   
    }
   
   @Override
   
   public void onAcknowledgement(RecordMetadata metadata, Exception exception) {
   
   System.out.println("拦截器3---back");
   
   if (exception != null) {
   
   // 如果发⽣异常，记录⽇志中
   
   LOGGER.error(exception.getMessage());
   
    }
   
    }
   
   @Override
   
   public void close() {
   
    }
   
   @Override
   
   7. ⽣产者
   
   public void configure(Map<String, ?> configs) {
   
    }
   
   }
   ```

   

7. 生产者

   ```
   package com.lagou.kafka.demo.producer;
   
   import com.lagou.kafka.demo.entity.User;
   
   import com.lagou.kafka.demo.serializer.UserSerializer;
   
   import org.apache.kafka.clients.producer.KafkaProducer;
   
   import org.apache.kafka.clients.producer.ProducerConfig;
   
   import org.apache.kafka.clients.producer.ProducerRecord;
   
   import org.apache.kafka.common.serialization.StringSerializer;
   
   import java.util.HashMap;
   
   import java.util.Map;
   
   public class MyProducer {
   
   public static void main(String[] args) {
   
   Map<String, Object> configs = new HashMap<>();
   
   configs.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "node1:9092");
   
   // 设置⾃定义分区器
   
   // configs.put(ProducerConfig.PARTITIONER_CLASS_CONFIG, MyPartitioner.class);
   
   configs.put("partitioner.class",
   
   "com.lagou.kafka.demo.partitioner.MyPartitioner");
   
   // 设置拦截器
   
   configs.put(ProducerConfig.INTERCEPTOR_CLASSES_CONFIG,
   
   "com.lagou.kafka.demo.interceptor.InterceptorOne," +
   
   "com.lagou.kafka.demo.interceptor.InterceptorTwo," +
   
   "com.lagou.kafka.demo.interceptor.InterceptorThree"
   
    );
   
   configs.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG,
   
   StringSerializer.class);
   
   // 设置⾃定义的序列化类
   
   configs.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,
   
   UserSerializer.class);
   
   KafkaProducer<String, User> producer = new KafkaProducer<String, User> 
   
   (configs);
   
   User user = new User();
   
   user.setUserId(1001);
   user.setUsername("张三");
   
   ProducerRecord<String, User> record = new ProducerRecord<>(
   
   "tp_user_01", 
   
   0,
   
   user.getUsername(),
   
   user
   
    );
   
   producer.send(record, (metadata, exception) -> {
   
   if (exception == null) {
   
   System.out.println("消息发送成功：" 
   
   \+ metadata.topic() + "\t"
   
   \+ metadata.partition() + "\t"
   
   \+ metadata.offset());
   
    } else {
   
   System.out.println("消息发送异常");
   
    }
   
    });
   
   // 关闭⽣产者
   
   producer.close();
   
    }
   
   }
   ```

   

8. 运⾏结果：

**2.1.2** 原理剖析

![image-20211120181959252](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.11.20/image-20211120181959252.png)

由上图可以看出：KafkaProducer有两个基本线程：

- 主线程：负责消息创建，拦截器，序列化器，分区器等操作，并将消息追加到消息收集器RecoderAccumulator中；
  - 消息收集器RecoderAccumulator为每个分区都维护了⼀个 Deque<ProducerBatch> 类型的双端队列。
  - ProducerBatch 可以理解为是 ProducerRecord 的集合，批量发送有利于提升吞吐量，降低⽹络影响；
  - 由于⽣产者客户端使⽤ java.io.ByteBuffer 在发送消息之前进⾏消息保存，并维护了⼀个BufferPool 实现 ByteBuffer 的复⽤；该缓存池只针对特定⼤⼩（ batch.size 指定）的 ByteBuffer进⾏管理，对于消息过⼤的缓存，不能做到重复利⽤。
  - 每次追加⼀条ProducerRecord消息，会寻找/新建对应的双端队列，从其尾部获取⼀个ProducerBatch，判断当前消息的⼤⼩是否可以写⼊该批次中。若可以写⼊则写⼊；若不可以写⼊，则新建⼀个ProducerBatch，判断该消息⼤⼩是否超过客户端参数配置 batch.size 的值，不超过，则以 batch.size建⽴新的ProducerBatch，这样⽅便进⾏缓存重复利⽤；若超过，则以计算的消息⼤⼩建⽴对应的 ProducerBatch ，缺点就是该内存不能被复⽤了。

- Sender线程：
  - 该线程从消息收集器获取缓存的消息，将其处理为 <Node, List<ProducerBatch> 的形式， Node参数表示集群的broker节点。
  - 进⼀步将<Node, List<ProducerBatch>转化为<Node, Request>形式，此时才可以向服务端发送数据。
  - 在发送之前，Sender线程将消息以 Map<NodeId, Deque<Request>> 的形式保存到InFlightRequests 中进⾏缓存，可以通过其获取 leastLoadedNode ,即当前Node中负载压⼒最⼩的⼀个，以实现消息的尽快发出。

**2.1.3** ⽣产者参数配置补充

1. 参数设置⽅式：

2. 补充参数：

   ![image-20211120182337649](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.11.20/image-20211120182337649-7403819.png)

   ![image-20211120182411450](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.11.20/image-20211120182411450-7403853.png)

   ![image-20211120182445002](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.11.20/image-20211120182445002.png)

   ![image-20211120182526102](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.11.20/image-20211120182526102-7403929.png)

### **2.2** 消费者

#### **2.2.1** 概念⼊口

##### **2.2.1.1** 消费者、消费组

消费者从订阅的主题消费消息，消费消息的偏移量保存在Kafka的名字是 __consumer_offsets 的主题中。



消费者还可以将⾃⼰的偏移量存储到Zookeeper，需要设置offset.storage=zookeeper。

推荐使⽤**Kafka**存储消费者的偏移量。因为Zookeeper不适合⾼并发。



多个从同⼀个主题消费的消费者可以加⼊到⼀个消费组中。

消费组中的消费者共享group_id。

configs.put("group.id", "xxx");

group_id⼀般设置为应⽤的逻辑名称。⽐如多个订单处理程序组成⼀个消费组，可以设置group_id为"order_process"。

group_id通过消费者的配置指定： group.id=xxxxx

消费组均衡地给消费者分配分区，每个分区只由消费组中⼀个消费者消费。

![image-20211120183527981](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.11.20/image-20211120183527981-7404531.png)

⼀个拥有四个分区的主题，包含⼀个消费者的消费组。

此时，消费组中的消费者消费主题中的所有分区。并且没有重复的可能。

如果在消费组中添加⼀个消费者2，则每个消费者分别从两个分区接收消息。

![image-20211120183546418](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.11.20/image-20211120183546418.png)

如果消费组有四个消费者，则每个消费者可以分配到⼀个分区。

![image-20211120183612620](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.11.20/image-20211120183612620-7404576.png)

如果向消费组中添加更多的消费者，超过主题分区数量，则有⼀部分消费者就会闲置，不会接收任何消息。

![image-20211120183637506](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.11.20/image-20211120183637506-7404599.png)

向消费组添加消费者是横向扩展消费能⼒的主要⽅式。必要时，需要为主题创建⼤量分区，在负载增长时可以加⼊更多的消费者。但是不要让消费者的数量超过主题分区的数量。

![image-20211120183724695](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.11.20/image-20211120183724695-7404646.png)

除了通过增加消费者来横向扩展单个应⽤的消费能⼒之外，经常出现多个应⽤程序从同⼀个主题消费的情况。

此时，每个应⽤都可以获取到所有的消息。只要保证每个应⽤都有⾃⼰的消费组，就可以让它们获取到主题所有的消息。

横向扩展消费者和消费组不会对性能造成负⾯影响。

为每个需要获取⼀个或多个主题全部消息的应⽤创建⼀个消费组，然后向消费组添加消费者来横向扩展消费能⼒和应⽤的处理能⼒，则每个消费者只处理⼀部分消息。

##### **2.2.1.2** ⼼跳机制

![image-20211120183758025](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.11.20/image-20211120183758025.png)

消费者宕机，退出消费组，触发再平衡，重新给消费组中的消费者分配分区。

![image-20211120183824083](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.11.20/image-20211120183824083-7404705.png)

由于broker宕机，主题X的分区3宕机，此时分区3没有Leader副本，触发再平衡，消费者4没有对应的主题分区，则消费者4闲置。

![image-20211120183850226](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.11.20/image-20211120183850226-7404731.png)



Kafka 的⼼跳是 Kafka Consumer 和 Broker 之间的健康检查，只有当 Broker Coordinator 正常时，Consumer才会发送⼼跳。

Consumer 和 Rebalance 相关的 2 个配置参数：

![image-20211120183920914](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.11.20/image-20211120183920914-7404762.png)

broker 端，sessionTimeoutMs 参数

broker 处理⼼跳的逻辑在 GroupCoordinator 类中：如果⼼跳超期， broker coordinator 会把消费者从 group中移除，并触发 rebalance。

```scala
private def completeAndScheduleNextHeartbeatExpiration(group: GroupMetadata, member:

MemberMetadata) {

// complete current heartbeat expectation

member.latestHeartbeat = time.milliseconds()

val memberKey = MemberKey(member.groupId, member.memberId)

heartbeatPurgatory.checkAndComplete(memberKey)

// reschedule the next heartbeat expiration deadline

// 计算⼼跳截⽌时刻

val newHeartbeatDeadline = member.latestHeartbeat + member.sessionTimeoutMs

val delayedHeartbeat = new DelayedHeartbeat(this, group, member,

newHeartbeatDeadline, member.sessionTimeoutMs)

heartbeatPurgatory.tryCompleteElseWatch(delayedHeartbeat, Seq(memberKey))

 }

// ⼼跳过期

def onExpireHeartbeat(group: GroupMetadata, member: MemberMetadata,

heartbeatDeadline: Long) {

group.inLock {

if (!shouldKeepMemberAlive(member, heartbeatDeadline)) {

info(s"Member ${member.memberId} in group ${group.groupId} has failed,

removing it from the group")

removeMemberAndUpdateGroup(group, member)

 }

 }

 }

private def shouldKeepMemberAlive(member: MemberMetadata, heartbeatDeadline: Long) 

=

member.awaitingJoinCallback != null ||

member.awaitingSyncCallback != null ||

member.latestHeartbeat + member.sessionTimeoutMs > heartbeatDeadline
```



consumer 端：sessionTimeoutMs，rebalanceTimeoutMs 参数

如果客户端发现⼼跳超期，客户端会标记 coordinator 为不可⽤，并阻塞⼼跳线程；如果超过了 poll 消息的间隔超过了 rebalanceTimeoutMs，则 consumer 告知 broker 主动离开消费组，也会触发 rebalance

org.apache.kafka.clients.consumer.internals.AbstractCoordinator.HeartbeatThread

```scala
if (coordinatorUnknown()) {

if (findCoordinatorFuture != null ||

lookupCoordinator().failed())

// the immediate future check ensures that we backoff

properly in the case that no

// brokers are available to connect to.

AbstractCoordinator.this.wait(retryBackoffMs);

 } else if (heartbeat.sessionTimeoutExpired(now)) {

// the session timeout has expired without seeing a

successful heartbeat, so we should

// probably make sure the coordinator is still healthy.

markCoordinatorUnknown();

 } else if (heartbeat.pollTimeoutExpired(now)) {

// the poll timeout has expired, which means that the

foreground thread has stalled

// in between calls to poll(), so we explicitly leave the

group.

maybeLeaveGroup();

 } else if (!heartbeat.shouldHeartbeat(now)) {

// poll again after waiting for the retry backoff in case

the heartbeat failed or the

// coordinator disconnected

AbstractCoordinator.this.wait(retryBackoffMs);

 } else {

heartbeat.sentHeartbeat(now);

sendHeartbeatRequest().addListener(new

RequestFutureListener<Void>() {

@Override

public void onSuccess(Void value) {

synchronized (AbstractCoordinator.this) {

heartbeat.receiveHeartbeat(time.milliseconds());

 }

 }

@Override

public void onFailure(RuntimeException e) {

synchronized (AbstractCoordinator.this) {

if (e instanceof

RebalanceInProgressException) {

// it is valid to continue heartbeatingwhile the group is rebalancing. This
  // ensures that the coordinator keeps the

member in the group for as long

// as the duration of the rebalance

timeout. If we stop sending heartbeats,

// however, then the session timeout may

expire before we can rejoin.

heartbeat.receiveHeartbeat(time.milliseconds());

 } else {

heartbeat.failHeartbeat();

// wake up the thread if it's sleeping to

reschedule the heartbeat

AbstractCoordinator.this.notify();

 }

 }

 }

 });

 }
```

#### **2.2.2** 消息接收

##### **2.2.2.1** 必要参数配置

![image-20211120184158835](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.11.20/image-20211120184158835-7404921.png)

##### **2.2.2.2** 订阅

###### **2.2.2.2.1** 主题和分区

- **Topic**，Kafka⽤于分类管理消息的逻辑单元，类似与MySQL的数据库。

- **Partition**，是Kafka下数据存储的基本单元，这个是物理上的概念。同⼀个**topic**的数据，会被分散的存储到多个**partition**中，这些partition可以在同⼀台机器上，也可以是在多台机器上。优势在于：有利于⽔平扩展，避免单台机器在磁盘空间和性能上的限制，同时可以通过复制来增加数据冗余性，提⾼容灾能⼒。为了做到均匀分布，通常partition的数量通常是Broker Server数量的整数倍。

- **Consumer Group**，同样是逻辑上的概念，是**Kafka**实现单播和⼴播两种消息模型的⼿段。保证⼀个消费组获取到特定主题的全部的消息。在消费组内部，若⼲个消费者消费主题分区的消息，消费组可以保证⼀个主题的每个分区只被消费组中的⼀个消费者消费。

  ![image-20211120184340435](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.11.20/image-20211120184340435-7405022.png)

consumer 采⽤ pull 模式从 broker 中读取数据。

采⽤ pull 模式，consumer 可⾃主控制消费消息的速率， 可以⾃⼰控制消费⽅式（批量消费/逐条消费)，还可以选择不同的提交⽅式从⽽实现不同的传输语义。

consumer.subscribe("tp_demo_01,tp_demo_02")

##### **2.2.2.3** 反序列化

Kafka的broker中所有的消息都是字节数组，消费者获取到消息之后，需要先对消息进⾏反序列化处理，然后才能

交给⽤户程序消费处理。

消费者的反序列化器包括key的和value的反序列化器。

key.deserializer

value.deserializer

IntegerDeserializer

StringDeserializer

需要实现 org.apache.kafka.common.serialization.Deserializer<T> 接⼝。

消费者从订阅的主题拉取消息：

consumer.poll(3_000);

在Fetcher类中，对拉取到的消息⾸先进⾏反序列化处理。

Kafka默认提供了⼏个反序列化的实现：

org.apache.kafka.common.serialization 包下包含了这⼏个实现：

**2.2.2.3.1** ⾃定义反序列化

⾃定义反序列化类，需要实现 org.apache.kafka.common.serialization.Deserializer<T> 接⼝。

com.lagou.kafka.demo.deserializer.UserDeserializer

```java
com.lagou.kafka.demo.consumer.MyConsumer

package com.lagou.kafka.demo.deserializer;

import com.lagou.kafka.demo.entity.User;

import org.apache.kafka.common.serialization.Deserializer;

import java.nio.ByteBuffer;

import java.util.Map;

public class UserDeserializer implements Deserializer<User> {

@Override

public void configure(Map<String, ?> configs, boolean isKey) {

 }

@Override

public User deserialize(String topic, byte[] data) {

ByteBuffer allocate = ByteBuffer.allocate(data.length);

allocate.put(data);

allocate.flip();

int userId = allocate.getInt();

int length = allocate.getInt();

System.out.println(length);

String username = new String(data, 8, length);

return new User(userId, username);

 }

@Override

public void close() {

 }

} 
```



```java
package com.lagou.kafka.demo.consumer;

import com.lagou.kafka.demo.deserializer.UserDeserializer;

import com.lagou.kafka.demo.entity.User; 
import org.apache.kafka.clients.consumer.ConsumerConfig;

import org.apache.kafka.clients.consumer.ConsumerRecord;

import org.apache.kafka.clients.consumer.ConsumerRecords;

import org.apache.kafka.clients.consumer.KafkaConsumer;

import org.apache.kafka.common.serialization.StringDeserializer;

import java.util.Collections;

import java.util.HashMap;

import java.util.Map;

import java.util.function.Consumer;

public class MyConsumer {

public static void main(String[] args) {

Map<String, Object> configs = new HashMap<>();

configs.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "node1:9092");

configs.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG,

StringDeserializer.class);

configs.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG,

UserDeserializer.class);

configs.put(ConsumerConfig.GROUP_ID_CONFIG, "consumer1");

configs.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");

configs.put(ConsumerConfig.CLIENT_ID_CONFIG, "con1");

KafkaConsumer<String, User> consumer = new KafkaConsumer<String, User> 

(configs);

consumer.subscribe(Collections.singleton("tp_user_01"));

ConsumerRecords<String, User> records = consumer.poll(Long.MAX_VALUE);

records.forEach(new Consumer<ConsumerRecord<String, User>>() {

@Override

public void accept(ConsumerRecord<String, User> record) {

System.out.println(record.value());

 }

 });

// 关闭消费者

consumer.close();

 }

} 
```



##### **2.2.2.4** 位移提交

1. Consumer需要向Kafka记录⾃⼰的位移数据，这个汇报过程称为 提交位移(Committing Offsets)
2. Consumer 需要为分配给它的每个分区提交各⾃的位移数据
3. 位移提交的由Consumer端负责的，Kafka只负责保管。__consumer_offsets
4. 位移提交分为⾃动提交和⼿动提交
5. 位移提交分为同步提交和异步提交

###### **2.2.2.4.1** ⾃动提交

Kafka Consumer 后台提交

开启⾃动提交： enable.auto.commit=true

配置⾃动提交间隔：Consumer端： auto.commit.interval.ms ，默认 5s

⾃动提交位移的顺序

配置 enable.auto.commit = true

Kafka会保证在开始调⽤poll⽅法时，提交上次poll返回的所有消息

因此⾃动提交不会出现消息丢失，但会 重复消费

重复消费举例

Consumer 每 5s 提交 offset

假设提交 offset 后的 3s 发⽣了 Rebalance

Rebalance 之后的所有 Consumer 从上⼀次提交的 offset 处继续消费

因此 Rebalance 发⽣前 3s 的消息会被重复消费

Map<String, Object> configs = new HashMap<>();

configs.put("bootstrap.servers", "node1:9092");

configs.put("group.id", "mygrp");

// 设置偏移量⾃动提交。⾃动提交是默认值。这⾥做示例。

configs.put("enable.auto.commit", "true");

// 偏移量⾃动提交的时间间隔

configs.put("auto.commit.interval.ms", "3000");

configs.put("key.deserializer", StringDeserializer.class);

configs.put("value.deserializer", StringDeserializer.class);

KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(configs);

consumer.subscribe(Collections.singleton("tp_demo_01"));

while (true) {

ConsumerRecords<String, String> records = consumer.poll(100);

for (ConsumerRecord<String, String> record : records) {

System.out.println(record.topic()

\+ "\t" + record.partition()

\+ "\t" + record.offset()

\+ "\t" + record.key()

\+ "\t" + record.value());

 }

} 

###### **2.2.2.4.2** 异步提交

使⽤ KafkaConsumer#commitSync()：会提交 KafkaConsumer#poll() 返回的最新 offset

该⽅法为同步操作，等待直到 offset 被成功提交才返回

commitSync 在处理完所有消息之后

⼿动同步提交可以控制offset提交的时机和频率

⼿动同步提交会：

调⽤ commitSync 时，Consumer 处于阻塞状态，直到 Broker 返回结果

会影响 TPS

可以选择拉⻓提交间隔，但有以下问题

会导致 Consumer 的提交频率下降

Consumer 重启后，会有更多的消息被消费

异步提交

KafkaConsumer#commitAsync()

commitAsync出现问题不会⾃动重试

处理⽅式：

while (true) {

ConsumerRecords<String, String> records =

consumer.poll(Duration.ofSeconds(1));

process(records); // 处理消息

try {

consumer.commitSync();

 } catch (CommitFailedException e) {

handle(e); // 处理提交失败异常

 }

} 



while (true) {

ConsumerRecords<String, String> records = consumer.poll(3_000);

process(records); // 处理消息

consumer.commitAsync((offsets, exception) -> {

if (exception != null) {

handle(exception);

 }

 });

} 

项⽬ 细节

API public void assign(Collection<TopicPartition> partitions)

说明

给当前消费者⼿动分配⼀系列主题分区。

⼿动分配分区不⽀持增量分配，如果先前有分配分区，则该操作会覆盖之前的分配。

如果给出的主题分区是空的，则等价于调⽤unsubscribe⽅法。

⼿动分配主题分区的⽅法不使⽤消费组管理功能。当消费组成员变了，或者集群或主题的元数据改变了，

不会触发分区分配的再平衡。

⼿动分区分配assign(Collection)不能和⾃动分区分配subscribe(Collection,

ConsumerRebalanceListener)⼀起使⽤。

如果启⽤了⾃动提交偏移量，则在新的分区分配替换旧的分区分配之前，会对旧的分区分配中的消费偏移

量进⾏异步提交。

API public Set<TopicPartition> assignment()

说

获取给当前消费者分配的分区集合。如果订阅是通过调⽤assign⽅法直接分配主题分区，则返回相同的集

##### **2.2.2.5** 消费者位移管理

Kafka中，消费者根据消息的位移顺序消费消息。

消费者的位移由消费者管理，可以存储于zookeeper中，也可以存储于Kafka主题__consumer_offsets中。

Kafka提供了消费者API，让消费者可以管理⾃⼰的位移。

API如下：KafkaConsumer<K, V>

try {

while(true) {

ConsumerRecords<String, String> records =

consumer.poll(Duration.ofSeconds(1));

process(records); // 处理消息

commitAysnc(); // 使⽤异步提交规避阻塞

 }

} catch(Exception e) {

handle(e); // 处理异常

} finally {

try {

consumer.commitSync(); // 最后⼀次提交使⽤同步阻塞式提交

 } finally {

consumer.close();

 }

} 

明 合。如果使⽤了主题订阅，该⽅法返回当前分配给该消费者的主题分区集合。如果分区订阅还没开始进⾏

分区分配，或者正在重新分配分区，则会返回none。

API public Map<String, List<PartitionInfo>> listTopics()

说明 

获取对⽤户授权的所有主题分区元数据。该⽅法会对服务器发起远程调⽤。

API public List<PartitionInfo> partitionsFor(String topic)

说明 

获取指定主题的分区元数据。如果当前消费者没有关于该主题的元数据，就会对服务器发起远程调⽤。

API public Map<TopicPartition, Long> beginningOffsets(Collection<TopicPartition> partitions)

说明

对于给定的主题分区，列出它们第⼀个消息的偏移量。

注意，如果指定的分区不存在，该⽅法可能会永远阻塞。

该⽅法不改变分区的当前消费者偏移量。

API public void seekToEnd(Collection<TopicPartition> partitions)

说明

将偏移量移动到每个给定分区的最后⼀个。

该⽅法延迟执⾏，只有当调⽤过poll⽅法或position⽅法之后才可以使⽤。

如果没有指定分区，则将当前消费者分配的所有分区的消费者偏移量移动到最后。

如果设置了隔离级别为：isolation.level=read_committed，则会将分区的消费偏移量移动到最后⼀个稳

定的偏移量，即下⼀个要消费的消息现在还是未提交状态的事务消息。

API public void seek(TopicPartition partition, long offset)

说明

将给定主题分区的消费偏移量移动到指定的偏移量，即当前消费者下⼀条要消费的消息偏移量。

若该⽅法多次调⽤，则最后⼀次的覆盖前⾯的。

如果在消费中间随意使⽤，可能会丢失数据。

API public long position(TopicPartition partition)

说明 

检查指定主题分区的消费偏移量

API public void seekToBeginning(Collection<TopicPartition> partitions)

说明

将给定每个分区的消费者偏移量移动到它们的起始偏移量。该⽅法懒执⾏，只有当调⽤过poll⽅法或

position⽅法之后才会执⾏。如果没有提供分区，则将所有分配给当前消费者的分区消费偏移量移动到起

始偏移量。

\1. 准备数据2. API实战

\# ⽣成消息⽂件

[root@node1 ~]# for i in `seq 60`; do echo "hello lagou $i" >> nm.txt; done

\# 创建主题，三个分区，每个分区⼀个副本

[root@node1 ~]# kafka-topics.sh --zookeeper node1:2181/myKafka --create --topic

tp_demo_01 --partitions 3 --replication-factor 1

\# 将消息⽣产到主题中

[root@node1 ~]# kafka-console-producer.sh --broker-list node1:9092 --topic tp_demo_01 <

nm.txt



package com.lagou.kafka.demo.consumer;

import org.apache.kafka.clients.consumer.ConsumerConfig;

import org.apache.kafka.clients.consumer.KafkaConsumer;

import org.apache.kafka.common.Node;

import org.apache.kafka.common.PartitionInfo;

import org.apache.kafka.common.TopicPartition;

import org.apache.kafka.common.serialization.StringDeserializer;

import java.util.*;

/**

\* # ⽣成消息⽂件

\* [root@node1 ~]# for i in `seq 60`; do echo "hello lagou $i" >> nm.txt; done

\* # 创建主题，三个分区，每个分区⼀个副本

\* [root@node1 ~]# kafka-topics.sh --zookeeper node1:2181/myKafka --create --topic

tp_demo_01 --partitions 3 --replication-factor 1

\* # 将消息⽣产到主题中

\* [root@node1 ~]# kafka-console-producer.sh --broker-list node1:9092 --topic

tp_demo_01 < nm.txt

*

\* 消费者位移管理

*/

public class MyConsumerMgr1 {

public static void main(String[] args) {

Map<String, Object> configs = new HashMap<>();

configs.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "node1:9092");

configs.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG,

StringDeserializer.class);

configs.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG,

StringDeserializer.class);

KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String> 

(configs);

/**

* 给当前消费者⼿动分配⼀系列主题分区。

\* ⼿动分配分区不⽀持增量分配，如果先前有分配分区，则该操作会覆盖之前的分配。

\* 如果给出的主题分区是空的，则等价于调⽤unsubscribe⽅法。

\* ⼿动分配主题分区的⽅法不使⽤消费组管理功能。当消费组成员变了，或者集群或主题的元数据改变

了，不会触发分区分配的再平衡。

*

\* ⼿动分区分配assign(Collection)不能和⾃动分区分配subscribe(Collection,

ConsumerRebalanceListener)⼀起使⽤。

\* 如果启⽤了⾃动提交偏移量，则在新的分区分配替换旧的分区分配之前，会对旧的分区分配中的消费

偏移量进⾏异步提交。

*

*/

// consumer.assign(Arrays.asList(new TopicPartition("tp_demo_01", 0)));

//

// Set<TopicPartition> assignment = consumer.assignment();

// for (TopicPartition topicPartition : assignment) {

// System.out.println(topicPartition);

// }

// 获取对⽤户授权的所有主题分区元数据。该⽅法会对服务器发起远程调⽤。

// Map<String, List<PartitionInfo>> stringListMap = consumer.listTopics();

//

// stringListMap.forEach((k, v) -> {

// System.out.println("主题：" + k);

// v.forEach(info -> {

// System.out.println(info);

// });

// });

// Set<String> strings = consumer.listTopics().keySet();

//

// strings.forEach(topicName -> {

// System.out.println(topicName);

// });

// List<PartitionInfo> partitionInfos = consumer.partitionsFor("tp_demo_01");

// for (PartitionInfo partitionInfo : partitionInfos) {

// Node leader = partitionInfo.leader();

// System.out.println(leader);

// System.out.println(partitionInfo);

// // 当前分区在线副本

// Node[] nodes = partitionInfo.inSyncReplicas();

// // 当前分区下线副本

// Node[] nodes1 = partitionInfo.offlineReplicas();

// }

// ⼿动分配主题分区给当前消费者

consumer.assign(Arrays.asList(

new TopicPartition("tp_demo_01", 0),

new TopicPartition("tp_demo_01", 1),

new TopicPartition("tp_demo_01", 2)

 ));

// 列出当前主题分配的所有主题分区

// Set<TopicPartition> assignment = consumer.assignment();

// assignment.forEach(k -> {

// System.out.println(k);

// });

// 对于给定的主题分区，列出它们第⼀个消息的偏移量。

// 注意，如果指定的分区不存在，该⽅法可能会永远阻塞。

// 该⽅法不改变分区的当前消费者偏移量。

// Map<TopicPartition, Long> topicPartitionLongMap =

consumer.beginningOffsets(consumer.assignment());

//

// topicPartitionLongMap.forEach((k, v) -> {

// System.out.println("主题：" + k.topic() + "\t分区：" + k.partition() +

"偏移量\t" + v);

// });

// 将偏移量移动到每个给定分区的最后⼀个。

// 该⽅法延迟执⾏，只有当调⽤过poll⽅法或position⽅法之后才可以使⽤。

// 如果没有指定分区，则将当前消费者分配的所有分区的消费者偏移量移动到最后。

// 如果设置了隔离级别为：isolation.level=read_committed，则会将分区的消费偏移量移动到

// 最后⼀个稳定的偏移量，即下⼀个要消费的消息现在还是未提交状态的事务消息。

// consumer.seekToEnd(consumer.assignment());

// 将给定主题分区的消费偏移量移动到指定的偏移量，即当前消费者下⼀条要消费的消息偏移量。

// 若该⽅法多次调⽤，则最后⼀次的覆盖前⾯的。

// 如果在消费中间随意使⽤，可能会丢失数据。

// consumer.seek(new TopicPartition("tp_demo_01", 1), 10);

//

// // 检查指定主题分区的消费偏移量

// long position = consumer.position(new TopicPartition("tp_demo_01", 1));

// System.out.println(position);

consumer.seekToEnd(Arrays.asList(new TopicPartition("tp_demo_01", 1)));

// 检查指定主题分区的消费偏移量

long position = consumer.position(new TopicPartition("tp_demo_01", 1));

System.out.println(position);

// 关闭⽣产者

consumer.close();

 }

}

##### **2.2.2.6** 再均衡

重平衡可以说是kafka为⼈诟病最多的⼀个点了。

重平衡其实就是⼀个协议，它规定了如何让消费者组下的所有消费者来分配topic中的每⼀个分区。⽐如⼀个topic

有100个分区，⼀个消费者组内有20个消费者，在协调者的控制下让组内每⼀个消费者分配到5个分区，这个分配的过

程就是重平衡。

重平衡的触发条件主要有三个：

\1. 消费者组内成员发⽣变更，这个变更包括了增加和减少消费者，⽐如消费者宕机退出消费组。

\2. 主题的分区数发⽣变更，kafka⽬前只⽀持增加分区，当增加的时候就会触发重平衡

\3. 订阅的主题发⽣变化，当消费者组使⽤正则表达式订阅主题，⽽恰好⼜新建了对应的主题，就会触发重平衡

消费者宕机，退出消费组，触发再平衡，重新给消费组中的消费者分配分区。由于broker宕机，主题X的分区3宕机，此时分区3没有Leader副本，触发再平衡，消费者4没有对应的主题分区，

则消费者4闲置。主题增加分区，需要主题分区和消费组进⾏再均衡。

由于使⽤正则表达式订阅主题，当增加的主题匹配正则表达式的时候，也要进⾏再均衡。为什么说重平衡为⼈诟病呢？因为重平衡过程中，消费者⽆法从**kafka**消费消息，这对**kafka**的**TPS**影响极⼤，⽽

如果**kafka**集内节点较多，⽐如数百个，那重平衡可能会耗时极多。数分钟到数⼩时都有可能，⽽这段时间**kafka**基本

处于不可⽤状态。所以在实际环境中，应该尽量避免重平衡发⽣。

避免重平衡

要说完全避免重平衡，是不可能，因为你⽆法完全保证消费者不会故障。⽽消费者故障其实也是最常⻅的引发重

平衡的地⽅，所以我们需要保证尽⼒避免消费者故障。

⽽其他⼏种触发重平衡的⽅式，增加分区，或是增加订阅的主题，抑或是增加消费者，更多的是主动控制。

如果消费者真正挂掉了，就没办法了，但实际中，会有⼀些情况，**kafka**错误地认为⼀个正常的消费者已经挂掉

了，我们要的就是避免这样的情况出现。⾸先要知道哪些情况会出现错误判断挂掉的情况。

在分布式系统中，通常是通过⼼跳来维持分布式系统的，kafka也不例外。

在分布式系统中，由于⽹络问题你不清楚没接收到⼼跳，是因为对⽅真正挂了还是只是因为负载过重没来得及发

⽣⼼跳或是⽹络堵塞。所以⼀般会约定⼀个时间，超时即判定对⽅挂了。⽽在**kafka**消费者场景中，

**session.timout.ms**参数就是规定这个超时时间是多少。

还有⼀个参数，**heartbeat.interval.ms**，这个参数控制发送⼼跳的频率，频率越⾼越不容易被误判，但也会消

耗更多资源。

此外，还有最后⼀个参数，**max.poll.interval.ms**，消费者poll数据后，需要⼀些处理，再进⾏拉取。如果两次

拉取时间间隔超过这个参数设置的值，那么消费者就会被踢出消费者组。也就是说，拉取，然后处理，这个处理的时

间不能超过 max.poll.interval.ms 这个参数的值。这个参数的默认值是5分钟，⽽如果消费者接收到数据后会执⾏

耗时的操作，则应该将其设置得⼤⼀些。

三个参数，

session.timout.ms控制⼼跳超时时间，

heartbeat.interval.ms控制⼼跳发送频率，

max.poll.interval.ms控制poll的间隔。

这⾥给出⼀个相对较为合理的配置，如下：

session.timout.ms：设置为6s

heartbeat.interval.ms：设置2s

max.poll.interval.ms：推荐为消费者处理消息最⻓耗时再加1分钟

##### **2.2.2.7** 消费者拦截器

消费者在拉取了分区消息之后，要⾸先经过反序列化器对key和value进⾏反序列化处理。

处理完之后，如果消费端设置了拦截器，则需要经过拦截器的处理之后，才能返回给消费者应⽤程序进⾏处理。

消费端定义消息拦截器，需要实现 org.apache.kafka.clients.consumer.ConsumerInterceptor<K, V> 接

⼝。

\1. ⼀个可插拔接⼝，允许拦截甚⾄更改消费者接收到的消息。⾸要的⽤例在于将第三⽅组件引⼊消费者应⽤程

序，⽤于定制的监控、⽇志处理等。2. 该接⼝的实现类通过configre⽅法获取消费者配置的属性，如果消费者配置中没有指定clientID，还可以获取

KafkaConsumer⽣成的clientId。获取的这个配置是跟其他拦截器共享的，需要保证不会在各个拦截器之间

产⽣冲突。

\3. ConsumerInterceptor⽅法抛出的异常会被捕获、记录，但是不会向下传播。如果⽤户配置了错误的key或

value类型参数，消费者不会抛出异常，⽽仅仅是记录下来。

\4. ConsumerInterceptor回调发⽣在org.apache.kafka.clients.consumer.KafkaConsumer#poll(long)⽅法同

⼀个线程。

该接⼝中有如下⽅法：

代码实现：

package org.apache.kafka.clients.consumer;

import org.apache.kafka.common.Configurable;

import org.apache.kafka.common.TopicPartition;

import java.util.Map;

public interface ConsumerInterceptor<K, V> extends Configurable {

/**

*

\* 该⽅法在poll⽅法返回之前调⽤。调⽤结束后poll⽅法就返回消息了。

*

\* 该⽅法可以修改消费者消息，返回新的消息。拦截器可以过滤收到的消息或⽣成新的消息。

\* 如果有多个拦截器，则该⽅法按照KafkaConsumer的configs中配置的顺序调⽤。

*

\* @param records 由上个拦截器返回的由客户端消费的消息。

*/

public ConsumerRecords<K, V> onConsume(ConsumerRecords<K, V> records);

/**

\* 当消费者提交偏移量时，调⽤该⽅法。

\* 该⽅法抛出的任何异常调⽤者都会忽略。

*/

public void onCommit(Map<TopicPartition, OffsetAndMetadata> offsets);

public void close();

} 



package com.lagou.kafka.demo.consumer;

import org.apache.kafka.clients.consumer.ConsumerConfig;

import org.apache.kafka.clients.consumer.ConsumerRecords;

import org.apache.kafka.clients.consumer.KafkaConsumer; 

import java.util.Collections;

import java.util.Properties;

public class MyConsumer {

public static void main(String[] args) {

Properties props = new Properties();

props.setProperty(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "node1:9092");

props.setProperty(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG,

"org.apache.kafka.common.serialization.StringDeserializer");

props.setProperty(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG,

"org.apache.kafka.common.serialization.StringDeserializer");

props.setProperty(ConsumerConfig.GROUP_ID_CONFIG, "mygrp");

// props.setProperty(ConsumerConfig.CLIENT_ID_CONFIG, "myclient");

// 如果在kafka中找不到当前消费者的偏移量，则设置为最旧的

props.setProperty(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");

// 配置拦截器

// One -> Two -> Three，接收消息和发送偏移量确认都是这个顺序

props.setProperty(ConsumerConfig.INTERCEPTOR_CLASSES_CONFIG,

"com.lagou.kafka.demo.interceptor.OneInterceptor" +

",com.lagou.kafka.demo.interceptor.TwoInterceptor" +

",com.lagou.kafka.demo.interceptor.ThreeInterceptor"

 );

KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String> 

(props);

// 订阅主题

consumer.subscribe(Collections.singleton("tp_demo_01"));

while (true) {

final ConsumerRecords<String, String> records = consumer.poll(3_000);

records.forEach(record -> {

System.out.println(record.topic()

\+ "\t" + record.partition()

\+ "\t" + record.offset()

\+ "\t" + record.key()

\+ "\t" + record.value());

 });

// consumer.commitAsync();

// consumer.commitSync();

 }

// consumer.close();

 }

} 



package com.lagou.kafka.demo.interceptor;

import org.apache.kafka.clients.consumer.ConsumerInterceptor;

import org.apache.kafka.clients.consumer.ConsumerRecords;

import org.apache.kafka.clients.consumer.OffsetAndMetadata;

import org.apache.kafka.common.TopicPartition;

import java.util.Map;

public class OneInterceptor implements ConsumerInterceptor<String, String> {

@Override

public ConsumerRecords<String, String> onConsume(ConsumerRecords<String, String>

records) {

// poll⽅法返回结果之前最后要调⽤的⽅法

System.out.println("One -- 开始");

// 消息不做处理，直接返回

return records;

 }

@Override

public void onCommit(Map<TopicPartition, OffsetAndMetadata> offsets) {

// 消费者提交偏移量的时候，经过该⽅法

System.out.println("One -- 结束");

 }

@Override

public void close() {

// ⽤于关闭该拦截器⽤到的资源，如打开的⽂件，连接的数据库等

 }

@Override

public void configure(Map<String, ?> configs) {

// ⽤于获取消费者的设置参数

configs.forEach((k, v) -> {

System.out.println(k + "\t" + v);

 });

 }

} 



package com.lagou.kafka.demo.interceptor;

import org.apache.kafka.clients.consumer.ConsumerInterceptor;

import org.apache.kafka.clients.consumer.ConsumerRecords; 

import org.apache.kafka.clients.consumer.OffsetAndMetadata;

import org.apache.kafka.common.TopicPartition;

import java.util.Map;

public class TwoInterceptor implements ConsumerInterceptor<String, String> {

@Override

public ConsumerRecords<String, String> onConsume(ConsumerRecords<String, String>

records) {

// poll⽅法返回结果之前最后要调⽤的⽅法

System.out.println("Two -- 开始");

// 消息不做处理，直接返回

return records;

 }

@Override

public void onCommit(Map<TopicPartition, OffsetAndMetadata> offsets) {

// 消费者提交偏移量的时候，经过该⽅法

System.out.println("Two -- 结束");

 }

@Override

public void close() {

// ⽤于关闭该拦截器⽤到的资源，如打开的⽂件，连接的数据库等

 }

@Override

public void configure(Map<String, ?> configs) {

// ⽤于获取消费者的设置参数

configs.forEach((k, v) -> {

System.out.println(k + "\t" + v);

 });

 }

} 



package com.lagou.kafka.demo.interceptor;

import org.apache.kafka.clients.consumer.ConsumerInterceptor;

import org.apache.kafka.clients.consumer.ConsumerRecords;

import org.apache.kafka.clients.consumer.OffsetAndMetadata;

import org.apache.kafka.common.TopicPartition;

import java.util.Map;

public class ThreeInterceptor implements ConsumerInterceptor<String, String> {

@Override

配置项 说明

bootstrap.servers

建⽴到Kafka集群的初始连接⽤到的host/port列表。

客户端会使⽤这⾥指定的所有的host/port来建⽴初始连接。

这个配置仅会影响发现集群所有节点的初始连接。

形式：host1:port1,host2:port2...

这个配置中不需要包含集群中所有的节点信息。

最好不要配置⼀个，以免配置的这个节点宕机的时候连不上。

group.id

⽤于定义当前消费者所属的消费组的唯⼀字符串。

如果使⽤了消费组的功能 subscribe(topic) ，

或使⽤了基于Kafka的偏移量管理机制，则应该配置group.id。

auto.commit.interval.ms 

如果设置了 enable.auto.commit 的值为true，

则该值定义了消费者偏移量向Kafka提交的频率。

如果Kafka中没有初始偏移量或当前偏移量在服务器中不存在

（⽐如数据被删掉了）：

**2.2.2.8** 消费者参数配置补充

public ConsumerRecords<String, String> onConsume(ConsumerRecords<String, String>

records) {

// poll⽅法返回结果之前最后要调⽤的⽅法

System.out.println("Three -- 开始");

// 消息不做处理，直接返回

return records;

 }

@Override

public void onCommit(Map<TopicPartition, OffsetAndMetadata> offsets) {

// 消费者提交偏移量的时候，经过该⽅法

System.out.println("Three -- 结束");

 }

@Override

public void close() {

// ⽤于关闭该拦截器⽤到的资源，如打开的⽂件，连接的数据库等

 }

@Override

public void configure(Map<String, ?> configs) {

// ⽤于获取消费者的设置参数

configs.forEach((k, v) -> {

System.out.println(k + "\t" + v);

 });

 }

}

auto.offset.reset earliest：⾃动重置偏移量到最早的偏移量。

latest：⾃动重置偏移量到最后⼀个

none：如果没有找到该消费组以前的偏移量没有找到，就抛异常。

其他值：向消费者抛异常。

fetch.min.bytes

服务器对每个拉取消息的请求返回的数据量最⼩值。

如果数据量达不到这个值，请求等待，以让更多的数据累积，

达到这个值之后响应请求。

默认设置是1个字节，表示只要有⼀个字节的数据，

就⽴即响应请求，或者在没有数据的时候请求超时。

将该值设置为⼤⼀点⼉的数字，会让服务器等待稍微

⻓⼀点⼉的时间以累积数据。

如此则可以提⾼服务器的吞吐量，代价是额外的延迟时间。

fetch.max.wait.ms

如果服务器端的数据量达不到 fetch.min.bytes 的话，

服务器端不能⽴即响应请求。

该时间⽤于配置服务器端阻塞请求的最⼤时⻓。

fetch.max.bytes

服务器给单个拉取请求返回的最⼤数据量。

消费者批量拉取消息，如果第⼀个⾮空消息批次的值⽐该值⼤，

消息批也会返回，以让消费者可以接着进⾏。

即该配置并不是绝对的最⼤值。

broker可以接收的消息批最⼤值通过

message.max.bytes (broker配置) 

或 max.message.bytes (主题配置)来指定。

需要注意的是，消费者⼀般会并发拉取请求。

enable.auto.commit 如果设置为true，则消费者的偏移量会周期性地在后台提交。

connections.max.idle.ms 在这个时间之后关闭空闲的连接。

check.crcs

⾃动计算被消费的消息的CRC32校验值。

可以确保在传输过程中或磁盘存储过程中消息没有被破坏。

它会增加额外的负载，在追求极致性能的场合禁⽤。

exclude.internal.topics 

是否内部主题应该暴露给消费者。如果该条⽬设置为true，

则只能先订阅再拉取。

isolation.level

控制如何读取事务消息。

如果设置了 read_committed ，消费者的poll()⽅法只会

返回已经提交的事务消息。

如果设置了 read_uncommitted (默认值)，

消费者的poll⽅法返回所有的消息，即使是已经取消的事务消息。

⾮事务消息以上两种情况都返回。

消息总是以偏移量的顺序返回。

read_committed 只能返回到达LSO的消息。

在LSO之后出现的消息只能等待相关的事务提交之后才能看到。

结果， read_committed 模式，如果有为提交的事务，

消费者不能读取到直到HW的消息。

read_committed 的seekToEnd⽅法返回LSO。

heartbeat.interval.ms

当使⽤消费组的时候，该条⽬指定消费者向消费者协调器

发送⼼跳的时间间隔。

⼼跳是为了确保消费者会话的活跃状态，

同时在消费者加⼊或离开消费组的时候⽅便进⾏再平衡。

该条⽬的值必须⼩于 session.timeout.ms ，也不应该⾼于 session.timeout.ms 的1/3。

可以将其调整得更⼩，以控制正常重新平衡的预期时间。

session.timeout.ms

当使⽤Kafka的消费组的时候，消费者周期性地向broker发送⼼跳数据，

表明⾃⼰的存在。

如果经过该超时时间还没有收到消费者的⼼跳，

则broker将消费者从消费组移除，并启动再平衡。

该值必须在broker配置 group.min.session.timeout.ms 和 group.max.session.timeout.ms 之间。

max.poll.records ⼀次调⽤poll()⽅法返回的记录最⼤数量。

max.poll.interval.ms

使⽤消费组的时候调⽤poll()⽅法的时间间隔。

该条⽬指定了消费者调⽤poll()⽅法的最⼤时间间隔。

如果在此时间内消费者没有调⽤poll()⽅法，

则broker认为消费者失败，触发再平衡，

将分区分配给消费组中其他消费者。

对每个分区，服务器返回的最⼤数量。消费者按批次拉取数据。

如果⾮空分区的第⼀个记录⼤于这个值，批处理依然可以返回，max.partition.fetch.bytes 以保证消费者可以进⾏下去。

broker接收批的⼤⼩由 message.max.bytes （broker参数）或 max.message.bytes （主题参数）指定。

fetch.max.bytes ⽤于限制消费者单次请求的数据量。

send.buffer.bytes 

⽤于TCP发送数据时使⽤的缓冲⼤⼩（SO_SNDBUF），

-1表示使⽤OS默认的缓冲区⼤⼩。

retry.backoff.ms

在发⽣失败的时候如果需要重试，则该配置表示客户端

等待多⻓时间再发起重试。

该时间的存在避免了密集循环。

request.timeout.ms 

客户端等待服务端响应的最⼤时间。如果该时间超时，

则客户端要么重新发起请求，要么如果重试耗尽，请求失败。

reconnect.backoff.ms 

重新连接主机的等待时间。避免了重连的密集循环。

该等待时间应⽤于该客户端到broker的所有连接。

reconnect.backoff.max.ms

重新连接到反复连接失败的broker时要等待的最⻓时间

（以毫秒为单位）。

如果提供此选项，则对于每个连续的连接失败，

每台主机的退避将成倍增加，直⾄达到此最⼤值。

在计算退避增量之后，添加20％的随机抖动以避免连接⻛暴。

receive.buffer.bytes TCP连接接收数据的缓存（SO_RCVBUF）。

-1表示使⽤操作系统的默认值。

partition.assignment.strategy 当使⽤消费组的时候，分区分配策略的类名。

metrics.sample.window.ms 计算指标样本的时间窗⼝。

metrics.recording.level 指标的最⾼记录级别。

metrics.num.samples ⽤于计算指标⽽维护的样本数量

interceptor.classes

拦截器类的列表。默认没有拦截器

拦截器是消费者的拦截器，该拦截器需要实现 org.apache.kafka.clients.consumer

.ConsumerInterceptor 接⼝。

拦截器可⽤于对消费者接收到的消息进⾏拦截处理。

**2.2.3** 消费组管理

⼀、消费者组 **(Consumer Group)**

**1** 什么是消费者组

consumer group是kafka提供的可扩展且具有容错性的消费者机制。

三个特性：

\1. 消费组有⼀个或多个消费者，消费者可以是⼀个进程，也可以是⼀个线程

\2. group.id是⼀个字符串，唯⼀标识⼀个消费组

\3. 消费组订阅的主题每个分区只能分配给消费组⼀个消费者。

**2** 消费者位移**(consumer position)**

消费者在消费的过程中记录已消费的数据，即消费位移（offset）信息。

每个消费组保存⾃⼰的位移信息，那么只需要简单的⼀个整数表示位置就够了；同时可以引⼊checkpoint机制定

期持久化。**3** 位移管理**(offset management)**

**3.1** ⾃动**VS**⼿动

Kafka默认定期⾃动提交位移( enable.auto.commit = true )，也⼿动提交位移。另外kafka会定期把group消

费情况保存起来，做成⼀个offset map，如下图所示：

**3.2** 位移提交

位移是提交到Kafka中的 __consumer_offsets 主题。 __consumer_offsets 中的消息保存了每个消费组某⼀时

刻提交的offset信息。

上图中，标出来的，表示消费组为 test-consumer-group ，消费的主题为 __consumer_offsets ，消费的分区

是4，偏移量为5。

__consumers_offsets 主题配置了compact策略，使得它总是能够保存最新的位移信息，既控制了该topic总体

的⽇志容量，也能实现保存最新offset的⽬的。

**4** 再谈再均衡

[root@node1 __consumer_offsets-0]# kafka-console-consumer.sh --topic __consumer_offsets

--bootstrap-server node1:9092 --formatter

"kafka.coordinator.group.GroupMetadataManager\$OffsetsMessageFormatter" --

consumer.config /opt/kafka_2.12-1.0.2/config/consumer.properties --from-beginning |

head

1**4.1** 什么是再均衡？

再均衡（Rebalance）本质上是⼀种协议，规定了⼀个消费组中所有消费者如何达成⼀致来分配订阅主题的每个

分区。

⽐如某个消费组有20个消费组，订阅了⼀个具有100个分区的主题。正常情况下，Kafka平均会为每个消费者分配

5个分区。这个分配的过程就叫再均衡。

**4.2** 什么时候再均衡？

再均衡的触发条件：

\1. 组成员发⽣变更(新消费者加⼊消费组组、已有消费者主动离开或崩溃了)

\2. 订阅主题数发⽣变更。如果正则表达式进⾏订阅，则新建匹配正则表达式的主题触发再均衡。

\3. 订阅主题的分区数发⽣变更

**4.3** 如何进⾏组内分区分配？

三种分配策略：RangeAssignor和RoundRobinAssignor以及StickyAssignor。后⾯讲。

**4.4** 谁来执⾏再均衡和消费组管理？

Kafka提供了⼀个⻆⾊：Group Coordinator来执⾏对于消费组的管理。

Group Coordinator——每个消费组分配⼀个消费组协调器⽤于组管理和位移管理。当消费组的第⼀个消费者启

动的时候，它会去和Kafka Broker确定谁是它们组的组协调器。之后该消费组内所有消费者和该组协调器协调通信。

**4.5** 如何确定**coordinator**？

两步：

\1. 确定消费组位移信息写⼊ __consumers_offsets 的哪个分区。具体计算公式：

__consumers_offsets partition# = Math.abs(groupId.hashCode() %

groupMetadataTopicPartitionCount) 注意：groupMetadataTopicPartitionCount

由 offsets.topic.num.partitions 指定，默认是50个分区。

\2. 该分区leader所在的broker就是组协调器。

**4.6 Rebalance Generation**

它表示Rebalance之后主题分区到消费组中消费者映射关系的⼀个版本，主要是⽤于保护消费组，隔离⽆效偏移

量提交的。如上⼀个版本的消费者⽆法提交位移到新版本的消费组中，因为映射关系变了，你消费的或许已经不是原

来的那个分区了。每次group进⾏Rebalance之后，Generation号都会加1，表示消费组和分区的映射关系到了⼀个新

版本，如下图所示： Generation 1时group有3个成员，随后成员2退出组，消费组协调器触发Rebalance，消费组进

⼊Generation 2，之后成员4加⼊，再次触发Rebalance，消费组进⼊Generation 3.**4.7** 协议**(protocol)**

kafka提供了5个协议来处理与消费组协调相关的问题：

Heartbeat请求：consumer需要定期给组协调器发送⼼跳来表明⾃⼰还活着

LeaveGroup请求：主动告诉组协调器我要离开消费组

SyncGroup请求：消费组Leader把分配⽅案告诉组内所有成员

JoinGroup请求：成员请求加⼊组

DescribeGroup请求：显示组的所有信息，包括成员信息，协议名称，分配⽅案，订阅信息等。通常该请求

是给管理员使⽤

组协调器在再均衡的时候主要⽤到了前⾯4种请求。

**4.8 liveness**

消费者如何向消费组协调器证明⾃⼰还活着？ 通过定时向消费组协调器发送Heartbeat请求。如果超过了设定的

超时时间，那么协调器认为该消费者已经挂了。⼀旦协调器认为某个消费者挂了，那么它就会开启新⼀轮再均衡，并

且在当前其他消费者的⼼跳响应中添加“REBALANCE_IN_PROGRESS”，告诉其他消费者：重新分配分区。

**4.9** 再均衡过程

再均衡分为2步：Join和Sync

1. Join， 加⼊组。所有成员都向消费组协调器发送JoinGroup请求，请求加⼊消费组。⼀旦所有成员都发送了JoinGroup请求，协调i器从中选择⼀个消费者担任Leader的⻆⾊，并把组成员信息以及订阅信息发给Leader。

2. Sync，Leader开始分配消费⽅案，即哪个消费者负责消费哪些主题的哪些分区。⼀旦完成分配，Leader会将这个⽅案封装进SyncGroup请求中发给消费组协调器，⾮Leader也会发SyncGroup请求，只是内容为空。消费组协调器接收到分配⽅案之后会把⽅案塞进SyncGroup的response中发给各个消费者。
3. 注意：在协调器收集到所有成员请求前，它会把已收到请求放⼊⼀个叫purgatory(炼狱)的地⽅。然后是分发分配

⽅案的过程，即SyncGroup请求：注意：消费组的分区分配⽅案在客户端执⾏。Kafka交给客户端可以有更好的灵活性。Kafka默认提供三种分配策略：range和round-robin和sticky。可以通过消费者的参数： partition.assignment.strategy 来实现⾃⼰分配策略。

**4.10** 消费组状态机

消费组组协调器根据状态机对消费组做不同的处理：选项 说明

为创建的或修改的主题指定配置信息。⽀持下述配置条⽬：

cleanup.policy

compression.type

delete.retention.ms

file.delete.delay.ms

flush.messages

flush.ms

follower.replication.throttled.replicas

index.interval.bytes

leader.replication.throttled.replicas

max.message.bytes

说明：

\1. Dead：组内已经没有任何成员的最终状态，组的元数据也已经被组协调器移除了。这种状态响应各种请求都

是⼀个response： UNKNOWN_MEMBER_ID

\2. Empty：组内⽆成员，但是位移信息还没有过期。这种状态只能响应JoinGroup请求

\3. PreparingRebalance：组准备开启新的rebalance，等待成员加⼊

\4. AwaitingSync：正在等待leader consumer将分配⽅案传给各个成员

\5. Stable：再均衡完成，可以开始消费。

### **2.3** 主题

#### **2.3.1** 管理

使⽤kafka-topics.sh脚本：

![image-20211120184854454](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.11.20/image-20211120184854454-7405336.png)

![image-20211120184926292](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.11.20/image-20211120184926292-7405368.png)

主题中可以使⽤的参数定义：

![image-20211120184953661](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.11.20/image-20211120184953661-7405396.png)

![image-20211120185020321](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.11.20/image-20211120185020321-7405423.png)

##### **2.3.1.1** 创建主题

```shell
kafka-topics.sh --zookeeper localhost:2181/myKafka --create --topic topic_x --partitions 1 --replication-factor 1
kafka-topics.sh --zookeeper localhost:2181/myKafka --create --topic topic_test_02 --partitions 3 --replication-factor 1 --config max.message.bytes=1048576 --config segment.bytes=10485760
```

##### **2.3.1.2** 查看主题

```shell
kafka-topics.sh --zookeeper localhost:2181/myKafka --list
kafka-topics.sh --zookeeper localhost:2181/myKafka --describe --topic topic_x
kafka-topics.sh --zookeeper localhost:2181/myKafka --topics-with-overrides --describe
```

##### **2.3.1.3** 修改主题

```sh
kafka-topics.sh --zookeeper localhost:2181/myKafka --create --topic topic_test_01 --partitions 2 --replication-factor 1
kafka-topics.sh --zookeeper localhost:2181/myKafka --alter --topic topic_test_01 --config max.message.bytes=1048576
kafka-topics.sh --zookeeper localhost:2181/myKafka --describe --topic topic_test_01
kafka-topics.sh --zookeeper localhost:2181/myKafka --alter --topic topic_test_01 --config segment.bytes=10485760
kafka-topics.sh --zookeeper localhost:2181/myKafka --alter --delete-config max.message.bytes --topic topic_test_01
```



##### **2.3.1.4** 删除主题

```shell
kafka-topics.sh --zookeeper localhost:2181/myKafka --delete --topic topic_x
```

给主题添加删除的标记：

要过⼀段时间删除。

#### **2.3.2** 增加分区

通过命令⾏⼯具操作，主题的分区只能增加，不能减少。否则报错：

```
ERROR org.apache.kafka.common.errors.InvalidPartitionsException: The number of
partitions for a topic can only be increased. Topic myTop1 currently has 2 partitions,
1 would not be an increase.
```

通过--alter修改主题的分区数，增加分区。

```shell
kafka-topics.sh --zookeeper localhost/myKafka --alter --topic myTop1 --partitions 2
```

#### **2.3.4** 必要参数配置

kafka-topics.sh --config xx=xx --config yy=yy

配置给主题的参数。

![image-20211120185846941](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.11.20/image-20211120185846941-7405929.png)



#### **2.3.5 KafkaAdminClient**应⽤

说明

除了使⽤Kafka的bin⽬录下的脚本⼯具来管理Kafka，还可以使⽤管理Kafka的API将某些管理查看的功能集成到系统中。在Kafka0.11.0.0版本之前，可以通过kafka-core包（Kafka的服务端，采⽤Scala编写）中的AdminClient和AdminUtils来实现部分的集群管理操作。Kafka0.11.0.0之后，⼜多了⼀个AdminClient，在kafka-client包下，⼀个抽象类，具体的实现是org.apache.kafka.clients.admin.KafkaAdminClient。

功能与原理介绍

Kafka官⽹：The AdminClient API supports managing and inspecting topics, brokers, acls, and other Kafka objects。

KafkaAdminClient包含了⼀下⼏种功能（以Kafka1.0.2版本为准）：

1. 创建主题：

   createTopics(final Collection<NewTopic> newTopics, final CreateTopicsOptions options)

2. 删除主题：

   deleteTopics(final Collection<String> topicNames, DeleteTopicsOptions options)

3. 列出所有主题：

   listTopics(final ListTopicsOptions options)

4. 查询主题：

   describeTopics(final Collection<String> topicNames, DescribeTopicsOptions options)

5. 查询集群信息：

   describeCluster(DescribeClusterOptions options)

6. 查询配置信息：

   describeConfigs(Collection<ConfigResource> configResources, final DescribeConfigsOptions options)

7. 修改配置信息：

   alterConfigs(Map<ConfigResource, Config> configs, final AlterConfigsOptions options)

8. 修改副本的⽇志⽬录：

   alterReplicaLogDirs(Map<TopicPartitionReplica, String> replicaAssignment, finalAlterReplicaLogDirsOptions options)

9. 查询节点的⽇志⽬录信息：

   describeLogDirs(Collection<Integer> brokers, DescribeLogDirsOptions options)

10. 查询副本的⽇志⽬录信息：

    describeReplicaLogDirs(Collection<TopicPartitionReplica> replicas, DescribeReplicaLogDirsOptions options)

11. 增加分区：

    createPartitions(Map<String, NewPartitions> newPartitions, final CreatePartitionsOptions options)

其内部原理是使⽤Kafka⾃定义的⼀套⼆进制协议来实现，详细可以参见Kafka协议。

⽤到的参数：

![image-20211120190240756](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.11.20/image-20211120190240756-7406163.png)

![image-20211120190315153](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.11.20/image-20211120190315153-7406197.png)

主要操作步骤：

客户端根据⽅法的调⽤创建相应的协议请求，⽐如创建Topic的createTopics⽅法，其内部就是发送CreateTopicRequest请求。

客户端发送请求⾄Kafka Broker。

Kafka Broker处理相应的请求并回执，⽐如与CreateTopicRequest对应的是CreateTopicResponse。客户端接收相应的回执并进⾏解析处理。

和协议有关的请求和回执的类基本都在org.apache.kafka.common.requests包中，AbstractRequest和

AbstractResponse是这些请求和响应类的两个⽗类。

综上，如果要⾃定义实现⼀个功能，只需要三个步骤：

1. ⾃定义XXXOptions;

2. ⾃定义XXXResult返回值；

3. ⾃定义Call，然后挑选合适的XXXRequest和XXXResponse来实现Call类中的3个抽象⽅法。

```
package com.lagou.kafka.demo;

import org.apache.kafka.clients.admin.*;

import org.apache.kafka.common.KafkaFuture;

import org.apache.kafka.common.Node;

import org.apache.kafka.common.TopicPartitionInfo;

import org.apache.kafka.common.config.ConfigResource;

import org.apache.kafka.common.requests.DescribeLogDirsResponse;

import org.junit.After;

import org.junit.Before;

import org.junit.Test;

import java.util.*;

import java.util.concurrent.ExecutionException;

import java.util.concurrent.TimeUnit;

import java.util.concurrent.TimeoutException;

import java.util.function.BiConsumer;

import java.util.function.Consumer;

public class MyAdminClient {

private KafkaAdminClient client;

@Before

public void before() {

Map<String, Object> conf = new HashMap<>();

conf.put("bootstrap.servers", "node1:9092");

conf.put("client.id", "adminclient-1");

client = (KafkaAdminClient) KafkaAdminClient.create(conf);

 }

@After

public void after() {

client.close();

 }

@Test

public void testListTopics1() throws ExecutionException, InterruptedException {

ListTopicsResult listTopicsResult = client.listTopics();

// KafkaFuture<Collection<TopicListing>> listings =

listTopicsResult.listings();

// Collection<TopicListing> topicListings = listings.get();

//

// topicListings.forEach(new Consumer<TopicListing>() {

// @Override

// public void accept(TopicListing topicListing) {

// boolean internal = topicListing.isInternal();

// String name = topicListing.name();

// String s = topicListing.toString();

// System.out.println(s + "\t" + name + "\t" + internal);

// }

// });

// KafkaFuture<Set<String>> names = listTopicsResult.names();

// Set<String> strings = names.get();

//

// strings.forEach(name -> {

// System.out.println(name);

// });

// KafkaFuture<Map<String, TopicListing>> mapKafkaFuture =

listTopicsResult.namesToListings();

// Map<String, TopicListing> stringTopicListingMap = mapKafkaFuture.get();

//

// stringTopicListingMap.forEach((k, v) -> {

// System.out.println(k + "\t" + v);

// });

ListTopicsOptions options = new ListTopicsOptions();

options.listInternal(false);

options.timeoutMs(500);

ListTopicsResult listTopicsResult1 = client.listTopics(options);

Map<String, TopicListing> stringTopicListingMap =

listTopicsResult1.namesToListings().get();

stringTopicListingMap.forEach((k, v) -> {

System.out.println(k + "\t" + v);

 });

// 关闭管理客户端

client.close();

 }

@Test

public void testCreateTopic() throws ExecutionException, InterruptedException {

Map<String, String> configs = new HashMap<>();

configs.put("max.message.bytes", "1048576");

configs.put("segment.bytes", "1048576000");

NewTopic newTopic = new NewTopic("adm_tp_01", 2, (short) 1);

newTopic.configs(configs);

CreateTopicsResult topics =

client.createTopics(Collections.singleton(newTopic));

KafkaFuture<Void> all = topics.all();

Void aVoid = all.get();

System.out.println(aVoid);

 }

@Test

public void testDeleteTopic() throws ExecutionException, InterruptedException {

DeleteTopicsOptions options = new DeleteTopicsOptions();

options.timeoutMs(500);

DeleteTopicsResult deleteResult =

client.deleteTopics(Collections.singleton("adm_tp_01"), options);

deleteResult.all().get();

 }

@Test

public void testAlterTopic() throws ExecutionException, InterruptedException {

NewPartitions newPartitions = NewPartitions.increaseTo(5);

Map<String, NewPartitions> newPartitionsMap = new HashMap<>();

newPartitionsMap.put("adm_tp_01", newPartitions);

CreatePartitionsOptions option = new CreatePartitionsOptions();

// Set to true if the request should be validated without creating new

partitions.

// 如果只是验证，⽽不创建分区，则设置为true

// option.validateOnly(true);

CreatePartitionsResult partitionsResult =

client.createPartitions(newPartitionsMap, option);

Void aVoid = partitionsResult.all().get();

 }

@Test

public void testDescribeTopics() throws ExecutionException, InterruptedException

{

DescribeTopicsOptions options = new DescribeTopicsOptions();

options.timeoutMs(3000);

DescribeTopicsResult topicsResult =

client.describeTopics(Collections.singleton("adm_tp_01"), options);

Map<String, TopicDescription> stringTopicDescriptionMap =

topicsResult.all().get();

stringTopicDescriptionMap.forEach((k, v) -> {

System.out.println(k + "\t" + v);

System.out.println("=======================================");

System.out.println(k);

boolean internal = v.isInternal();

String name = v.name();

List<TopicPartitionInfo> partitions = v.partitions();

String partitionStr = Arrays.toString(partitions.toArray());

System.out.println("内部的？" + internal);

System.out.println("topic name = " + name);

System.out.println("分区：" + partitionStr);

partitions.forEach(partition -> {

System.out.println(partition);

 });

 });

 }

@Test

public void testDescribeCluster() throws ExecutionException,

InterruptedException {

DescribeClusterResult describeClusterResult = client.describeCluster();

KafkaFuture<String> stringKafkaFuture = describeClusterResult.clusterId();

String s = stringKafkaFuture.get();

System.out.println("cluster name = " + s);

KafkaFuture<Node> controller = describeClusterResult.controller();

Node node = controller.get();

System.out.println("集群控制器：" + node);

Collection<Node> nodes = describeClusterResult.nodes().get();

nodes.forEach(node1 -> {

System.out.println(node1);

 });

 }

@Test

public void testDescribeConfigs() throws ExecutionException,

InterruptedException, TimeoutException {

ConfigResource configResource = new

ConfigResource(ConfigResource.Type.BROKER, "0");

133

134

135

DescribeConfigsResult describeConfigsResult

= client.describeConfigs(Collections.singleton(configResource));

Map<ConfigResource, Config> configMap = describeConfigsResult.all().get(15,

TimeUnit.SECONDS);

configMap.forEach(new BiConsumer<ConfigResource, Config>() {

@Override

public void accept(ConfigResource configResource, Config config) {

ConfigResource.Type type = configResource.type();

String name = configResource.name();

System.out.println("资源名称：" + name);

Collection<ConfigEntry> entries = config.entries();

entries.forEach(new Consumer<ConfigEntry>() {

@Override

public void accept(ConfigEntry configEntry) {

boolean aDefault = configEntry.isDefault();

boolean readOnly = configEntry.isReadOnly();

boolean sensitive = configEntry.isSensitive();

String name1 = configEntry.name();

String value = configEntry.value();

System.out.println("是否默认：" + aDefault + "\t是否只读？" +

readOnly + "\t是否敏感？" + sensitive

\+ "\t" + name1 + " --> " + value);

 }

 });

ConfigEntry retries = config.get("retries");

if (retries != null) {

System.out.println(retries.name() + " -->" + retries.value());

 } else {

System.out.println("没有这个属性");

 }

 }

 });

 }

@Test

public void testAlterConfig() throws ExecutionException, InterruptedException {

// 这⾥设置后，原来资源中不冲突的属性也会丢失，直接按照这⾥的配置设置

Map<ConfigResource, Config> configMap = new HashMap<>();

ConfigResource resource = new ConfigResource(ConfigResource.Type.TOPIC,

"adm_tp_01");

Config config = new Config(Collections.singleton(new

ConfigEntry("segment.bytes", "1048576000")));

configMap.put(resource, config);

AlterConfigsResult alterConfigsResult = client.alterConfigs(configMap);

Void aVoid = alterConfigsResult.all().get();

 }

@Test

public void testDescribeLogDirs() throws ExecutionException,

InterruptedException {

DescribeLogDirsOptions option = new DescribeLogDirsOptions();

option.timeoutMs(1000);

DescribeLogDirsResult describeLogDirsResult =

client.describeLogDirs(Collections.singleton(0), option);

Map<Integer, Map<String, DescribeLogDirsResponse.LogDirInfo>> integerMapMap

= describeLogDirsResult.all().get();

integerMapMap.forEach(new BiConsumer<Integer, Map<String,

DescribeLogDirsResponse.LogDirInfo>>() {

@Override

public void accept(Integer integer, Map<String,

DescribeLogDirsResponse.LogDirInfo> stringLogDirInfoMap) {

System.out.println("broker.id = " + integer);

stringLogDirInfoMap.forEach(new BiConsumer<String,

DescribeLogDirsResponse.LogDirInfo>() {

@Override

public void accept(String s, DescribeLogDirsResponse.LogDirInfo

logDirInfo) {

System.out.println("log.dirs：" + s);

// 查看该broker上的主题/分区/偏移量等信息

// logDirInfo.replicaInfos.forEach(new

BiConsumer<TopicPartition, DescribeLogDirsResponse.ReplicaInfo>() {

// @Override

// public void accept(TopicPartition topicPartition,

DescribeLogDirsResponse.ReplicaInfo replicaInfo) {

// int partition = topicPartition.partition();

// String topic = topicPartition.topic();

// boolean isFuture = replicaInfo.isFuture;

// long offsetLag = replicaInfo.offsetLag;

// long size = replicaInfo.size;

// System.out.println("partition:" + partition +

"\ttopic:" + topic

// + "\tisFuture:" + isFuture

// + "\toffsetLag:" + offsetLag

// + "\tsize:" + size);

// }

// });

 }

 });

 }
 });

 }

}
```



#### **2.3.6** 偏移量管理

Kafka 1.0.2，__consumer_offsets主题中保存各个消费组的偏移量。

早期由zookeeper管理消费组的偏移量。

查询⽅法：

通过原⽣ kafka 提供的⼯具脚本进⾏查询。

⼯具脚本的位置与名称为 bin/kafka-consumer-groups.sh

⾸先运⾏脚本，查看帮助：

![image-20211120190558020](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.11.20/image-20211120190558020-7406362.png)

![image-20211120190620572](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.11.20/image-20211120190620572-7406382.png)

![image-20211120190635960](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.11.20/image-20211120190635960-7406397.png)

![image-20211120190657082](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.11.20/image-20211120190657082-7406418.png)

![image-20211120190719731](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.11.20/image-20211120190719731-7406442.png)

![image-20211120190735098](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.11.20/image-20211120190735098-7406456.png)

这⾥我们先编写⼀个⽣产者，消费者的例⼦：

我们先启动消费者，再启动⽣产者， 再通过 bin/kafka-consumer-groups.sh 进⾏消费偏移量查询，

由于kafka 消费者记录group的消费偏移量有两种⽅式 ： 

1）kafka ⾃维护 （新）

2）zookpeer 维护 (旧) ，已经逐渐被废弃

所以 ，脚本只查看由broker维护的，由zookeeper维护的可以将 --bootstrap-server 换成 --zookeeper 即可。

1**.** 查看有那些 **group ID** 正在进⾏消费：

```
[root@node11 ~]# kafka-consumer-groups.sh --bootstrap-server node1:9092 --list
Note: This will not show information about old Zookeeper-based consumers.

group
```

注意：

1. 这⾥⾯是没有指定 **topic**，查看的是所有**topic**消费者的 **group.id** 的列表。

2. 注意： 重名的 **group.id** 只会显示⼀次

**2.**查看指定**group.id** 的消费者消费情况

```shell
[root@node11 ~]# kafka-consumer-groups.sh --bootstrap-server node1:9092 --describe --group group
Note: This will not show information about old Zookeeper-based consumers.
TOPIC PARTITION CURRENT-OFFSET LOG-END-OFFSET LAG 
CONSUMER-ID HOST 
CLIENT-ID
tp_demo_02 0 923 923 0
consumer-1-6d88cc72-1bf1-4ad7-8c6c-060d26dc1c49 /192.168.100.1 
consumer-1
tp_demo_02 1 872 872 0
consumer-1-6d88cc72-1bf1-4ad7-8c6c-060d26dc1c49 /192.168.100.1 
consumer-1
tp_demo_02 2 935 935 0
consumer-1-6d88cc72-1bf1-4ad7-8c6c-060d26dc1c49 /192.168.100.1 
consumer-1
[root@node11 ~]# 
```

如果消费者停⽌，查看偏移量信息：

将偏移量设置为最早的：

将偏移量设置为最新的：

分别将指定主题的指定分区的偏移量向前移动10个消息：



代码：

**KafkaProducerSingleton.java**

**ProducerHandler.java**

**KafkaConsumerAuto.java**

**ConsumerThreadAuto.java**



### **2.4** 分区

#### **2.4.1** 副本机制

Kafka在⼀定数量的服务器上对主题分区进⾏复制。

当集群中的⼀个broker宕机后系统可以⾃动故障转移到其他可⽤的副本上，不会造成数据丢失。

--replication-factor 3 1leader+2follower

1. 将复制因⼦为1的未复制主题称为复制主题。

2. 主题的分区是复制的最⼩单元。

3. 在⾮故障情况下，Kafka中的每个分区都有⼀个Leader副本和零个或多个Follower副本。

4. 包括Leader副本在内的副本总数构成复制因⼦。

5. 所有读取和写⼊都由Leader副本负责。

6. 通常，分区⽐broker多，并且Leader分区在broker之间平均分配。

**Follower**分区像普通的**Kafka**消费者⼀样，消费来⾃**Leader**分区的消息，并将其持久化到⾃⼰的⽇志中。

允许Follower对⽇志条⽬拉取进⾏批处理。

同步节点定义：

1. 节点必须能够维持与ZooKeeper的会话（通过ZooKeeper的⼼跳机制）

2. 对于Follower副本分区，它复制在Leader分区上的写⼊，并且不要延迟太多

Kafka提供的保证是，只要有⾄少⼀个同步副本处于活动状态，提交的消息就不会丢失。

宕机如何恢复

（1）少部分副本宕机

当leader宕机了，会从follower选择⼀个作为leader。当宕机的重新恢复时，会把之前commit的数据清空，重新

从leader⾥pull数据。

（2）全部副本宕机

当全部副本宕机了有两种恢复⽅式

1、等待ISR中的⼀个恢复后，并选它作为leader。（等待时间较⻓，降低可⽤性）

2、选择第⼀个恢复的副本作为新的leader，⽆论是否在ISR中。（并未包含之前leader commit的数据，因此造成

数据丢失）

#### **2.4.2 Leader**选举

下图中

分区P1的Leader是0，ISR是0和1

分区P2的Leader是2，ISR是1和2

分区P3的Leader是1，ISR是0，1，2。⽣产者和消费者的请求都由Leader副本来处理。Follower副本只负责消费Leader副本的数据和Leader保持同步。

对于P1，如果0宕机会发⽣什么？

Leader副本和Follower副本之间的关系并不是固定不变的，在Leader所在的broker发⽣故障的时候，就需要进⾏

分区的Leader副本和Follower副本之间的切换，需要选举Leader副本。

如何选举？

如果某个分区所在的服务器除了问题，不可⽤，kafka会从该分区的其他的副本中选择⼀个作为新的Leader。之后所有的读写就会转移到这个新的Leader上。现在的问题是应当选择哪个作为新的Leader。

只有那些跟Leader保持同步的Follower才应该被选作新的Leader。

Kafka会在Zookeeper上针对每个Topic维护⼀个称为ISR（in-sync replica，已同步的副本）的集合，该集合中是⼀些分区的副本。

只有当这些副本都跟Leader中的副本同步了之后，kafka才会认为消息已提交，并反馈给消息的⽣产者。

如果这个集合有增减，kafka会更新zookeeper上的记录。

如果某个分区的Leader不可⽤，Kafka就会从ISR集合中选择⼀个副本作为新的Leader。

显然通过ISR，kafka需要的冗余度较低，可以容忍的失败数⽐较⾼。

假设某个topic有N+1个副本，kafka可以容忍N个服务器不可⽤。

为什么不⽤少数服从多数的⽅法少数服从多数是⼀种⽐较常⻅的⼀致性算发和Leader选举法。

它的含义是只有超过半数的副本同步了，系统才会认为数据已同步；

选择Leader时也是从超过半数的同步的副本中选择。

这种算法需要较⾼的冗余度，跟Kafka⽐起来，浪费资源。

譬如只允许⼀台机器失败，需要有三个副本；⽽如果只容忍两台机器失败，则需要五个副本。

⽽kafka的ISR集合⽅法，分别只需要两个和三个副本。

如果所有的**ISR**副本都失败了怎么办？

此时有两种⽅法可选，

1. 等待ISR集合中的副本复活，

2. 选择任何⼀个⽴即可⽤的副本，⽽这个副本不⼀定是在ISR集合中。

需要设置 unclean.leader.election.enable=true

这两种⽅法各有利弊，实际⽣产中按需选择。

  如果要等待ISR副本复活，虽然可以保证⼀致性，但可能需要很⻓时间。⽽如果选择⽴即可⽤的副本，则很可能该副本并不⼀致。

总结：

Kafka中Leader分区选举，通过维护⼀个动态变化的ISR集合来实现，⼀旦Leader分区丢掉，则从ISR中随机挑选

⼀个副本做新的Leader分区。

如果ISR中的副本都丢失了，则：

1. 可以等待ISR中的副本任何⼀个恢复，接着对外提供服务，需要时间等待。

2. 从OSR中选出⼀个副本做Leader副本，此时会造成数据丢失

#### **2.4.3** 分区重新分配

向已经部署好的Kafka集群⾥⾯添加机器，我们需要从已经部署好的Kafka节点中复制相应的配置⽂件，然后把⾥⾯的broker id修改成全局唯⼀的，最后启动这个节点即可将它加⼊到现有Kafka集群中。

问题：新添加的Kafka节点并不会⾃动地分配数据，⽆法分担集群的负载，除⾮我们新建⼀个topic。

需要⼿动将部分分区移到新添加的Kafka节点上，Kafka内部提供了相关的⼯具来重新分布某个topic的分区。

在重新分布topic分区之前，我们先来看看现在topic的各个分区的分布位置：

1. 创建主题：
2. 查看主题信息：
3. 在node11搭建Kafka：



#### **2.4.4** ⾃动再均衡

我们可以在新建主题的时候，⼿动指定主题各个Leader分区以及Follower分区的分配情况，即什么分区副本在哪个broker节点上。

随着系统的运⾏，broker的宕机重启，会引发Leader分区和Follower分区的⻆⾊转换，最后可能Leader⼤部分都集中在少数⼏台broker上，由于Leader负责客户端的读写操作，此时集中Leader分区的少数⼏台服务器的⽹络I/O，CPU，以及内存都会很紧张。

Leader和Follower的⻆⾊转换会引起Leader副本在集群中分布的不均衡，此时我们需要⼀种⼿段，让Leader的分布重新恢复到⼀个均衡的状态。

执⾏脚本：

上述脚本执⾏的结果是：创建了主题tp_demo_03，有三个分区，每个分区两个副本，Leader副本在列表中第⼀个指定的brokerId上，Follower副本在随后指定的brokerId上。

然后模拟broker0宕机的情况：



是否有⼀种⽅式，可以让Kafka⾃动帮我们进⾏修改？改为初始的副本分配？

此时，⽤到了Kafka提供的⾃动再均衡脚本： kafka-preferred-replica-election.sh

先看介绍：

该⼯具会让每个分区的Leader副本分配在合适的位置，让Leader分区和Follower分区在服务器之间均衡分配。

如果该脚本仅指定zookeeper地址，则会对集群中所有的主题进⾏操作，⾃动再平衡。

具体操作：

1. 创建preferred-replica.json，内容如下：

2. 执⾏操作：

3. 查看操作的结果：

恢复到最初的分配情况。

之所以是这样的分配，是因为我们在创建主题的时候：

在逗号分割的每个数值对中排在前⾯的是Leader分区，后⾯的是副本分区。那么所谓的preferred replica，就是排在前⾯的数字就是Leader副本应该在的brokerId。



#### **2.4.5** 修改分区副本

实际项⽬中，我们可能由于主题的副本因⼦设置的问题，需要重新设置副本因⼦

或者由于集群的扩展，需要重新设置副本因⼦。

topic⼀旦使⽤⼜不能轻易删除重建，因此动态增加副本因⼦就成为最终的选择。

说明：kafka 1.0版本配置⽂件默认没有default.replication.factor=x， 因此如果创建topic时，不指定–

replication-factor 想， 默认副本因⼦为1. 我们可以在⾃⼰的server.properties中配置上常⽤的副本因⼦，省去⼿动调

整。例如设置default.replication.factor=3， 详细内容可参考官⽅⽂档https://kafka.apache.org/documentation/#r

eplication

原因分析：

假设我们有2个kafka broker分别broker0，broker1。

1. 当我们创建的topic有2个分区partition时并且replication-factor为1，基本上⼀个broker上⼀个分区。当⼀个broker宕机了，该topic就⽆法使⽤了，因为两个个分区只有⼀个能⽤。

2. 当我们创建的topic有3个分区partition时并且replication-factor为2时，可能分区数据分布情况是

每个分区有⼀个副本，当其中⼀个broker宕机了，kafka集群还能完整凑出该topic的两个分区，例如当

broker0宕机了，可以通过broker1组合出topic的两个分区。

1. 创建主题：

2. 查看主题细节：

3. 修改副本因⼦：错误

4. 使⽤ kafka-reassign-partitions.sh 修改副本因⼦：

5. 执⾏分配：

6. 查看主题细节：
7. 搞定！

#### **2.4.6** 分区分配策略

在Kafka中，每个Topic会包含多个分区，默认情况下⼀个分区只能被⼀个消费组下⾯的⼀个消费者消费，这⾥就

产⽣了分区分配的问题。Kafka中提供了多重分区分配算法（PartitionAssignor）的实现：RangeAssignor、

RoundRobinAssignor、StickyAssignor。

##### **2.4.6.1 RangeAssignor**

PartitionAssignor接⼝⽤于⽤户定义实现分区分配算法，以实现Consumer之间的分区分配。

消费组的成员订阅它们感兴趣的Topic并将这种订阅关系传递给作为订阅组协调者的Broker。协调者选择其中的⼀

个消费者来执⾏这个消费组的分区分配并将分配结果转发给消费组内所有的消费者。**Kafka**默认采⽤**RangeAssignor**

的分配算法。RangeAssignor对每个Topic进⾏独⽴的分区分配。对于每⼀个Topic，⾸先对分区按照分区ID进⾏数值排序，然

后订阅这个Topic的消费组的消费者再进⾏字典排序，之后尽量均衡的将分区分配给消费者。这⾥只能是尽量均衡，因

为分区数可能⽆法被消费者数量整除，那么有⼀些消费者就会多分配到⼀些分区。

⼤致算法如下：

RangeAssignor策略的原理是按照消费者总数和分区总数进⾏整除运算来获得⼀个跨度，然后将分区按照跨度进

⾏平均分配，以保证分区尽可能均匀地分配给所有的消费者。对于每⼀个Topic，RangeAssignor策略会将消费组内所

有订阅这个Topic的消费者按照名称的字典序排序，然后为每个消费者划分固定的分区范围，如果不够平均分配，那么

字典序靠前的消费者会被多分配⼀个分区。



这种分配⽅式明显的⼀个问题是随着消费者订阅的Topic的数量的增加，不均衡的问题会越来越严重，⽐如上图中

4个分区3个消费者的场景，C0会多分配⼀个分区。如果此时再订阅⼀个分区数为4的Topic，那么C0⼜会⽐C1、C2多

分配⼀个分区，这样C0总共就⽐C1、C2多分配两个分区了，⽽且随着Topic的增加，这个情况会越来越严重。

字典序靠前的消费组中的消费者⽐较“贪婪”。

##### **2.4.6.2 RoundRobinAssignor**

RoundRobinAssignor的分配策略是将消费组内订阅的所有Topic的分区及所有消费者进⾏排序后尽量均衡的分配

（RangeAssignor是针对单个Topic的分区进⾏排序分配的）。如果消费组内，消费者订阅的Topic列表是相同的（每

个消费者都订阅了相同的Topic），那么分配结果是尽量均衡的（消费者之间分配到的分区数的差值不会超过1）。如

果订阅的Topic列表是不同的，那么分配结果是不保证“尽量均衡”的，因为某些消费者不参与⼀些Topic的分配。相对于RangeAssignor，在订阅多个Topic的情况下，RoundRobinAssignor的⽅式能消费者之间尽量均衡的分配

到分区（分配到的分区数的差值不会超过1——RangeAssignor的分配策略可能随着订阅的Topic越来越多，差值越来

越⼤）。

对于消费组内消费者订阅Topic不⼀致的情况：假设有两个个消费者分别为C0和C1，有2个Topic T1、T2，分别拥

有3和2个分区，并且C0订阅T1和T2，C1订阅T2，那么RoundRobinAssignor的分配结果如下：

看上去分配已经尽量的保证均衡了，不过可以发现C0承担了4个分区的消费⽽C1订阅了T2⼀个分区，是不是把

T2P0交给C1消费能更加的均衡呢？

##### **2.4.6.3 StickyAssignor**

动机

尽管RoundRobinAssignor已经在RangeAssignor上做了⼀些优化来更均衡的分配分区，但是在⼀些情况下依旧会

产⽣严重的分配偏差，⽐如消费组中订阅的Topic列表不相同的情况下。

更核⼼的问题是⽆论是RangeAssignor，还是RoundRobinAssignor，当前的分区分配算法都没有考虑上⼀次的分

配结果。显然，在执⾏⼀次新的分配之前，如果能考虑到上⼀次分配的结果，尽量少的调整分区分配的变动，显然是

能节省很多开销的。

⽬标

从字⾯意义上看，Sticky是“粘性的”，可以理解为分配结果是带“粘性的”：

\1. 分区的分配尽量的均衡

\2. 每⼀次重分配的结果尽量与上⼀次分配结果保持⼀致

当这两个⽬标发⽣冲突时，优先保证第⼀个⽬标。第⼀个⽬标是每个分配算法都尽量尝试去完成的，⽽第⼆个⽬

标才真正体现出StickyAssignor特性的。

我们先来看预期分配的结构，后续再具体分析StickyAssignor的算法实现。

例如：

有3个Consumer：C0、C1、C2

有4个Topic：T0、T1、T2、T3，每个Topic有2个分区

所有Consumer都订阅了这4个分区

StickyAssignor的分配结果如下图所示（增加RoundRobinAssignor分配作为对⽐）：如果消费者1宕机，则按照RoundRobin的⽅式分配结果如下：

打乱从新来过，轮询分配：

按照Sticky的⽅式：

仅对消费者1分配的分区进⾏重分配，红线部分。最终达到均衡的⽬的。

再举⼀个例⼦：

有3个Consumer：C0、C1、C2

3个Topic：T0、T1、T2，它们分别有1、2、3个分区

C0订阅T0；C1订阅T0、T1；C2订阅T0、T1、T2

分配结果如下图所示：消费者0下线，则按照轮询的⽅式分配：

按照Sticky⽅式分配分区，仅仅需要动的就是红线部分，其他部分不动。 StickyAssignor分配⽅式的实现稍微复杂点⼉，我们可以先理解图示部分即可。感兴趣的同学可以研究⼀下。

**2.4.6.4** ⾃定义分配策略

⾃定义的分配策略必须要实现org.apache.kafka.clients.consumer.internals.PartitionAssignor接⼝。

PartitionAssignor接⼝的定义如下：

PartitionAssignor接⼝中定义了两个内部类：Subscription和Assignment。

Subscription类⽤来表示消费者的订阅信息，类中有两个属性：topics和userData，分别表示消费者所订阅topic列表和⽤户⾃定义信息。PartitionAssignor接⼝通过subscription()⽅法来设置消费者⾃身相关的Subscription信息，注意到此⽅法中只有⼀个参数topics，与Subscription类中的topics的相互呼应，但是并没有有关userData的参数体现。为了增强⽤户对分配结果的控制，可以在subscription()⽅法内部添加⼀些影响分配的⽤户⾃定义信息赋予userData，⽐如：权重、ip地址、host或者机架（rack）等等。

再来说⼀下Assignment类，它是⽤来表示分配结果信息的，类中也有两个属性：partitions和userData，分别表示所分配到的分区集合和⽤户⾃定义的数据。可以通过PartitionAssignor接⼝中的onAssignment()⽅法是在每个消费者收到消费组leader分配结果时的回调函数，例如在StickyAssignor策略中就是通过这个⽅法保存当前的分配⽅案，以备在下次消费组再平衡（rebalance）时可以提供分配参考依据。

接⼝中的name()⽅法⽤来提供分配策略的名称，对于Kafka提供的3种分配策略⽽⾔，RangeAssignor对应的protocol_name为“range”，RoundRobinAssignor对应的protocol_name为“roundrobin”，StickyAssignor对应的protocol_name为“sticky”，所以⾃定义的分配策略中要注意命名的时候不要与已存在的分配策略发⽣冲突。这个命名⽤来标识分配策略的名称，在后⾯所描述的加⼊消费组以及选举消费组leader的时候会有涉及。

真正的分区分配⽅案的实现是在assign()⽅法中，⽅法中的参数metadata表示集群的元数据信息，⽽subscriptions表示消费组内各个消费者成员的订阅信息，最终⽅法返回各个消费者的分配信息。

Kafka中还提供了⼀个抽象类org.apache.kafka.clients.consumer.internals.AbstractPartitionAssignor，它可以简化PartitionAssignor接⼝的实现，对assign()⽅法进⾏了实现，其中会将Subscription中的userData信息去掉后，在进⾏分配。Kafka提供的3种分配策略都是继承⾃这个抽象类。如果开发⼈员在⾃定义分区分配策略时需要使⽤userData信息来控制分区分配的结果，那么就不能直接继承AbstractPartitionAssignor这个抽象类，⽽需要直接实现PartitionAssignor接⼝。

在使⽤时，消费者客户端需要添加相应的Properties参数，示例如下：



### **2.5** 物理存储

#### **2.5.1** ⽇志存储概述

Kafka 消息是以主题为单位进⾏归类，各个主题之间是彼此独⽴的，互不影响。

每个主题⼜可以分为⼀个或多个分区。

每个分区各⾃存在⼀个记录消息数据的⽇志⽂件。

图中，创建了⼀个 tp_demo_01 主题，其存在6个 Parition，对应的每个Parition下存在⼀个 [Topic

Parition] 命名的消息⽇志⽂件。在理想情况下，数据流量分摊到各个 Parition 中，实现了负载均衡的效果。在分区⽇志⽂件中，你会发现很多类型的⽂件，⽐如： .index、.timestamp、.log、.snapshot 等。

其中，⽂件名⼀致的⽂件集合就称为 LogSement。

**LogSegment**

1. 分区⽇志⽂件中包含很多的 LogSegment

2. Kafka ⽇志追加是顺序写⼊的

3. LogSegment 可以减⼩⽇志⽂件的⼤⼩

4. 进⾏⽇志删除的时候和数据查找的时候可以快速定位。

5. ActiveLogSegment 是活跃的⽇志分段，拥有⽂件拥有写⼊权限，其余的 LogSegment 只有只读的权限。⽇志⽂件存在多种后缀⽂件，重点需要关注 .index、.timestamp、.log 三种类型。

类别作用

![image-20211120194419332](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.11.20/image-20211120194419332-7408662.png)

每个 LogSegment 都有⼀个基准偏移量，表示当前 LogSegment 中第⼀条消息的 **offset**。

偏移量是⼀个 64 位的⻓整形数，固定是20位数字，⻓度未达到，⽤ 0 进⾏填补，索引⽂件和⽇志⽂件都由该作为⽂件名命名规则（00000000000000000000.index、00000000000000000000.timestamp、00000000000000000000.log）。

如果⽇志⽂件名为 00000000000000000121.log ，则当前⽇志⽂件的⼀条数据偏移量就是 121（偏移量从 0 开始）。

⽇志与索引⽂件

![image-20211120194508965](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.11.20/image-20211120194508965-7408710.png)

配置项默认值说明

偏移量索引⽂件⽤于记录消息偏移量与物理地址之间的映射关系。

时间戳索引⽂件则根据时间戳查找对应的偏移量。

Kafka 中的索引⽂件是以稀疏索引的⽅式构造消息的索引，并不保证每⼀个消息在索引⽂件中都有对应的索引项。

每当写⼊⼀定量的消息时，偏移量索引⽂件和时间戳索引⽂件分别增加⼀个偏移量索引项和时间戳索引项。

通过修改 log.index.interval.bytes 的值，改变索引项的密度。

**切分⽂件**

当满⾜如下⼏个条件中的其中之⼀，就会触发⽂件的切分：

1. 当前⽇志分段⽂件的⼤⼩超过了 broker 端参数 log.segment.bytes 配置的值。 log.segment.bytes 参数的默认值为 1073741824，即 1GB。

2. 当前⽇志分段中消息的最⼤时间戳与当前系统的时间戳的差值⼤于 log.roll.ms 或 log.roll.hours 参数配置的值。如果同时配置了 log.roll.ms 和 log.roll.hours 参数，那么 log.roll.ms 的优先级⾼。默认情况下，只配置了 log.roll.hours 参数，其值为168，即 7 天。

3. 偏移量索引⽂件或时间戳索引⽂件的⼤⼩达到 broker 端参数 log.index.size.max.bytes 配置的值。 log.index.size.max.bytes 的默认值为 10485760，即 10MB。

4. 追加的消息的偏移量与当前⽇志分段的偏移量之间的差值⼤于 Integer.MAX_VALUE ，即要追加的消息的偏移量不能转变为相对偏移量。

为什么是 **Integer.MAX_VALUE** ？

1024 * 1024 * 1024=1073741824

在偏移量索引⽂件中，每个索引项共占⽤ 8 个字节，并分为两部分。

相对偏移量和物理地址。相对偏移量：表示消息相对与基准偏移量的偏移量，占 4 个字节

物理地址：消息在⽇志分段⽂件中对应的物理位置，也占 4 个字节

4 个字节刚好对应 Integer.MAX_VALUE ，如果⼤于 Integer.MAX_VALUE ，则不能⽤ 4 个字节进⾏表示了。

**索引⽂件切分过程**

索引⽂件会根据 log.index.size.max.bytes 值进⾏预先分配空间，即⽂件创建的时候就是最⼤值

当真正的进⾏索引⽂件切分的时候，才会将其裁剪到实际数据⼤⼩的⽂件。

这⼀点是跟⽇志⽂件有所区别的地⽅。其意义降低了代码逻辑的复杂性。

#### **2.5.2** ⽇志存储

##### **2.5.2.1** 索引

偏移量索引⽂件⽤于记录消息偏移量与物理地址之间的映射关系。时间戳索引⽂件则根据时间戳查找对应的偏移量。

⽂件：

查看⼀个topic分区⽬录下的内容，发现有log、index和timeindex三个⽂件：

1. log⽂件名是以⽂件中第⼀条message的offset来命名的，实际offset⻓度是64位，但是这⾥只使⽤了20位，应付⽣产是⾜够的。

2. ⼀组index+log+timeindex⽂件的名字是⼀样的，并且log⽂件默认写满1G后，会进⾏log rolling形成⼀个新的组合来记录消息，这个是通过broker端 log.segment.bytes =1073741824指定的。

3. index和timeindex在刚使⽤时会分配10M的⼤⼩，当进⾏ log rolling 后，它会修剪为实际的⼤⼩。

   

1、创建主题：

2、创建消息⽂件：

3、将⽂本消息⽣产到主题中：

4、查看存储⽂件：

如果想查看这些⽂件，可以使⽤kafka提供的shell来完成，⼏个关键信息如下：

（1）offset是逐渐增加的整数，每个offset对应⼀个消息的偏移量。

（2）position：消息批字节数，⽤于计算物理地址。

（3）CreateTime：时间戳。

（4）magic：2代表这个消息类型是V2，如果是0则代表是V0类型，1代表V1类型。

（5）compresscodec：None说明没有指定压缩类型，kafka⽬前提供了4种可选择，0-None、1-GZIP、2-snappy、3-lz4。 

（6）crc：对所有字段进⾏校验后的crc值。

关于消息偏移量：

⼀、消息存储

1. 消息内容保存在log⽇志⽂件中。

2. 消息封装为Record，追加到log⽇志⽂件末尾，采⽤的是顺序写模式。

3. ⼀个topic的不同分区，可认为是queue，顺序写⼊接收到的消息。

消费者有offset。下图中，消费者A消费的offset是9，消费者B消费的offset是11，不同的消费者offset是交给⼀个内部公共topic来记录的。

（3）时间戳索引⽂件，它的作⽤是可以让⽤户查询某个时间段内的消息，它⼀条数据的结构是时间戳（8byte） +相对offset（4byte），如果要使⽤这个索引⽂件，⾸先需要通过时间范围，找到对应的相对offset，然后再去对应的index⽂件找到position信息，然后才能遍历log⽂件，它也是需要使⽤上⾯说的index⽂件的。

但是由于producer⽣产消息可以指定消息的时间戳，这可能将导致消息的时间戳不⼀定有先后顺序，因此尽量不要⽣产消息时指定时间戳。

**2.5.2.1.1** 偏移量

1. 位置索引保存在index⽂件中

2. log⽇志默认每写⼊4K（log.index.interval.bytes设定的），会写⼊⼀条索引信息到index⽂件中，因此索引

⽂件是稀疏索引，它不会为每条⽇志都建⽴索引信息。

3. log⽂件中的⽇志，是顺序写⼊的，由message+实际offset+position组成

4. 索引⽂件的数据结构则是由相对offset（4byte）+position（4byte）组成，由于保存的是相对第⼀个消息的

相对offset，只需要4byte就可以了，可以节省空间，在实际查找后还需要计算回实际的offset，这对⽤户是

透明的。稀疏索引，索引密度不⾼，但是offset有序，⼆分查找的时间复杂度为O(lgN)，如果从头遍历时间复杂度是O(N)。

示意图如下：

偏移量索引由相对偏移量和物理地址组成。

可以通过如下命令解析 .index ⽂件

注意：offset 与 position 没有直接关系，因为会删除数据和清理⽇志。

kafka-run-class.sh kafka.tools.DumpLogSegments --files 00000000000000000000.index --

print-data-log | head

1在偏移量索引⽂件中，索引数据都是顺序记录 offset ，但时间戳索引⽂件中每个追加的索引时间戳必须⼤于之前

追加的索引项，否则不予追加。在 Kafka 0.11.0.0 以后，消息元数据中存在若⼲的时间戳信息。如果 broker 端参

数 log.message.timestamp.type 设置为 LogAppendTIme ，那么时间戳必定能保持单调增⻓。反之如果是

CreateTime 则⽆法保证顺序。

注意：timestamp⽂件中的 offset 与 index ⽂件中的 relativeOffset 不是⼀⼀对应的。因为数据的写⼊是各⾃追加。

思考：如何查看偏移量为23的消息？



Kafka 中存在⼀个 ConcurrentSkipListMap 来保存在每个⽇志分段，通过跳跃表⽅式，定位到在

00000000000000000000.index ，通过⼆分法在偏移量索引⽂件中找到不⼤于 23 的最⼤索引项，即 offset 20 那栏，然后从⽇志分段⽂件中的物理位置为320 开始顺序查找偏移量为 23 的消息。

###### **2.5.2.1.2** 时间戳

在偏移量索引⽂件中，索引数据都是顺序记录 offset ，但时间戳索引⽂件中每个追加的索引时间戳必须⼤于之前

追加的索引项，否则不予追加。在 Kafka 0.11.0.0 以后，消息信息中存在若⼲的时间戳信息。如果 broker 端参数

log.message.timestamp.type 设置为 LogAppendTIme ，那么时间戳必定能保持单调增⻓。反之如果是

CreateTime 则⽆法保证顺序。

通过时间戳⽅式进⾏查找消息，需要通过查找时间戳索引和偏移量索引两个⽂件。

时间戳索引索引格式：前⼋个字节表示时间戳，后四个字节表示偏移量。思考：查找时间戳为 **1557554753430** 开始的消息？

1. 查找该时间戳应该在哪个⽇志分段中。将1557554753430和每个⽇志分段中最⼤时间戳largestTimeStamp

逐⼀对⽐，直到找到不⼩于1557554753430所对应的⽇志分段。⽇志分段中的largestTimeStamp的计算

是：先查询该⽇志分段所对应时间戳索引⽂件，找到最后⼀条索引项，若最后⼀条索引项的时间戳字段值⼤

于0，则取该值，否则取该⽇志分段的最近修改时间。

2. 查找该⽇志分段的偏移量索引⽂件，查找该偏移量对应的物理地址。

3. ⽇志⽂件中从 320 的物理位置开始查找不⼩于 1557554753430 数据。

注意：timestamp⽂件中的 offset 与 index ⽂件中的 relativeOffset 不是⼀⼀对应的，因为数据的写⼊是各⾃追

加。

##### **2.5.2.2** 清理

Kafka 提供两种⽇志清理策略：

⽇志删除：按照⼀定的删除策略，将不满⾜条件的数据进⾏数据删除

⽇志压缩：针对每个消息的 Key 进⾏整合，对于有相同 Key 的不同 Value 值，只保留最后⼀个版本。

Kafka 提供 log.cleanup.policy 参数进⾏相应配置，默认值： delete ，还可以选择 compact 。

主题级别的配置项是 cleanup.policy 。

###### **2.5.2.2.1** ⽇志删除

**基于时间**

⽇志删除任务会根据 log.retention.hours/log.retention.minutes/log.retention.ms 设定⽇志保留的时间节点。如果超过该设定值，就需要进⾏删除。默认是 7 天， log.retention.ms 优先级最⾼。

Kafka 依据⽇志分段中最⼤的时间戳进⾏定位。

⾸先要查询该⽇志分段所对应的时间戳索引⽂件，查找时间戳索引⽂件中最后⼀条索引项，若最后⼀条索引项的时间戳字段值⼤于 0，则取该值，否则取最近修改时间。

**为什么不直接选最近修改时间呢**？

因为⽇志⽂件可以有意⽆意的被修改，并不能真实的反应⽇志分段的最⼤时间信息。

**删除过程**

1. 从⽇志对象中所维护⽇志分段的跳跃表中移除待删除的⽇志分段，保证没有线程对这些⽇志分段进⾏读取操作。

2. 这些⽇志分段所有⽂件添加 上 .delete 后缀。

3. 交由⼀个以 "delete-file" 命名的延迟任务来删除这些 .delete 为后缀的⽂件。延迟执⾏时间可以通过file.delete.delay.ms 进⾏设置

**如果活跃的⽇志分段中也存在需要删除的数据时？**

Kafka 会先切分出⼀个新的⽇志分段作为活跃⽇志分段，该⽇志分段不删除，删除原来的⽇志分段。

先腾出地⽅，再删除。

**基于⽇志⼤⼩**

⽇志删除任务会检查当前⽇志的⼤⼩是否超过设定值。设定项为 log.retention.bytes ，单个⽇志分段的⼤⼩由 log.segment.bytes 进⾏设定。

**删除过程**

1. 计算需要被删除的⽇志总⼤⼩ (当前⽇志⽂件⼤⼩（所有分段）减去retention值)。

2. 从⽇志⽂件第⼀个 LogSegment 开始查找可删除的⽇志分段的⽂件集合。

3. 执⾏删除。

**基于偏移量**

根据⽇志分段的下⼀个⽇志分段的起始偏移量是否⼤于等于⽇志⽂件的起始偏移量，若是，则可以删除此⽇志分

段。

注意：⽇志⽂件的起始偏移量并不⼀定等于第⼀个⽇志分段的基准偏移量，存在数据删除，可能与之相等的那条数据已经被删除了。

**删除过程**

1. 从头开始遍历每个⽇志分段，⽇志分段1的下⼀个⽇志分段的起始偏移量为21，⼩于logStartOffset，将⽇志

分段1加⼊到删除队列中

2. ⽇志分段 2 的下⼀个⽇志分段的起始偏移量为35，⼩于 logStartOffset，将 ⽇志分段 2 加⼊到删除队列中

3. ⽇志分段 3 的下⼀个⽇志分段的起始偏移量为57，⼩于logStartOffset，将⽇志分段3加⼊删除集合中

4. ⽇志分段4的下⼀个⽇志分段的其实偏移量为71，⼤于logStartOffset，则不进⾏删除。**2.5.2.2.2** ⽇志压缩策略

**1.** 概念

⽇志压缩是Kafka的⼀种机制，可以提供较为细粒度的记录保留，⽽不是基于粗粒度的基于时间的保留。

对于具有相同的Key，⽽数据不同，只保留最后⼀条数据，前⾯的数据在合适的情况下删除。

**2.** 应⽤场景

⽇志压缩特性，就实时计算来说，可以在异常容灾⽅⾯有很好的应⽤途径。⽐如，我们在Spark、Flink中做实时计算时，需要长期在内存⾥⾯维护⼀些数据，这些数据可能是通过聚合了⼀天或者⼀周的⽇志得到的，这些数据⼀旦由于异常因素（内存、⽹络、磁盘等）崩溃了，从头开始计算需要很⻓的时间。⼀个⽐较有效可⾏的⽅式就是定时将内存⾥的数据备份到外部存储介质中，当崩溃出现时，再从外部存储介质中恢复并继续计算。

使⽤⽇志压缩来替代这些外部存储有哪些优势及好处呢？这⾥为⼤家列举并总结了⼏点：

- Kafka即是数据源⼜是存储⼯具，可以简化技术栈，降低维护成本
- 使⽤外部存储介质的话，需要将存储的Key记录下来，恢复的时候再使⽤这些Key将数据取回，实现起来有⼀定的⼯程难度和复杂度。使⽤Kafka的⽇志压缩特性，只需要把数据写进Kafka，等异常出现恢复任务时再读回到内存就可以了
- Kafka对于磁盘的读写做了⼤量的优化⼯作，⽐如磁盘顺序读写。相对于外部存储介质没有索引查询等⼯作量的负担，可以实现⾼性能。同时，Kafka的⽇志压缩机制可以充分利⽤廉价的磁盘，不⽤依赖昂贵的内存来处理，在性能相似的情况下，实现⾮常⾼的性价⽐（这个观点仅仅针对于异常处理和容灾的场景来说）

**2.3** ⽇志压缩⽅式的实现细节

主题的 cleanup.policy 需要设置为compact。

Kafka的后台线程会定时将Topic遍历两次：

1. 记录每个key的hash值最后⼀次出现的偏移量

2. 第⼆次检查每个offset对应的Key是否在后⾯的⽇志中出现过，如果出现了就删除对应的⽇志。

⽇志压缩允许删除，除最后⼀个key之外，删除先前出现的所有该key对应的记录。在⼀段时间后从⽇志中清理，以释放空间。

注意：⽇志压缩与key有关，确保每个消息的**key**不为**null**。

压缩是在Kafka后台通过定时重新打开Segment来完成的，Segment的压缩细节如下图所示：

⽇志压缩可以确保：

1. 任何保持在⽇志头部以内的使⽤者都将看到所写的每条消息，这些消息将具有顺序偏移量。可以使⽤Topic的min.compaction.lag.ms属性来保证消息在被压缩之前必须经过的最短时间。也就是说，它为每个消息在（未压缩）头部停留的时间提供了⼀个下限。可以使⽤Topic的max.compaction.lag.ms属性来保证从收到消息到消息符合压缩条件之间的最⼤延时
2. 消息始终保持顺序，压缩永远不会重新排序消息，只是删除⼀些⽽已
3. 消息的偏移量永远不会改变，它是⽇志中位置的永久标识符
4. 从⽇志开始的任何使⽤者将⾄少看到所有记录的最终状态，按记录的顺序写⼊。另外，如果使⽤者在⽐Topic的log.cleaner.delete.retention.ms短的时间内到达⽇志的头部，则会看到已删除记录的所有delete标记。保留时间默认是24⼩时。

默认情况下，启动⽇志清理器，若需要启动特定Topic的⽇志清理，请添加特定的属性。配置⽇志清理器，这⾥为⼤家总结了以下⼏点：

1. log.cleanup.policy 设置为 compact ，Broker的配置，影响集群中所有的Topic。

2. log.cleaner.min.compaction.lag.ms ，⽤于防⽌对更新超过最⼩消息进⾏压缩，如果没有设置，除最后⼀个Segment之外，所有Segment都有资格进⾏压缩

   log.cleaner.max.compaction.lag.ms ，⽤于防⽌低⽣产速率的⽇志在⽆限制的时间内不压缩。

Kafka的⽇志压缩原理并不复杂，就是定时把所有的⽇志读取两遍，写⼀遍，⽽CPU的速度超过磁盘完全不是问题，只要⽇志的量对应的读取两遍和写⼊⼀遍的时间在可接受的范围内，那么它的性能就是可以接受的。

#### **2.5.3** 磁盘存储

##### **2.5.3.1** 零拷贝

kafka⾼性能，是多⽅⾯协同的结果，包括宏观架构、分布式partition存储、ISR数据同步、以及“⽆所不⽤其极”的

⾼效利⽤磁盘/操作系统特性。

零拷贝并不是不需要拷贝，⽽是减少不必要的拷贝次数。通常是说在IO读写过程中。

nginx的⾼性能也有零拷⻉的身影。

**传统IO**

⽐如：读取⽂件，socket发送

传统⽅式实现：先读取、再发送，实际经过1~4四次copy。 

```
buffer = File.read
Socket.send(buffer) 
```

1、第⼀次：将磁盘⽂件，读取到操作系统内核缓冲区；

2、第⼆次：将内核缓冲区的数据，copy到application应⽤程序的buffer； 

3、第三步：将application应⽤程序buffer中的数据，copy到socket⽹络发送缓冲区(属于操作系统内核的缓冲区)；

4、第四次：将socket buffer的数据，copy到⽹络协议栈，由⽹卡进⾏⽹络传输。

![image-20211120193423651](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.11.20/image-20211120193423651-7408065.png)

实际IO读写，需要进⾏IO中断，需要CPU响应中断(内核态到⽤户态转换)，尽管引⼊DMA(Direct Memory Access，直接存储器访问)来接管CPU的中断请求，但四次copy是存在“不必要的拷贝”的。

实际上并不需要第⼆个和第三个数据副本。数据可以直接从读缓冲区传输到套接字缓冲区。

kafka的两个过程：

1、⽹络数据持久化到磁盘 (Producer 到 Broker)

2、磁盘⽂件通过⽹络发送（Broker 到 Consumer）

数据落盘通常都是⾮实时的，Kafka的数据并不是实时的写⼊硬盘，它充分利⽤了现代操作系统分页存储来利⽤内存提⾼I/O效率。

磁盘⽂件通过⽹络发送（**Broker** 到 **Consumer**）

磁盘数据通过DMA(Direct Memory Access，直接存储器访问)拷贝到内核态 Buffer

直接通过 DMA 拷贝到 NIC Buffer(socket buffer)，⽆需 CPU 拷贝。

除了减少数据拷贝外，整个读⽂件 ==> ⽹络发送由⼀个 sendfile 调⽤完成，整个过程只有两次上下⽂切换，因此⼤⼤提⾼了性能。

Java NIO对sendfile的⽀持就是FileChannel.transferTo()/transferFrom()。

fileChannel.transferTo( position, count, socketChannel);

把磁盘⽂件读取OS内核缓冲区后的fileChannel，直接转给socketChannel发送；底层就是sendfile。消费者从broker读取数据，就是由此实现。

具体来看，Kafka 的数据传输通过 TransportLayer 来完成，其⼦类 PlaintextTransportLayer 通过Java NIO 的

FileChannel 的 transferTo 和 transferFrom ⽅法实现零拷贝。

注： transferTo 和 transferFrom 并不保证⼀定能使⽤零拷贝，需要操作系统⽀持。

Linux 2.4+ 内核通过 sendfile 系统调⽤，提供了零拷贝。

##### **2.5.3.2** 页缓存

页缓存是操作系统实现的⼀种主要的磁盘缓存，以此⽤来减少对磁盘 I/O 的操作。

具体来说，就是把磁盘中的数据缓存到内存中，把对磁盘的访问变为对内存的访问。

Kafka接收来⾃socket buffer的⽹络数据，应⽤进程不需要中间处理、直接进⾏持久化时。可以使⽤mmap内存⽂件映射。

Memory Mapped Files

简称mmap，简单描述其作⽤就是：将磁盘⽂件映射到内存, ⽤户通过修改内存就能修改磁盘⽂件。

它的⼯作原理是直接利⽤操作系统的Page来实现磁盘⽂件到物理内存的直接映射。完成映射之后你对物理内存的操作会被同步到硬盘上（操作系统在适当的时候）。

通过mmap，进程像读写硬盘⼀样读写内存（当然是虚拟机内存）。使⽤这种⽅式可以获取很⼤的I/O提升，省去了⽤户空间到内核空间复制的开销。

mmap也有⼀个很明显的缺陷：不可靠，写到mmap中的数据并没有被真正的写到硬盘，操作系统会在程序主动调⽤flush的时候才把数据真正的写到硬盘。

Kafka提供了⼀个参数 producer.type 来控制是不是主动flush；

如果Kafka写⼊到mmap之后就⽴即flush然后再返回Producer叫同步(sync)；

写⼊mmap之后⽴即返回Producer不调⽤flush叫异步(async)。

Java NIO对⽂件映射的⽀持

Java NIO，提供了⼀个MappedByteBuffer 类可以⽤来实现内存映射。

MappedByteBuffer只能通过调⽤FileChannel的map()取得，再没有其他⽅式。

FileChannel.map()是抽象⽅法，具体实现是在 FileChannelImpl.map()可⾃⾏查看JDK源码，其map0()⽅法就是

调⽤了Linux内核的mmap的API。使⽤ MappedByteBuffer类要注意的是

mmap的⽂件映射，在full gc时才会进⾏释放。当close时，需要⼿动清除内存映射⽂件，可以反射调⽤sun.misc.Cleaner⽅法。

当⼀个进程准备读取磁盘上的⽂件内容时：

1. 操作系统会先查看待读取的数据所在的⻚ (page)是否在⻚缓存(pagecache)中，如果存在(命中)则直接返回数据，从⽽避免了对物理磁盘的 I/O 操作；

2. 如果没有命中，则操作系统会向磁盘发起读取请求并将读取的数据⻚存⼊⻚缓存，之后再将数据返回给进程。

如果⼀个进程需要将数据写⼊磁盘：

1. 操作系统也会检测数据对应的⻚是否在⻚缓存中，如果不存在，则会先在页缓存中添加相应的页，最后将数据写⼊对应的页。

2. 被修改过后的⻚也就变成了脏⻚，操作系统会在合适的时间把脏⻚中的数据写⼊磁盘，以保持数据的⼀致性。

对⼀个进程⽽⾔，它会在进程内部缓存处理所需的数据，然⽽这些数据有可能还缓存在操作系统的页缓存中，因此同⼀份数据有可能被缓存了两次。并且，除⾮使⽤Direct I/O的⽅式， 否则页缓存很难被禁⽌。

当使⽤页缓存的时候，即使Kafka服务重启， ⻚缓存还是会保持有效，然⽽进程内的缓存却需要重建。这样也极⼤地简化了代码逻辑，因为维护⻚缓存和⽂件之间的⼀致性交由操作系统来负责，这样会⽐进程内维护更加安全有效。

Kafka中⼤量使⽤了页缓存，这是 Kafka 实现⾼吞吐的重要因素之⼀。

消息先被写⼊⻚缓存，由操作系统负责刷盘任务。

##### **2.5.3.3** 顺序写⼊

操作系统可以针对线性读写做深层次的优化，⽐如预读(read-ahead，提前将⼀个⽐较⼤的磁盘块读⼊内存) 和后

写(write-behind，将很多⼩的逻辑写操作合并起来组成⼀个⼤的物理写操作)技术。Kafka 在设计时采⽤了⽂件追加的⽅式来写⼊消息，即只能在⽇志⽂件的尾部追加新的消 息，并且也不允许修改已写⼊的消息，这种⽅式属于典型的顺序写盘的操作，所以就算 Kafka 使⽤磁盘作为存储介质，也能承载⾮常⼤的吞吐量。

mmap和sendfile：

1. Linux内核提供、实现零拷⻉的API；

2. sendfile 是将读到内核空间的数据，转到socket buffer，进⾏⽹络发送；

3. mmap将磁盘⽂件映射到内存，⽀持读和写，对内存的操作会反映在磁盘⽂件上。

4. RocketMQ 在消费消息时，使⽤了 mmap。kafka 使⽤了 sendFile。

Kafka速度快是因为：

1. partition顺序读写，充分利⽤磁盘特性，这是基础；

2. Producer⽣产的数据持久化到broker，采⽤mmap⽂件映射，实现顺序的快速写⼊；

3. Customer从broker读取数据，采⽤sendfile，将磁盘⽂件读到OS内核缓冲区后，直接转到socket buffer进⾏⽹络发送。

## 3. **Kafka**集群与运维

### **3.1** 集群应⽤场景

#### 第**1**节 消息传递

Kafka可以很好地替代传统邮件代理。消息代理的使⽤有多种原因（将处理与数据⽣产者分离，缓冲未处理的消息等）。与⼤多数邮件系统相⽐，Kafka具有更好的吞吐量，内置的分区，复制和容错功能，这使其成为⼤规模邮件处理应⽤程序的理想解决⽅案。

根据我们的经验，消息传递的使⽤通常吞吐量较低，但是可能需要较低的端到端延迟，并且通常取决于Kafka提供的强⼤的持久性保证。

在这个领域，Kafka与ActiveMQ或 RabbitMQ等传统消息传递系统相当。

#### 第**2**节 ⽹站活动路由

Kafka最初的⽤例是能够将⽤户活动跟踪管道重建为⼀组实时的发布-订阅。这意味着将⽹站活动（页⾯浏览，搜索或⽤户可能采取的其他操作）发布到中⼼主题，每种活动类型只有⼀个主题。这些提要可⽤于⼀系列⽤例的订阅，

包括实时处理，实时监控，以及加载到Hadoop或脱机数据仓库系统中以进⾏脱机处理和报告。活动跟踪通常量很⼤，因为每个⽤户⻚⾯视图都会⽣成许多活动消息。

#### 第**3**节 监控指标

Kafka通常⽤于操作监控数据。这涉及汇总来⾃分布式应⽤程序的统计信息，以⽣成操作数据的集中。

#### 第**4**节 ⽇志汇总

许多⼈使⽤Kafka代替⽇志聚合解决⽅案。⽇志聚合通常从服务器收集物理⽇志⽂件，并将它们放在中央位置（也许是⽂件服务器或HDFS）以进⾏处理。Kafka提取⽂件的详细信息，并以⽇志流的形式更清晰地抽象⽇志或事件数据。这允许较低延迟的处理，并更容易⽀持多个数据源和分布式数据消耗。与以⽇志为中⼼的系统（例如Scribe或Flume）相⽐，Kafka具有同样出⾊的性能，由于复制⽽提供的更强的耐⽤性保证以及更低的端到端延迟。

#### 第**5**节 流处理

Kafka的许多⽤户在由多个阶段组成的处理管道中处理数据，其中原始输⼊数据从Kafka主题中使⽤，然后进⾏汇总，充实或以其他⽅式转换为新主题，以供进⼀步使⽤或后续处理。例如，⽤于推荐新闻⽂章的处理管道可能会从RSS提要中检索⽂章内容，并将其发布到“⽂章”主题中。进⼀步的处理可能会使该内容规范化或重复数据删除，并将清洗后的⽂章内容发布到新主题中；最后的处理阶段可能会尝试向⽤户推荐此内容。这样的处理管道基于各个主题创建实时数据流的图形。从0.10.0.0开始，⼀个轻量但功能强⼤的流处理库称为Kafka Streams 可以在Apache Kafka中使⽤来执⾏上述数据处理。除了Kafka Streams以外，其他开源流处理⼯具还包括Apache Storm和 Apache Samza。 

#### 第**6**节 活动采集

事件源是⼀种应⽤程序，其中状态更改以时间顺序记录记录。Kafka对⼤量存储的⽇志数据的⽀持使其成为以这种样式构建的应⽤程序的绝佳后端。

#### 第**7**节 提交⽇志

Kafka可以⽤作分布式系统的⼀种外部提交⽇志。该⽇志有助于在节点之间复制数据，并充当故障节点恢复其数据的重新同步机制。Kafka中的⽇志压缩功能有助于⽀持此⽤法。在这种⽤法中，Kafka类似于Apache BookKeeper项⽬。

1. 横向扩展，提⾼Kafka的处理能⼒
2. 镜像，副本，提供⾼可⽤。

