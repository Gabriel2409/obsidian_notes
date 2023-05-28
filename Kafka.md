https://kafka.apache.org/intro

## Overview

### Very high level

- Event streaming technology
- A kafka cluster is composed of several brokers identified by their ID (= bootstrap servers) which know about each other => after connecting to one broker, you are connected to the entire cluster
- Events are organized in topics identified by their names
- Producers send message to a topic and consumers read from it

### Topics

- Topics are divided into partitions.
  - Each partition is stored on a different broker (= bootstrap server)
  - Messages are ordered **within a given partition** with an incremental id called **offset**
- Data is kept for a limited time (default is one week)
- Topics are immutable
- Topics can not be queried directly

- Kafka message anatomy: key, value, compression type, headers, partition + offset, timestamp
- Kafka serialize messages: you need to specify the correct Serializer and Deserializer when Producing / Consuming
- NOTE: the serialization / deserialization type must not change during a topic lifecycle (create a new topic instead)

- Topic replication factor determines how many time the data is replicated. For ex, if there are 3 brokers, each partition can have a replication factor of at most 3.
- At any time only ONE broker can be a leader for a given partition

### Producers

- Producers write data to topics
- If no key is specified, producers will write message accross partitions based on the strategy (for ex RoundRobin will send each message to another partition in cycle, StickyPartitionner will send temporally close data in batch to the same partition, ...)
- If a key is specified, key will be hashed and sent to the corresponding partition. All messages with the same key will go to the same partition
- Producers can only send data to the broker that is the leader of the partition
- Producers can choose to receive acknowledgement of data writes
- Producers will use the correct serializer to convert original object to binary data

#### Important properties

Acknowledgement (acks)
- `acks=0`: producers consider messages as written successfully the moment the message was sent. If the broker goes offline or and exception happens, we can lose data. However, we have the highest throughput
- `acks=1`: producers consider messages as written successfully when it was acknowledged by the leader. Replication is not guaranteed if the broker goes offline before the replication. If the ack is not received, the producer retries the request
- `acks=all (or acks=-1)`: safest kind of guarantee (default in kafka): message must be accepted by all in sync replicas: Producer sends data to leader. Leader sends to all ISR and wait for acknowledgement. Once it is received, leader responds to producer.  
- When using`acks=all`, you must also specify `min.insync.replicas`. For ex, if  `min.insync.replicas=2`, the leader must wait for an ISR to ack before sending ack to producer. Note that `min.insync.replicas` must be lower than `replication.factor`
**To have the safest guarantees: `replication_factor=3` and `min.insync.replicas=2`**

Retries:
- max nb of retries: `retries`, defaults to very high number
- time before next retry:`retry.backoff.ms`, defaults to 100ms 
- timeout on retries: `delivery.timeout.ms`, defaults to 120000=2min. Records will be failed if they are not acknowledged before the timeout

Idempotence:
- In case of network error, we can have duplicate messages (for ex if the message was committed but the ack was not received by the producer, leading to a retry)
- With Idempotent producer, kafka is able to detect if there is a duplicate message. Set the property: `enable.idempotence` to `true`

Compression:

 - Producer side: `compression.type` can be `none`, `gzip`, `lz4`, `snappy`, and `zstd` and is performed by the producer.
 - Compression can also be enable **broker-side**  at the topic level (or for all topics)
	 - default is `compression.type=producer`: writes the compressed message as is
	 - `compression.type=<compression>`: If `<compression>` matches the compression on the broker side, it is written as is. If not, the broker decompress and recompress in the correct format
	 - `compression.type=none`: the broker decompresses the message before storing it
	 - Enabling compression broker-side will consume extra CPU cycles (if it does not match producer compression) to perform the compression

Batching mechanism:
- By default, kafka try to send records as soon as possible. `max.in.flight.requests.per.connection=5` means at most 5 unacknowledged message batches being sent between producer and broker. When all flights are busy, kafka will start batching the incoming messages so that they can be ready for the next batch send.
- `linger.ms` (default 0): how long to wait until we send a batch. Adding a small number helps add more message in the batch but increases latency
- `batch.size`: if a batch is filled before `linger.ms`, send the batch
 
