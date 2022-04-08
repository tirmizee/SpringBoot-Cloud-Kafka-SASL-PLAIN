# SpringBoot-Cloud-Kafka-SASL-PLAIN


### docker-compose.yaml

```yaml

version: "3.8"
services:
  zookeeper:
    container_name: zookeeper
    image: bitnami/zookeeper:3.8
    ports:
      - 2181:2181
    environment:
      - ALLOW_ANONYMOUS_LOGIN=yes
  kafka:
    container_name: kafka
    image: bitnami/kafka:3.1
    ports:
      - '9096:9096'
    environment:
      - KAFKA_BROKER_ID=1
      - KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper:2181
      - ALLOW_PLAINTEXT_LISTENER=yes
      - KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=INTERNAL:PLAINTEXT,EXTERNAL:SASL_PLAINTEXT
      - KAFKA_CFG_LISTENERS=INTERNAL://:9092,EXTERNAL://:9096
      - KAFKA_CFG_ADVERTISED_LISTENERS=INTERNAL://kafka:9092,EXTERNAL://kafka:9096
      - KAFKA_INTER_BROKER_LISTENER_NAME=INTERNAL
    volumes:
      - './.kafka/kafka_jaas.conf:/opt/bitnami/kafka/conf/kafka_jaas.conf'
    depends_on:
      - zookeeper
  kafdrop:
    container_name: kafdrop
    image: obsidiandynamics/kafdrop
    restart: "no"
    ports:
      - "9000:9000"
    environment:
      KAFKA_BROKERCONNECT: "INTERNAL://kafka:9092"
      JVM_OPTS: "-Xms16M -Xmx48M -Xss180K -XX:-TieredCompilation -XX:+UseStringDeduplication -noverify"
    depends_on:
      - "kafka"

```

### application.yaml

```yaml

spring:
  cloud:
    stream:
      binders:
        kafka-plain:
          type: kafka
          environment.spring.cloud.stream.kafka.binder:
            brokers: localhost:9096
            autoAddPartitions: true
            autoCreateTopics: true
            configuration:
              security.protocol: SASL_PLAINTEXT
              sasl.mechanism: PLAIN
              sasl.jaas.config: "org.apache.kafka.common.security.plain.PlainLoginModule required username=\"test\" password=\"testtest\";"
      bindings:
        customerConsumer:
          binder: kafka-plain
          group: Customer.Group
          destination: topic.customer
        customerProducer:
          binder: kafka-plain
          destination: topic.customer

```


### Reference 

- https://stackoverflow.com/questions/67832769/connect-to-kakfka-broker-with-sasl-plaintext-in-docker-compose-binami-kafka
- https://github.com/israelio/kafkajs-sasl-docker-example/blob/master/docker-compose.yml