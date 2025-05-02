---
title: "How to Implement Kafka in Your Spring Boot Application"
seoTitle: "Kafka Integration with Spring Boot Guide"
seoDescription: "Learn how to integrate Kafka with your Spring Boot application for efficient real-time data streaming and processing"
datePublished: Tue Apr 29 2025 11:53:30 GMT+0000 (Coordinated Universal Time)
cuid: cma2g7y4m000709lg0qoe9lad
slug: how-to-implement-kafka-in-your-spring-boot-application
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1745927554095/dc7fcb80-1977-411a-bcd5-801a0e19a341.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1745927572177/35ad46d9-895c-4195-a09f-2bbb556ad99a.png

---

Hello everyone ,today we will learn about kafka and how can we use it in our spring boot application.

So starting with what is Kafka ?

Kafka is a distributed event streaming platform used for building real time data pipelines and streaming applications. It is designed to handle high-throughput, fault-tolerant, publish-subscribe messaging by storing streams of records (events) in categories called *topics*.

## Before proceeding further we need to understand some terminologies first

**Broker**  
A broker refers to the server running the kafka application , producer sends the message ( data) to a kafka broker and consumer receives the message from the broker.

One system can have either one or many brokers . In case of multiple brokers one broker is elected as controller and the controller broker manages all the information like

* Which partitions are assigned to which broker
    
* Which brokers are alive and dead
    
* Which broker is the leader for which partition
    
* Starting or stopping partition replicas
    
* Handling leader elections if a broker crashes
    

All this information makes the kafka fault tolerant and in case of any broker dies all it’s assigned partitions can be re-assigned to different broker with old information.

**Topics**

**Topic** is a logical namespace used to categorize and organize data streams. You can understand a **topic** as a group of messages of a category . Producers send messages to these **topics** only and a consumer also subscribes to these topics.

Now a Topic is divided into partitions to enable parallelism, scalability, and fault tolerance. These partitions are distributed across various brokers . For example a TOPIC A can have partitions P0 , P1 , P2 and these partitions can be assigned to various brokers like this

P0 → B1 , P1→B1 , P2→B0

When partition of a topics are assigned to various brokers , one of the broker is elected as a leader for a particular partition and all the read and write for that partition happens in that leader broker and other manages the replicas of that partition , so in case of any broker fails data can be retrieved providing fault tolerance.

We can also configure no of partitions for a topic and replication factor for a topic also.

Replication factor of 2 means 2 copies of each partitions in a topic.

**Important points to remember**

1. Many producer can send to the same topic.
    
2. Many consumers can read from the same topic.
    
3. Topics are not directly stored , instead partitions are stored on the broker , and one partition can only be on one broker others are replicas.
    

**Producer**

A Producer is the client which sends records (data) to a topic .One producer can push multiple records to multiple topics and partitions.

**Record**

A record is the data which is sent by the producer , it comprises of three parts

1. Key
    
2. Value ( actual message)
    
3. Timestamp
    

Now we can also configure our producer to send which record to which partition on the basis of this key called partition logic. By default Kafka evenly distributes the record across various partitions.

**Consumer**

A consumer is the component which consumes the messages from a topic. One consumer can subscribe to more than one topic independently .

Kafka uses the concept of consumer groups , different consumers are grouped into a single consumer group and within that consumer group partitions are distributed across various consumers.

Ideally all the consumers within a consumer group should subscribe to same set of topics to ensure efficient partition balancing.

Within a consumer group each partition of topic can only be read by one consumer , but different consumer groups can read same partition hence we can say that consumer groups are independent of each other.

* Kafka allows **consumer groups to work independently**. This means that each consumer group gets its own view of the data, and the consumers in different groups can consume from the **same partition** without affecting each other.
    
* **Each consumer group maintains its own offset**, so it can read messages from a partition independently of other consumer groups, and each group can keep track of its own position in the partition (i.e., which message it has processed).
    

**Offset**

An **Offset** is a **unique identifier** for each message within a partition. Each consumer groups tracks offsets for each partition and consumers read messages according to that offset value for the partition , on a successfull read it is the consumer who will commit the offset and will update the value in the consumer offset group maintained by the consumer group.

**Kafka Cluster**

A **Kafka cluster** is a collection of **Kafka brokers** (servers) that work together to manage and process streams of data in Kafka. It is the central unit of Kafka's architecture, enabling scalability, fault tolerance, and high availability. The cluster manages the **topics**, **partitions**, and **consumer groups** and distributes data across multiple brokers.