#### Key hashing
Process to mapping a key (when not null) to a partition
- By default, kafka uses the murmur2 algorithm: `targetPartition` = `Murmur2(keyBytes) % (numPartitions)`. Which means adding / removing partitions shifts where a message with the same key goes. It may be better to create a new topic. 


### Consumers

- Consumers read data from a topic. This is a poll model where the consumer requests the data
- Data is read in order from low to high offset **within each partition**

- Consumer will use the correct Deserializer to convert binary data to original object

- Consumers are part of consumer groups: each consumer within a group reads from exclusive partitions. Note that if there are more consumers than partitions, extra will be idle
- You can have multiple consumer groups: each group has its individual consumers reading exclusive partitions
- kafka stores the offsets at which a consumer group has been reading (allows to read from there in case of failure) in a topic `__consumer_offsets`
- Note that if no consumer group is specified, kafka will create one for the individual consumer. This group is temporary

- Rebalance = moving partitions between consumers
	- When
		- a consumer leaves of joins a group
		-  an admin adds a new partition into a topic
	- Strategy:
		- Eager Rebalance: all consumers stop, give up their membership of partition, then rejoin the group and get a new partition assignment (no guarantee they will get back the same partition)
		- Cooperative Rebalance (or Incremental rebalance)
			- Reassign a small subset of the partitions from one consumer to another
			- Other consumers are not interrupted
			- can go through several iterations to find a stable alignment

- Static group membership
	- By default, when a consumer leaves a group, its partitions are revoked and reassigned. It will get a new member ID when rejoining
	- However, we can specify a group.instance.id to make it a static member. If it leaves and comes back within the session.timeout.ms, it will get its partition back (without triggering a rebalance)

#### Important properties

Delivery semantics
- at most once: offsets are committed as soon as the message batch is received. If the processing goes wrong, the message will be lost
- at least once: offsets are committed after the message batch is processed. This can result in duplicate processing.
- exactly once: can be achieved with kafka to kafka workflows using the transactional API. Alternatively, this can be done with the at least once delivery if the consumer is idempotent


## Starting kafka

