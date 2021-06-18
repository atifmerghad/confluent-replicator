# Confluent Replicator
## Replicating Data Between Clusters

 1. Start the destination cluster
 2. Start the origin cluster
 3. Create a topic
 4. Configure and run Replicator
 5. Result: test replication



 ### 1. Start the destination cluster
 
 existing aws msk cluster
 
 ### 2. Start the origin cluster
 
 existing confluent cluster
 
 ### 3. Create a topic

Now, lets create a topic named “test-topic” in the origin cluster with the following command:

> /bin/kafka-topics --create --topic test-topic-1 --replication-factor 1 -- partitions 1 --zookeeper server-address:2171

Once we configure and run Replicator, this topic will get replicated to the destination cluster with the exact configuration we defined above. Note that for the sake of this example, we created a topic with just one partition. Replicator will work with any number of topics and partitions.

 ### 4. Configure and run Replicator
 
 We’ll start by configuring the Connect Worker, and then configure Replicator. The Worker configuration file located:
 > confluent/etc/kafka/connect-standalone.properties
Make sure that this configuration matches the settings in the destination cluster with security properties.

```
bootstrap.servers=b-1.address.amazonaws.com:9096,b-2.address.amazonaws.com:9096,b-3.address.amazonaws.com:9096
security.protocol=SASL_SSL 
sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required username="admin" password="*********s";
ssl.endpoint.identifiaction.algorithm=https 
sasl.mechanism=SCRAM-SHA-512
client.bootstrap.servers=b-1.address.amazonaws.com:9096,b-2.address.amazonaws.com:9096,b-3.address.amazonaws.com:9096
client.security.protocol=SASL_SSL 
client.sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginMo dule required username="admin" password="*********"; 
client.ssl.endpoint.identifiaction.algorithm=https
client.sasl.mechanism=SCRAM-SHA-512
producer.sasl.mechanism=SCRAM-SHA-512
producer.security.protocol=SASL_SSL 
producer.sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLogin Module required username="admin" password="********";
consumer.sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLogin Module required username="admin" password="*********"; 
consumer.sasl.mechanism=SCRAM-SHA-512
consumer.security.protocol=SASL_SSL
key.converter=org.apache.kafka.connect.json.JsonConverter 
value.converter=org.apache.kafka.connect.json.JsonConverter 
key.converter.schemas.enable=true 
value.converter.schemas.enable=true 
offset.storage.file.filename=/tmp/connect.offsets offset.flush.interval.ms=10000 
plugin.path=/usr/share/java,/opt/connectors
```

The Replicator configuration file is located in:

> confluent/etc/kafka-connect-replicator/quickstart-replicator.properties

This file contains the information about the source and destination clusters along with the topics to replicate.

```
name=replicator-source 
connector.class=io.confluent.connect.replicator.ReplicatorSourceConnector
key.converter=io.confluent.connect.replicator.util.ByteArrayConverter 
value.converter=io.confluent.connect.replicator.util.ByteArrayConverter 
header.converter=io.confluent.connect.replicator.util.ByteArrayConverter
tasks.max=4

src.kafka.bootstrap.servers=kc-rkjyk-l5762.aws.glb.confluent.cloud:9092 
src.kafka.security.protocol=SASL_SSL 
src.kafka.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLogi nModule required username="test" password=”**************” src.kafka.sasl.mechanism=PLAIN

dest.kafka.bootstrap.servers=b-1.address:9096,b-2.address:9096,b-2.address:9096
dest.kafka.security.protocol=SASL_SSL 
dest.kafka.sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLog inModule required username="test" password="**********"; 
dest.kafka.sasl.mechanism=SCRAM-SHA-512

confluent.topic.replication.factor=1 
topic.whitelist=test-topic-1 
topic.rename.format=${topic}#.replica
```

Run the connector in a standalone Kafka Connect worker:

> ./confluent/bin/connect-standalone confluent/etc/kafka/connect- standalone1.properties confluent/etc/kafka-connect-replicator/quickstart- replicator.properties


 ### 5. Result: test replication
 
 Run a producer to ensure that the source cluster sends data.
 
 >  ./confluent/bin/kafka-console-producer --broker-list lkc-rkjyk-l5768.us- west-2.aws.glb.confluent.cloud:9092 --producer.config producer.properties - -topic test-topic

Send some messages from the console
> > Hello from cluster 1
> > test

Run a consumer to confirm that the destination cluster got the data.

>./confluent/bin/kafka-console-consumer -bootstrap-server b-3.address.amazonaws.com:9096,b-1.address.amazonaws.com:9096,b-2.address.amazonaws.com:9096 --consumer.config consumer2.properties --topic test-topic

Check the console

> > Hello from cluster 1
> > test