This kafka cluster is managed by Zookeeper ( in earlier versions) and inside the kafka cluster information like partition leaders is managed by the controller broker .

So when a controller broker dies , zookeeper will be notified and it will elect a new controller and when a partition leader dies controller broker will elect a new leader for that topic.

The controller broker also manages **partition assignments** across brokers. When new brokers join the cluster or existing brokers leave, the controller broker is responsible for rebalancing partitions among the brokers to distribute the load evenly.

The controller broker is responsible for overseeing **replication**. It ensures that the appropriate number of replicas are maintained for each partition and coordinates the synchronization of replicas across different brokers.

One important thing is the controller broker is dependent on zookeeper for the state information of each broker .

Each broker sends regular information to zookeeper that bro i am alive (formally called heartbeats) and the controller broker sets a watch on the partition leaders so whenever a partition leader fails , controller will get information and it will re elect the partition leader and also rebalance the partition distribution.

😮‍💨 I think it’s enough of theory now we will start practical implementation.

I will be using docker image for kafka and zookeeper you can also set them up locally.

**Starting with docker image these are the images which we will use**

kafka `confluentinc/cp-kafka:latest`

zookeeper `confluentinc/cp-zookeeper:latest`

Here i will not be telling how to setup and use kafka using docker you can use either docker-compose file or directly run the containers.

Now once the docker container starts let’s jump to the spring boot application ,