- Originally shipped with zookeeper (https://zookeeper.apache.org/)

  - Zookeeper manages brokers, helps in performing leader election, sends notifications to kafka in case of changes
  - Zookeeper by design operates with an odd nb of servers
  - Zookeeper does NOT store consumer offsets
  - For kafka clients, do not use zookeeper (deprecated) but instead leverage broker as connection endpoint

- starting from 3.x, kafka can work without zookeeper (with kafka raft)

### Starting Kafka on WSL 2

- add kafka bin folder to path in .bashrc:
  For ex, after installing it in the home directory:

```bash
export KAFKA_PATH="$HOME/kafka_2.13-3.4.0"
export PATH="$KAFKA_PATH/bin:$PATH"
```

After this step CLI should be operationnal

### First option: With zookeeper

- start zookeeper: `zookeeper-server-start.sh $KAFKA_PATH/config/zookeeper.properties`
- start kafka: `kafka-server-start.sh $KAFKA_PATH/config/server.properties`

### Without zookeeper: KRaft Mode

- production ready since Kafka 3.3.1
- generate a new ID for the cluster: `KAFKA_CLUSTER_ID="$(kafka-storage.sh random-uuid)"`
- Format the storage: `kafka-storage.sh format -t $KAFKA_CLUSTER_ID -c $KAFKA_PATH/config/kraft/server.properties`
- start kafka server: `kafka-server-start.sh $KAFKA_PATH/config/kraft/server.properties`

## CLI

### Passing the config

- Most of the commands in the cli need the config file and the bootstrap server. note that the option to pass the config depends on the command.
  Example config `myconfig.config`:

```text
security.protocol=SASL_SSL
sasl.mechanism=PLAIN
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username='user' password='password';
```

- For ex to connect to a cluster on the cloud, you would pass: `--command-config <myconfig.config> --bootstrap-server <mycluster>`
- For localhost (if you started on WSL for ex), no need to pass the config, it will use the default: `--bootstrap-server 127.0.0.1:9092`

### Topics CLI

- all commands start with `kafka-topics.sh --command-config <myconfig.config> --bootstrap-server <mycluster>`
- Create topic: `--create --topic <mytopic>`
  - add partitions: `--partitions 5`
  - specify replication factor: `--replication-factor 2`
- List all topics: `--list`
- Describe a topic: `--topic <mytopic> --describe`
- Delete a topic: `--topic <mytopic> --delete`

### Producer CLI

- all commands start with `kafka-topics.sh --producer.config <myconfig.config> --bootstrap-server <mycluster>` (note that we don't use command-config)
- `--topic <mytopic>`: opens prompt to pass message to topic. If topic does not exist, a new one is created by default but good practice is to disable topic creation that way
- `--property <key>=<value>`: specifies a property
	- Specifying a key: `--property parse.key=true --property key.separator=:`: message is parsed and `:` is used as the separator between key and value
- `--producer-property <key>=<value>`: specifies a producer property. For ex:
  - `acks`: acknowledgement strategy
    - `acks=0`: do not wait for acknowledgement
    - `acks=1`: wait for leader acknowledgement
    - `acks=all`: wait for leader + in sync replicas acknowledgement
  - `partitioner.class`: specify partition strategy for messages without keys
    - `org.apache.kafka.clients.producer.RoundRobinPartitioner`: each message sent to another partition (in a cycle): pretty bad strategy
    - `StickyPartitioner`: addresses the problem of spreading out records without keys into smaller batches by picking a single partition to send all non-keyed records within a batch

### Consumer CLI

- all commands start with `kafka-topics.sh --consumer.config <myconfig.config> --bootstrap-server <mycluster>` (note that we don't use command-config)
- `--topic <topic_name>`: consuming topic. If we produce to it, we'll see the messages
- `--from-beginning`: see all messages from beginning in order (note that there is no notion of ordering cross partition so they may appear in an order different from sent)
- `--property print.partition=true`: see partition nb associated to message
- `--property print.key=true`: see key associated to message
- `--formatter kafka.tools.DefaultMessageFormatter`: select formatter for message output

- group consumers in groups: `--group <group_name>`. If the consumers are stopped, when another consumer of the same group starts, it will automatically consume the messages that were produced in the meantime
- `--from-beginning` will only work within a group if no offset was committed

## Java API

### Java Producer 

#### Basic use 
```java
// Imports
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.common.serialization.StringSerializer; // or other serializer


... 
// specify properties
Properties properties = new Properties()
properties.setProperty("<myprop>", "<myval>") // bootstrap.servers, connection config, etc
// specify serializers
properties.setProperty("key.serializer", StringSerializer.class.getName())
properties.setProperty("value.serializer", StringSerializer.class.getName())

// Create the producer
KafkaProducer<String, String> producer = new KafkaProducer<>(properties);

// Create a record
ProducerRecord<String, String> producerRecord = new ProducerRecord<>("<topic>", "<value>")

// send data -- asynchronous
producer.send(producerRecord);

// flush the producer. Tells the producer to send all data and block until done -- synchronous
producer.flush();

// flush and close the producer
producer.close();
```

#### Better way to specify the config

ProducerConfig can be used to ensure we don't make typos in the config
```java
import org.apache.kafka.clients.producer.ProducerConfig;

properties.setProperty(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
properties.setProperty(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
properties.setProperty(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
```
#### With a key
```java
ProducerRecord<String, String> producerRecord = new ProducerRecord<>("<topic>", "<key>", "<value>")
```

#### With callback
- we can add a callback that is executed when the producer sends a message

```java
producer.send(producerRecord, new Callback() {
	@Override
	public void onCompletion(RecordMetadata metadata, Exception exception) {
		// executed every time record is successfully sent or exception is thrown
		if (exception == null) {
			log.info("Received new metadata \n" +
					"Topic " + metadata.topic() + "\n" +
					"Partition " + metadata.partition() + "\n" +
					"Offset " + metadata.offset() + "\n" +
					"Timestamp " + metadata.timestamp() + "\n"
			);
		} else {
			log.error("Error while producing");
		}

	}

});

```
### Java consumer

#### Basic use

```java
// imports
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.common.serialization.StringDeserializer; // or other deserializer

...

// specify properties
Properties properties = new Properties()
properties.setProperty("<myprop>", "<myval>") // bootstrap.servers, connection config, etc
// specify deserializers
properties.setProperty("key.deserializer", StringDeserializer.class.getName())
properties.setProperty("value.deserializer", StringDeserializer.class.getName())

// add consumer group
properties.setProperty("group.id", groupId);
// specify offset strategy
properties.setProperty("auto.offset.reset", "earliest"); // earliest corresponds to --from-beginning from the cli
// Create the consumer
KafkaConsumer<String,String> consumer = new KafkaConsumer<>(properties);

// subscribe to a topic
consumer.subscribe(Arrays.asList(topic));

// continuously poll data
while (true){
	log.info("polling");
	ConsumerRecords<String,String> records = consumer.poll(Duration.ofMillis(1000)); // we are willing to wait 1 second before each poll
	for(ConsumerRecord<String, String> record: records){
		log.info("Key: " + record.key() + ", Value: " + record.value());
		log.info("Partition: " + record.partition() + ", Offset: " + record.offset());
	}
}
```

#### Better way to pass config

```java
import org.apache.kafka.clients.consumer.ConsumerConfig;
...
properties.setProperty(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
properties.setProperty(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
properties.setProperty(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
properties.setProperty(ConsumerConfig.GROUP_ID_CONFIG, groupId);
properties.setProperty(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "latest");

```

#### Add graceful shutdown

Note: to do a graceful shutdown from cli
- find your pid: `ps wup $(pgrep -f <prog>)`
- `kill -15 <pid>`

```java
// adding the shutdown hook
Runtime.getRuntime().addShutdownHook(new Thread() {
	public void run() {
		log.info("Detecting a shutdown");
		consumer.wakeup(); // will throw a wake up exception on next poll

		// join the main thread to allow the execution of the code in the main thread
		try {
			mainThread.join();
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
});


// wrap the while loop in a try catch
try {
	consumer.subscribe(Arrays.asList(topic));
	while (true) {
		...
	}
} catch (WakeupException e) {
	log.info("Consumer is starting to shut down");
} finally {
	consumer.close(); // closes and commits the offset
	log.info("Consumer gracefully shut down");
}
```
In case of clean shutdown, consumer rebalancing within consumer groups will work (they will be assigned the partitions of the consumer that shut down)

#### Rebalance strategy
In the consumer, set the partition.assignment.strategy property
- Eager strategies:
	- `RangeAssignor`: assign partition on a per topic basis
	- `RoundRobin`: assign partition across all topics so that 
	- `StickyAssignor`: Starts with round robin then mimizes movement
- Cooperative rebalance
	- `CooperativeStickyAssignor`: supports for cooperative rebalance (no need to interrupt all consumers)

```java
import org.apache.kafka.clients.consumer.CooperativeStickyAssignor;
...
properties.setProperty(CooperativeStickyAssignor.class.getName()); 
properties.setProperty("group.instance.id", "...") // only needed for static assignment
```
By default, strategy is `[class org.apache.kafka.clients.consumer.RangeAssignor, class org.apache.kafka.clients.consumer.CooperativeStickyAssignor]`

#### Auto offset commit behavior

- offsets are committed on `poll` call if auto.commit.interval.ms has elapsed
- Example: `auto.commit.interval.ms=5000` and `enable.auto.commit=true`
	- on poll, a timer start.
	- once the 5000ms elapsed, consumer calls `.commitAsync` behind the scences
- At least once scenario: make sure messages are all successfully processed before calling `poll` again
- Note: we can also disable `enable.auto.commit` and from time to time call `.commitSync()` or `.commitAsync()` with the correct offset (advanced use case)



