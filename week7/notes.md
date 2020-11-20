### Kafka (pt 1)

Kafka is another distributed tool, this time it's a distributed event streaming platform.  Kafka works with Spark Streaming and structured streaming as an input source, and is sometimes the only input source to streaming applications.  The reason for this is that Kafka itself communicates well with a wide variety of tools/applications/sources.

### Pub Sub : Publisher Subscriber Design Pattern

Pub sub is a design pattern that is very common.  The recent trend in software development is to write applications as distributed services that communicate, rather than writing one large application that fulfills all your needs by itself.  These smaller services need to communicate with each other.  Services can communicate directly using HTTP (client-server model), where a service sends a request to another service directly and receives a response.  This model is fine, but doesn't work well for information that needs to be distributed across the entire network.  We can see this because communicating one fact to the entire network would require N requests where N is the number of machines in the entire network.  That much traffic isn't going to scale, and organizing that amount of traffic (who needs what info?) is difficult.

A publisher-subscriber communication model involves entities that are *publishers*, entities that are *subscribers* and *topics* or *channels* that communcations are posted to.  A single application or entity can be both a publisher and a subscriber, and a single application or entity can publish/subscribe to multiple channels.

Publishers submit messages/events/some pieces of data to communicate to channels, at which point all the subscribers to those channels are/can be notified of those messages/events/pieces of data.  Publishers don't need to worry about who is subscribed to the channels they publish to, and subscribers don't need to worry about who published the data they're reading -- in pub-sub publishers and subscribers are *decoupled*.

### Message Queues

Related to pub-sub we have messaging queues, another tool for distributed communcation across a network of services.  Pub-sub doesn't assume anything about the use of the messages/events posted to its topics/channels and any number of subscribers can subscribe and will receive all messages/events.

A message queue will similarly take input from anywhere (any publisher), has a specific subject like a topic/channel, and can be read by different consumers.  The difference is that a messaging queue assumes that the messages/events it contains require some resolution.  So instead of messages/events being published to a topic and going out to all subscribers + being saved in that topic's history, messages placed in a messaging queue can be retrieved by a consumer and resolved, which will remove it from the queue.  One handy use case for messaging queues is ensuring consistency among multiple databases/datastores.  A change in one database can trigger publishing messages to a queue which will be resolved by the other databases, changing their data to be consistent.

### Pub-sub and Kafka

Kafka provides pub-sub functionality with *events* that are sent to *topics* by *Producers* and that are read from topics by *consumers*.  We also associate a *log* with each *topic*, with the log containing all the events for that topic.  It's configurable, but we can have Kafka save the complete history forever for our topics, or we can have Kafka maintain history for 1 day, 1 week, ... etc.

The machines that are workers in our distributed Kafka are called *brokers*.  The brokers are responsible for receiving + making availble events, and maintaining resilience/HA of our pub-sub architecture.  Our network of brokers has replications of each topic and a procedure for recovering from failed brokers.

One important distinction is between push and pull based subscriptions.  In pub sub we can have messages/events *pushed* from topics to subscribers, or we can have messages/events *pulled* from topics by subscribers.  Apache Kafka is pull-based pub-sub, so *consumers* must retrieve the data from their topics.  This pull approach works well with the saved *log* per *topic*, because we don't need to worry as much about our consumers failing to pull for some amount of time and missing data.

Another important distinction in pub-sub and other distributed messaging systems is delivery guarantees.  We might want at-least-once delivery, where we ensure messages are received by consumers at least once.  We might want at-most-once delivery, where we ensure messages are received by consumers at most once (no duplicate messages).  We might also want exactly-once delivery, where we ensure messages are recevied exactly once.  This is nice, but harder to achieve than the first 2.  Kafka can achieve all of these, based on configuration.  

at-least-once and at-most-once delivery guarantees happen based on how we treat failing consumers.  If a failure occurs and we re-send the message, we're guaranteeing at-least-once delivery.  If a failure occurs and we don't re-send the message, we're guarateeing at-most-once delivery.