In our spring boot application we will need a dependency for kafka , so starting with setting up our application. Go to [start.spring.io](http://start.spring.io) and create a new project with these configurations and dependencies.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745917035240/af0ed1a4-0a65-4770-8e16-c06faf59b069.png align="center")

And just click on generate project a zip file will be downloaded and it will contain your starter project.

# Producer

Now for connecting our producer application with Kafka we need to tell our application that our kafka server is running there and we can do it by two ways either by defining the configuration properties in application.properties file ( or application.yml) or we can also configure them via code by creating beans of Consumer and Producer factory in consumer and producer respectively. We will cover both starting with application.properties file

In our application.properties (or application.yml) file we need to add these configurations

```java
spring.kafka.bootstrap-servers=${KAFKA_URL:localhost:9092}
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.producer.value-serializer=com.xyz.abc.config.kafka.KafkaDTOSerializer
```

Now if you will notice there are two more things in producer config (i.e serializers) we need to define the Key serializer and the Value Serializer ( either we can use default which are String serializer ) .

For the key and value serializer either we can use Json Serializer or we create our own , here i have created my own but if you want you can also use JsonSerializer. For key serialization we generally use default StringSerializer.

```java
spring.kafka.producer.value-serializer=org.springframework.kafka.support.serializer.JsonSerializer
```

But remember if we use JsonSerializer we have to configure the trusted packages ( package of our POJO class) in the consumer like this

```java
spring.kafka.consumer.properties.spring.json.trusted.packages=your.package.name
```

## Creating our own serializer

```java

import com.fasterxml.jackson.databind.ObjectMapper;
import org.apache.kafka.common.serialization.Serializer;

public class KafkaDTOSerializer implements Serializer<MessageDTOO> {
    private final ObjectMapper objectMapper = new ObjectMapper();
    @Override
    public byte[] serialize(String s, MessageDTO messageDTO) {
       try {
        return objectMapper.writeValueAsBytes(messageDTO);
       }
       catch (Exception e) {
           throw new RuntimeException("Error serializing MessageDTO", e);
       }
    }
}
```

and that’s it we just need to give the path of this file in the configuration with package name.

## Creating topics

We can either create in the kafka itself using cli or we can do it by code also. These annotations are must.

```java
@Configuration
public class KafkaConfig {
    @Bean
    public NewTopic topicBuilder() {
        return TopicBuilder.name("topic-name")
                .build();
    }
}
```

## Creating a producer

Now finally we need to create the producer

```java
@Service
@RequiredArgsConstructor
public class KafkaProducer {
    private final KafkaTemplate<String, MessageDTO> kafkaTemplate;

    public void produce(String topic, URLClickEventDTO messageDTO) {
        Message<MessageDTO> message = MessageBuilder.withPayload(messageDTO)
                .setHeader(KafkaHeaders.TOPIC,"your-topic-name")
                .setHeader(KafkaHeaders.KEY,messageDTO.getId())
                .build();
        try{
            CompletableFuture<SendResult<String, MessageDTO>> future = kafkaTemplate.send(message);

            future.whenComplete((result, ex) -> {
                if (ex != null) {
                    System.out.println("Failed to send message: " + ex.getMessage());
                } else {
                    System.out.println(result.getProducerRecord());
                    System.out.println("Successfully sent to partition: " + result.getRecordMetadata().partition());
                }
            });
        }
        catch(Exception e){
            e.printStackTrace();
        }


    }
}
```

Let us define some of the terms used here,

**KafkaTemplate&lt;K,V&gt;**

`KafkaTemplate<K, V>` is a **high-level helper class** provided by **Spring for Apache Kafka** (`spring-kafka` module) that simplifies the **process of sending messages from a Spring Boot application to a Kafka topic**.

It wraps the low-level **KafkaProducer** API provided by Apache Kafka and integrates it with the **Spring messaging model**, offering both **asynchronous** and **synchronous** send capabilities along with full support for headers, partitions, custom serializers, and transactional messaging.

Think of `KafkaTemplate` as your **main tool to send data to Kafka** from Spring Boot.  
It wraps all the Kafka producer complexity in an easy-to-use class.

**Message&lt;T&gt;**

`Message<T>` is a **core interface** from Spring’s `org.springframework.messaging` package that represents a **generic message** carrying two components:

1. A **payload** (the actual data you want to send or receive)
    
2. A **headers map** (key-value metadata about the message)
    

This abstraction is used across Spring Messaging, including **Spring Integration**, **Spring WebSocket**, **Spring Kafka**, etc.

It is optional to use , it just provides us ability send custom headers.

Now we are good to go to produce messages , we just need to call the `produce()` function of this class which is annotated with `@Service`

# Consumer

Again we have to create a project using spring initializer for our consumer with same dependencies.

## Properties

```java
spring.kafka.consumer.bootstrap-servers=${KAFKA_URL:localhost:9092}
spring.kafka.consumer.group-id=consumer-group-id
spring.kafka.consumer.auto-offset-reset=earliest
spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.value-deserializer=com.xyz.abc.config.kafka.KafkaDeserializer
```

1. `spring.kafka.consumer.bootstrap-servers=${KAFKA_URL:localhost:9092}`
    

* This tells your Kafka consumer which Kafka broker(s) to connect to.
    
* `${KAFKA_URL:localhost:9092}` is an **environment variable fallback syntax**:
    
    * If `KAFKA_URL` is defined in env or `.env` file, it uses that.
        
    * Else it falls back to `localhost:9092`.
        

**In plain words:**  
“Connect to the Kafka broker on `localhost:9092` unless an external `KAFKA_URL` is set.”

2. `spring.kafka.consumer.group-id=consumer-group-id`
    

* This sets the **Kafka consumer group ID**.
    
* Kafka uses this to track what messages have already been read by this group.
    

3. `spring.kafka.consumer.auto-offset-reset=earliest`
    

* If Kafka **doesn’t find a stored offset** for this group (like on first run), it decides where to start consuming from:
    
    * `earliest`: start from **beginning of the topic**
        
    * `latest`: start from **latest (newest) messages only**
        

**Common use case:**  
You use `earliest` during development or analytics to reprocess all past messages.

**In plain words:**  
“If there's no stored offset, start consuming from the beginning of the topic.”

4. `spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer`
    
    * Kafka messages come in as **byte arrays**.
        
    * You need a **deserializer** to convert the key (usually a string like `"user-123"` or UUID) back into a usable type.
        
    * This uses the **built-in Kafka** `StringDeserializer` to turn byte\[\] into a `String`.
        
5. `spring.kafka.consumer.value-deserializer=com.package.KafkaDeserializer`
    
    * This tells Kafka how to deserialize the **value (body)** of the message.
        
    * Here we need to specify our custom deserializer or the JsonDeserializer
        

## DeSerializer

```java
public class KafkaDeserializer<T> implements Deserializer<T> {
    private Class<T> clazz;
    public KafkaDeserializer(Class<T> clazz) {
        this.clazz = clazz;
    }
    private final ObjectMapper mapper = new ObjectMapper();
    @Override
    public T deserialize(String s, byte[] bytes) {
        if (bytes == null) {
            return null;
        }
        try {
            T dto = mapper.readValue(bytes,clazz);
            return dto;
        }
        catch (Exception e) {
            System.out.println("❌ Failed to deserialize: " + new String(bytes));
            e.printStackTrace();
            throw new RuntimeException("Error de serializing the dto", e);

        }
    }
}
```

## Listener

```java
@Component
public class KafkaConsumer {
    @KafkaListener(topics = "your-topic-name", groupId = "consumer-group-id")
    public void listen(MessageDTO h) {
         // process your message dto
  }
}
```

And that’s it now you can run both the applications and as you will produce the message from the producer we will be able to listen it on the consumer.

You can simply produce the message from a rest controller or any where .

# Custom Producer and Consumer Factories

Another way to configure properties of our producer and consumers is creating a Bean of ConsumerFactory and ProducerFactory.

First of all we use these when we need custom configurations or we have multiple Kafka producers , consumers in one project.

Now What are these ?

### `ProducerFactory<K, V>`

* It’s an **interface** that Spring Kafka uses to **create Kafka producers**.
    
* You give it your configuration (like serializers, broker URL, etc.), and it gives you a new `KafkaProducer` instance as needed.
    

### `ConsumerFactory<K, V>`

* Similar thing, but for **creating Kafka consumers**.
    
* It creates instances of `KafkaConsumer` with the config you specify.
    

Sample ConsumerConfig using ConsumerFactory

```java
@Configuration
public class KafkaConsumerConfig {

    // 🟢 Consumer Factory for Waste Data
    @Bean
    public ConsumerFactory<String, WasteDTO> wasteConsumerFactory() {
        Map<String, Object> props = new HashMap<>();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(ConsumerConfig.GROUP_ID_CONFIG, "waste-group");
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, WasteDTOSerializer.class);
        return new DefaultKafkaConsumerFactory<>(props);
    }

    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, WasteDTO> wasteKafkaListenerContainerFactory() {
        ConcurrentKafkaListenerContainerFactory<String, WasteDTO> factory = new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(wasteConsumerFactory());
        return factory;
    }

    // 🔵 Consumer Factory for Status updates
    @Bean
    public ConsumerFactory<String, StatusDTO> walletConsumerFactory() {
        Map<String, Object> props = new HashMap<>();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(ConsumerConfig.GROUP_ID_CONFIG, "wallet-group");
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StatusDeSerializer.class);
        return new DefaultKafkaConsumerFactory<>(props);
    }

    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, StatusDTO> walletKafkaListenerContainerFactory() {
        ConcurrentKafkaListenerContainerFactory<String, StatusDTO> factory = new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(walletConsumerFactory());
        return factory;
    }
}
```

Sample Producer Factory

```java
@Configuration
public class KafkaProducerConfig {

    //  Producer for Waste Data
    @Bean
    public ProducerFactory<String, WasteDTO> wasteProducerFactory() {
        Map<String, Object> configProps = new HashMap<>();
        configProps.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        configProps.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        configProps.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, WasteDTOSerializer.class);
        return new DefaultKafkaProducerFactory<>(configProps);
    }

    @Bean
    public KafkaTemplate<String, WasteDTO> wasteKafkaTemplate() {
        return new KafkaTemplate<>(wasteProducerFactory());
    }

    //  Producer for Wallet Data
    @Bean
    public ProducerFactory<String, WalletUpdateDTO> walletProducerFactory() {
        Map<String, Object> configProps = new HashMap<>();
        configProps.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        configProps.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        configProps.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, WalletSerializer.class);
        return new DefaultKafkaProducerFactory<>(configProps);
    }

    @Bean
    public KafkaTemplate<String, WalletUpdateDTO> walletKafkaTemplate() {
        return new KafkaTemplate<>(walletProducerFactory());
    }

    @Bean
    public ProducerFactory<String, StatusDTO> statusProducerFactory() {
        Map<String, Object> configProps = new HashMap<>();
        configProps.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        configProps.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        configProps.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StatusSerializer.class);
        return new DefaultKafkaProducerFactory<>(configProps);
    }

    @Bean
    public KafkaTemplate<String, StatusDTO>statusKafkaTemplate() {
        return new KafkaTemplate<>(statusProducerFactory());
    }
}
```

If you use this way for configuring we just need to add these Configuration files which will declare beans of ConsumerFactory and ProducerFactory and the rest of the code and configuration will remain same.

That’s it for this article , hope you liked it will see you next time, if you have any questions you can ask them , I will try my best to answer them.