docker-zookeeper
================

Builds a docker image for Zookeeper.

```docker build -t <user>/zookeeper:3.4.13 .```


About Kafka
Kafka is Fast, Scalable, Durable, and Fault-Tolerant publish-subscribe messaging system which can be used to real time data streaming. We can introduce Kafka as Distributed Commit Log which follows publish subscribe architecture. Like many publish-subscribe messaging systems, Kafka maintains feeds of messages in topics. Producers write data to topics and consumers read from topics. Since Kafka is a distributed system, topics are partitioned and replicated across multiple nodes.

Messages in Kafka are simple byte arrays(String , JSON etc). When publishing, message can be attached to a key. Producer distributes all the messages with same key into same partition.

About Zookeeper
Zookeeper is distributed systems configuration management tool. Zookeeper provides multiple features for distributed applications like distributed configuration management, leader election, consensus handling, coordination and locks etc.

Zookeeper storing its data on tree like structure. It introduced as Data Tree and nodes introduced as zNodes. Zookeeper follows standard unix notations for file paths. For an example /A/B/C denote the path to zNode C, where C has B as its parent and B has A as its parent.

Read more about Zookeeper from here.

Kafaka with Zookeeper
Distributed systems required to do coordination tasks, configuration managements, state managements etc. Some distributed systems built their own tools/mechanism to manage these tasks. Cassandra, Elasticsearch, Mongodb are some examples for these kind of systems. Other distributed systems such as Kafka, Storm, HBase chosen existing configuration management tools like Zookeeper, Etcd etc

Kafka uses Zookeeper to mange following tasks

Electing a controller - The controller is one of the brokers and is responsible for maintaining the leader/follower relationship for all the partitions. When a node shuts down, it is the controller that tells other replicas to become partition leaders to replace the partition leaders on the node that is going away. Zookeeper is used to elect a controller, make sure there is only one and elect a new one it if it crashes.
Cluster membership - Which brokers are alive and part of the cluster? this is also managed through ZooKeeper.
Topic configuration - Which topics exist, how many partitions each has, where are the replicas, who is the preferred leader, what configuration overrides are set for each topic
Manage Quotas - How much data is each client allowed to read and write
Access control - Who is allowed to read and write to which topic (old high level consumer). Which consumer groups exist, who are their members and what is the latest offset each group got from each partition.
Run Kafka and Zookeeper with docker
1. Run zookeeper
docker run -d \
--name zookeeper \
-p 2181:2181 \
jplock/zookeeper

2. Run kafka
docker run -d \
--name kafka \
-p 7203:7203 \
-p 9092:9092 \
-e KAFKA_ADVERTISED_HOST_NAME=10.4.1.29 \
-e ZOOKEEPER_IP=10.4.1.29 \
ches/kafka

KAFKA_ADVERTISED_HOST_NAME is the IP address of the machine(my local machine) which Kafka container running. ZOOKEEPER_IP is the Zookeeper container running machines IP. By this way producers and consumers can access Kafka and Zookeeper by using that IP address.

3. Create topic
docker run \
--rm ches/kafka kafka-topics.sh \
--create \
--topic senz \
--replication-factor 1 \
--partitions 1 \
--zookeeper 10.4.1.29:2181
Kafka container’s bin directory contains some shell scripts which can use to manage topics, consumers, publishers etc. By using kafka-topics.sh script I’m creating a topic named senz. 10.4.1.29:2181 is the zookeeper running host and port.


4. List topics
docker run \
--rm ches/kafka kafka-topics.sh \
--list \
--zookeeper 10.4.1.29:2181
This command will list all available topics.


5. Create publisher
docker run --rm --interactive \
ches/kafka kafka-console-producer.sh \
--topic senz \
--broker-list 10.4.1.29:9092
This command will creates a producer for senz topic. This producer will take inputs from command line and publish them to Kafka. broker-list 10.4.1.29:9092 specifies the host and port of Kafka.


6. Create consumer
docker run --rm \
ches/kafka kafka-console-consumer.sh \
--topic senz \
--from-beginning \
--zookeeper 10.4.1.29:2181
This command will create consumer for senz topic. This consumer will take the messages from topic and output into the command line. from-beginning parameter specifies to take the messages from the beginning of the topic. zookeeper 10.4.1.29:2181 specifies the host and port of Zookeeper.


Kafka related data in Zookeeper
We can go inside to Zookeeper container and see how Kafka related data(broker, publisher details) are stored.

1. Go inside Zookeeper container
docker exec -it zookeeper bash
Inside the bin directory we can find the commands which are available to manage Zookeeper


2. Connect to Zookeeper server
bin/zkCli.sh -server 127.0.0.1:2181
Since we are inside the Zookeeper container, we can specify server address as the localhost(127.0.0.1).


3. List root
ls /

4. List brokers
ls /brokers

5. List topics
ls /brokers/topics

6. List consumers
ls /consumers

7. List consumer owner
ls /consumers/console-consumer-1532/owners

Reference
https://www.quora.com/What-is-the-actual-role-of-Zookeeper-in-Kafka-What-benefits-will-I-miss-out-on-if-I-don%E2%80%99t-use-Zookeeper-and-Kafka-together
https://stackoverflow.com/questions/23751708/kafka-is-zookeeper-a-must
https://blog.cloudera.com/blog/2014/09/apache-kafka-for-beginners/
http://cloudurable.com/blog/what-is-kafka/index.html
