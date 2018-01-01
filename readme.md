OptaFlume
===

This docker compose image contains Flume and Kafka and the Kakfa Management UI.  
The objective is to take messages from a source (netcat in this case) and move
them to a specified Kafka topic.  

Quickstart
---
1. Make sure the flume image is create by navigating to the 'flume' directory and executing:
    `docker build -t flume .`

2. Navigate to the 'optaflume' root directory and start the components with `docker-compse up`.

3. Make the test topic in Kafka by opening a new terminal
```
docker exec optaflume_middle_1 /opt/kafka_2.11-0.10.1.0/bin/kafka-topics.sh --create --zookeeper optaflume_middle_1:2181 --replication-factor 1 --partitions 1 --topic test
```
4. Make a test consumer that will output anything received on the target kafka topic to the terminal window.
```
docker run -it --rm --link optaflume_middle_1 --net optaflume_default spotify/kafka /opt/kafka_2.11-0.10.1.0/bin/kafka-console-consumer.sh --bootstrap-server optaflume_middle_1:9092 --topic test --from-beginning
```

5. Open an interactive terminal on the flume container and send some message (We need to 'apt-get install netcat' first)
```
  docker exec -it optaflume_flume_1 /bin/bash
  apt-get install netcat
  echo foo bar baz | nc 127.0.0.1 44444
```
6.  Confirm that 'foo bar baz' is shown in the window created in step (2).  This is the standard flume logger recording inputs in flume.
7.  Confirm that 'foo bar baz' is shown in the window created in step (4).  This is the Kafka consumer showing messages received on the test Kafka topic.

If we see output on in the Kafka consumer window then we have confirmed everything is working.


Using the Kafka Manager UI
---
It can be useful to see exactly what's happening with your cluster, so we've installed the
Kakfa Manager Web UI.  You can access this UI by pointing your web browser at 'localhost:9000'.

You'll need to add the cluster manually using the UI.
