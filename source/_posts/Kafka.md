---
title: Kafka
tags: Kafka
categories: Kafka
---


### Kafka 具有3大功能:
- 1.Publish & Subscribe: to streams of data like a messaging system
- 2.Process: streams of data efficiently
- 3.Store: streams of data safely in a distributed replicated cluster

![](/images/Kafka_1.png)

![](/images/Kafka_2.png)

### Producer & Consumer 导图
![](/images/Kafka_3.png)

![](/images/Kafka_4.png)

![](/images/Kafka_5.png)

### Kafka four core APIs
Kafka has four core APIs:
- The Producer API allows an application to publish a stream records to one or more Kafka topics.
- The Consumer API allows an application to subscribe to one or more topics and process the stream of records produced to them.
- The Streams API allows an application to act as a stream processor, consuming an input stream from one or more topics and producing an output stream to one or more output topics, effectively transforming the input streams to output streams.
- The Connector API allows building and running reusable producers or consumers that connect Kafka topics to existing applications or data systems.

### Producer APIs

_kafka.producer.SyncProducer and kafka.producer.async.AsyncProducer._
``` java
class Producer {

  /* Sends the data, partitioned by key to the topic using either the */
  /* synchronous or the asynchronous producer */
  public void send(kafka.javaapi.producer.ProducerData<K,V> producerData);

  /* Sends a list of data, partitioned by key to the topic using either */
  /* the synchronous or the asynchronous producer */
  public void send(java.util.List<kafka.javaapi.producer.ProducerData<K,V>> producerData);

  /* Closes the producer and cleans up */
  public void close();

}
```
- __can handle queueing/buffering of multiple producer requests and asynchronous dispatch of the batched data__
`kafka.producer.Producer` provides the ability to batch multiple produce requests (producer.type=async), before serializing and dispatching them to the appropriate kafka broker partition. The size of the batch can be controlled by a few config parameters. As events enter a queue, they are buffered in a queue, until either queue.time or batch.size is reached. A background thread (`kafka.producer.async.ProducerSendThread`) dequeues the batch of data and lets the `kafka.producer.EventHandler` serialize and send the data to the appropriate kafka broker partition. A custom event handler can be plugged in through the event.handler config parameter. At various stages of this producer queue pipeline, it is helpful to be able to inject callbacks, either for plugging in custom logging/tracing code or custom monitoring logic. This is possible by implementing the `kafka.producer.async.CallbackHandler` interface and setting `callback.handlerconfig` parameter to that class.

- __handles the serialization of data through a user-specified Encoder:__
```
interface Encoder<T> {
  public Message toMessage(T data);
}
```
The default is the no-op `kafka.serializer.DefaultEncoder`

- __provides software load balancing through an optionally user-specified Partitioner:__

The routing decision is influenced by the kafka.producer.Partitioner.
```
interface Partitioner<T> {
   int partition(T key, int numPartitions);
}
```

### Consumer APIs
``` java
class SimpleConsumer {

  /* Send fetch request to a broker and get back a set of messages. */
  public ByteBufferMessageSet fetch(FetchRequest request);

  /* Send a list of fetch requests to a broker and get back a response set. */
  public MultiFetchResponse multifetch(List<FetchRequest> fetches);

  /**
   * Get a list of valid offsets (up to maxSize) before the given time.
   * The result is a list of offsets, in descending order.
   * @param time: time in millisecs,
   *              if set to OffsetRequest$.MODULE$.LATEST_TIME(), get from the latest offset available.
   *              if set to OffsetRequest$.MODULE$.EARLIEST_TIME(), get from the earliest offset available.
   */
  public long[] getOffsetsBefore(String topic, int partition, long time, int maxNumOffsets);
}

/* create a connection to the cluster */
ConsumerConnector connector = Consumer.create(consumerConfig);

interface ConsumerConnector {

  /**
   * This method is used to get a list of KafkaStreams, which are iterators over
   * MessageAndMetadata objects from which you can obtain messages and their
   * associated metadata (currently only topic).
   *  Input: a map of <topic, #streams>
   *  Output: a map of <topic, list of message streams>
   */
  public Map<String,List<KafkaStream>> createMessageStreams(Map<String,Int> topicCountMap);

  /**
   * You can also obtain a list of KafkaStreams, that iterate over messages
   * from topics that match a TopicFilter. (A TopicFilter encapsulates a
   * whitelist or a blacklist which is a standard Java regex.)
   */
  public List<KafkaStream> createMessageStreamsByFilter(
      TopicFilter topicFilter, int numStreams);

  /* Commit the offsets of all messages consumed so far. */
  public commitOffsets()

  /* Shut down the connector */
  public shutdown()
}
```

### Kafka ZooKeeper文件目录
__Broker Node Registry:__
```
/brokers/ids/[0...N] --> {"jmx_port":...,"timestamp":...,"endpoints":[...],"host":...,"version":...,"port":...} (ephemeral node)
```
__Broker Topic Registry:__
```
/brokers/topics/[topic]/partitions/[0...N]/state --> {"controller_epoch":...,"leader":...,"version":...,"leader_epoch":...,"isr":[...]} (ephemeral node)
```
__Consumer Id Registry:__
```
/consumers/[group_id]/ids/[consumer_id] --> {"version":...,"subscription":{...:...},"pattern":...,"timestamp":...} (ephemeral node)
```
__Consumer Offsets:__
```
/consumers/[group_id]/offsets/[topic]/[partition_id] --> offset_counter_value ((persistent node)
```
__Partition Owner registry:__
```
/consumers/[group_id]/owners/[topic]/[partition_id] --> consumer_node_id (ephemeral node)
```
