Starting and Testing Flume
---
To build our flume image that contains our custom config file we do:

1. Navigate to the 'flume' folder that contains the flume Dockerfile.
2. Execute `docker build -t flume .` from the terminal.
3. This will build a new image with the name 'flume'.
4. Run the image with:

```
  docker run -e FLUME_AGENT_NAME=docker -e FLUME_CONF_FILE=/var/tmp/flume.conf -p 44444:44444 -t flume
```
5.  You should see a series of log output with information about sinks and sources being started.
6.  To test things out you can open a new terminal and enter:

```
  echo foo bar baz | nc 127.0.0.1 44444
```
7.  After a slight delay you should see 'foo bar baz' appear in the flume docker container output window.


Running Kafka
---
1.  Run the kafka container with:
```
  docker run -d -p 2181:2181 -p 9092:9092 --env ADVERTISED_HOST=kafka --env ADVERTISED_PORT=9092 --name kafka --hostname kafka spotify/kafka
```
ADVERTISED_HOST was set to 'kafka', which will allow other containers to be able to run Producers and Consumers.

Setting ADVERTISED_HOST to localhost, 127.0.0.1, or 0.0.0.0 will work great only if Producers and Consumers are started within the kafka container itself, or if you are using DockerForMac (like me) and you want to run Producers and Consumers from OSX. These are far less interesting use cases though, so we'll start Producers and Consumers from other containers.

We need to use an IP address or hostname in order for the kafka service to be reachable from another container. IP address is not known before the container is started, so we have to choose a hostname, and I chose kafka in this example.


Creating a topic
---
1.  Create a test topic with:
```
docker exec kafka /opt/kafka_2.11-0.10.1.0/bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
```
2.  Check that the output is 'Created topic "test".'

Note: To get a list of all topics in kafka use:
```
  docker exec kafka /opt/kafka_2.11-0.10.1.0/bin/kafka-topics.sh --list --zookeeper localhost:2181
```

3.  Create a test consumer of the 'test' topic so we can see messages flowing:
```
docker run -it --rm --link kafka spotify/kafka /opt/kafka_2.11-0.10.1.0/bin/kafka-console-consumer.sh --bootstrap-server kafka:9092 --topic test --from-beginning
```

4.  Create a test producer to send things to the 'test' topic:  (in a new terminal window)

```
docker run -it --rm --link kafka spotify/kafka /opt/kafka_2.11-0.10.1.0/bin/kafka-console-producer.sh --broker-list kafka:9092 --topic test
```
5.  Type things in the terminal, they should appear in the window opened in (Step 3)


Networking Docker Containers
----
Its easier to use docker compose to create a network of docker containers, but if you really want to,
you can use these commands to network things old school style:

1.  In a new terminal window, create a network for the two containers to use and connect the containers to the new network:
(docker compose probably does this better)
```
docker network create <network_name> ('opta' for example)
network connect <network_name> <container_id>  (use 'opta' for the network and the container id for the flume container)
network connect <network_name> <container_id>  (use 'opta' for the network and the container id for the kafka container)
```

2.  Test that the flume container can see the opta container:
```
docker exec -it <flume_container_id> ping <kafka_container_id>  (enusre that you can ping the kafka container by the hostname set when the kafka continer was created - the same that is specified in the flume config)
```

3. Open a new terminal and send input to the flume netcat source with the following:
`echo foo bar baz | nc 127.0.0.1 44444`

Useful Commands
---

Note: to get the ip address of a container use:
```
  docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' container_name_or_id
```

Note: to get an interactive terminal inside a running container use:
```
  docker exec -it <containerIdOrName> bash
```
