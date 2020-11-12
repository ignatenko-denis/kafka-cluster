## Kafka Cluster (2 nodes)

[How to Setup a Kafka Cluster](https://dzone.com/articles/how-to-setup-kafka-cluster)

Changed files:
* /kafka-cluster/kafka_2.13-2.6.0_1/config/server.properties
* /kafka-cluster/kafka_2.13-2.6.0_2/config/server.properties

1. [Download Kafka from Apache's site](https://kafka.apache.org/downloads). 
1. Extract the zip file. Make two copies of the extracted folder. Add the suffix _1, _2 to these folders name. So if your extracted folder name was kafka_2.11-1.1.0, now you will have the folders kafka_2.13-2.6.0_1, kafka_2.13-2.6.0_2. 
1. Go to the kafka_2.13-2.6.0_1 folder.
1. Go to the config directory and open the `server.properties` file. This file contains Kafka broker configurations.
1. Set `broker.id` to 1. This is the id of the broker in a cluster. It must be unique for each broker.
1. Uncomment the `listeners` configuration and set it to `PLAINTEXT://localhost:9091`. This configuration means the broker will be listening on port 9091 for connection requests.
1. Set the `log.dirs` configuration with the logs folder path that we created in step 1.
1. In the `zookeeper.connect` configuration, set the Zookeeper address. If Zookeeper is running in a cluster, then give the address as a comma-separated list, i.e.:localhost:2181,localhost:2182. 
Above are the basic configurations that need to be set up for the development environment but if you want to change some advanced configurations, like retention policy, etc., then I have discussed that in another DZone article, here.

Our first broker configuration is ready. For the other folder or brokers, follow the same steps with the following changes.

In step 5, change `broker.id` to 2.

In step 6, change the `listeners` ports used to 9092 (you can provide any port number that is available).

```ssh
# To clean cluster logs
rm -rf /tmp/kafka-logs-demo-cluster-1 /tmp/kafka-logs-demo-cluster-2 /tmp/zookeeper

# On Node 1. Start Zookeeper
bin/zookeeper-server-start.sh config/zookeeper.properties

# Start Node 1
bin/kafka-server-start.sh config/server.properties

# Start Node 2 (in folder kafka_2.13-2.6.0_2)
bin/kafka-server-start.sh config/server.properties

# On Node 1. Create topic. Where amount of nodes equals replication-factor
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 2 --partitions 5 --topic demo

# On Node 1. Check topic.
bin/kafka-topics.sh --describe --topic demo --zookeeper localhost:2181

# On Node 1. Send message
bin/kafka-console-producer.sh --topic demo --bootstrap-server localhost:9091

# On Node 2. Reseive message (in folder kafka_2.13-2.6.0_2)
bin/kafka-console-consumer.sh --topic demo --from-beginning --bootstrap-server localhost:9091
# or
bin/kafka-console-consumer.sh --topic demo --from-beginning --bootstrap-server localhost:9092

# Then shutdown one of the Nodes and test send/receive messages again.
```

```yml
# To use in Spring Boot in application.yml
spring:
  kafka:
    bootstrap-servers: localhost:9091,localhost:9092
```

Connect to Kafka via [Kafka Tool](https://www.kafkatool.com/) - GUI application for managing and using Apache Kafka clusters.
