Kafka in Docker
===

This repository provides everything you need to run Kafka in Docker.

For convenience also contains a packaged proxy that can be used to get data from
a legacy Kafka 7 cluster into a dockerized Kafka 8.

Why?
---
The main hurdle of running Kafka in Docker is that it depends on Zookeeper.
Compared to other Kafka docker images, this one runs both Zookeeper and Kafka
in the same container. This means:

* No dependency on an external Zookeeper host, or linking to another container
* Zookeeper and Kafka are configured to work together out of the box

Run
---

```bash
docker run -p 2181:2181 -p 9092:9092 --env ADVERTISED_HOST=`docker-machine ip \`docker-machine active\`` --env ADVERTISED_PORT=9092 spotify/kafka
```

```bash
export KAFKA=`docker-machine ip \`docker-machine active\``:9092
kafka-console-producer.sh --broker-list $KAFKA --topic test
```

```bash
export ZOOKEEPER=`docker-machine ip \`docker-machine active\``:2181
kafka-console-consumer.sh --zookeeper $ZOOKEEPER --topic test
```

Running the proxy
-----------------

Take the same parameters as the spotify/kafka image with some new ones:
 * `CONSUMER_THREADS` - the number of threads to consume the source kafka 7 with
 * `TOPICS` - whitelist of topics to mirror
 * `ZK_CONNECT` - the zookeeper connect string of the source kafka 7
 * `GROUP_ID` - the group.id to use when consuming from kafka 7

```bash
docker run -p 2181:2181 -p 9092:9092 \
    --env ADVERTISED_HOST=`boot2docker ip` \
    --env ADVERTISED_PORT=9092 \
    --env CONSUMER_THREADS=1 \
    --env TOPICS=my-topic,some-other-topic \
    --env ZK_CONNECT=kafka7zookeeper:2181/root/path \
    --env GROUP_ID=mymirror \
    spotify/kafkaproxy
```

Docker compose example
----------------------

```bash
version: '3'

services:

  kafka:
    image: kasique/kafka-zookeeper:2.0
    environment:
      ADVERTISED_HOST: 127.0.0.1
      ADVERTISED_PORT: 9092
    ports:
      - 2181:2181 # Zookeper
      - 9092:9092 # Kafka
```