Kafka achieves exactly-once delivery (and/or provides the tools to achieve it) by providing unique identifiers for messages and making message retrieval idempotent.  When Kafka is communicating with outside applications (like our Scala/Spark applications), it requires cooperation from the client application to achieve exactly-once delivery.

### Kafka Stream Processing

In addition to being a platform that lets client applications publish and subscribe, Kafka itself provides tools to manipualte streaming data as it passes through Kafka.  Kafka topics can be used as input to Kafka Stream Processing, which produces output in a different Kafka topic.  Notably, this lets Kafka join data from multiple streams by time, and transform/aggregate streaming data in topics.  Since Kafka controls both the input topic(s) and output topic(s), Kafka provides exactly-once delivery and processing guarantees for Kafka Stream Processing.

Relation to Spark: if we want to transform a Kafka topic into another Kafka topic, our first tool to use would be Kafka Streams.  If that processing/transformation was expensive, we might instead subscribe to the topic with Spark, do the processing on our Spark cluster, and publish the result back to Kafka.

On a more general note, Spark is a tool for Big/Fast data processing.  Kafka is distributed pub/sub used in large networks.  Lots of big companies use Kafka primarily as a way for their microservices to communicate.  We're learning about it because Kafka is widespread and handles a large amount of data, so those same messages sent across the network with Kafka for business use can be pulled into Spark and processed for real time analytics.  Similar to stream transformation, Spark is probably not the first tool your average developer would reach for, it's instead the tool you use when you need big/fast data processing.

### Zookeeper

Zookeeper is a service that maintains information about cluster configuration in the Hadoop ecosystem.  Zookeeper is necessary to run to use Kafka.  This is going to be phased out (in the future you won't need Zookeeper), but for now we still need it.  Zookeeper is used by Kafka when kafka brokers (workers/nodes) fail to help elect new leaders.

### Kafka Partitions

Kafka is built to be scalable, which means its individual topics must be able to scale as well.  The way that Kafka does this is by partitioning each topic.  We choose (or have Kafka choose for us) some value in our events in a topic to use as a key to partition by.  Kafka then hashes the key and assigns the event to the appropriate partition.  All of the events that hash to the same value will be found in the same partition.  Kafka guarantees that events will maintain their order within partitions, but not necessarily between partitions.  This is why we often want to choose the key Kafka uses to partition, so we can maintain exact order in important contexts.

Each partition in Kafka is replicated across the cluster.  We will have a replication factor (number of replications).  We'll have a leader/followers, where the leader is responsible for writes and the followers are used for backup and reads.  When a leader fails, one of the followers is elected as a leader.

Leader and follower here are both brokers, so they're both nodes in Kafka.  They communicate with producers and consumers (who are most often external) and manage the events in topics.

### Events

key value pairs that are ordered in a channel/topic in Kafka.  They are like messages in messaging queues or other pub-sub setups.  We call them events because they represent events occuring somewhere in your distributed system, and they go through Kafka topics so that all the necessary pieces of your distributed system are notified.

One way to think about "events" instead of "messages" is that we want each individual piece of our distributed system to only be responsible for letting the rest of the system know an event has occurred, instead of being responsible for specifying what to do with that event.

Example: FB has servers in the US and Germany due to GDPR requirements, but these two servers need to maintain consistent state.  Liam (US) and Marie(Germany) become friends on facebook, initiated on the German side.
Now, state must become consistent between the two servers, this means the US server needs to catch up with the German server.  To do this, the US server must be notified.  Two ways:

messages:
German server sends "Make Liam + Marie friends" to US server
US server reads the message and does that

events:
German server sends "Liam + Marie became friends" to US Server
US server makes that change as appropriate

With the event approach, the german server is only responsible for notifying what happened, rather than worrying at all about what should be done/who should do it.

^ A bit unwieldly.

Note that events are immutable once they occur.  Events have an order in a topic.  The order may or may not exactly match the order the events were received by kafka.  Within partitions though, the order will match the order events were received by kafka.

