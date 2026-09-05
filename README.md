# Домашнее задание Сартисона Евгения N13 #

Описание/Пошаговая инструкция выполнения домашнего задания:


(1) Запустите RabbitMQ (можно в docker)

(2) Отправьте несколько тем для сообщений через web UI

(3) Прочитайте их, используя web UI в браузере

(4) Отправьте и прочитайте сообщения программно - выберите знакомый язык программирования (C#, Java, Go, Python или любой другой, для которого есть библиотека для работы с RabbitMQ), отправьте и прочитайте несколько сообщений


Для пунктов 2 и 3 сделайте скриншоты отправки и получения сообщений.
Для пункта 4 приложите ссылку на репозитарий на гитхабе с исходным кодом.




## (1) Запустите RabbitMQ (можно в docker) ## 

Использую пример из [otus-nosql](https://github.com/evgnep/otus-nosql) 

Поднял docker контейнер
```
student:~/kafka$ cat docker-compose.yml 

services:
  broker:
    image: apache/kafka:latest
    container_name: broker
    environment:
      KAFKA_NODE_ID: 1
      KAFKA_PROCESS_ROLES: broker,controller
      KAFKA_LISTENERS: PLAINTEXT://localhost:9092,CONTROLLER://localhost:9093
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
      KAFKA_CONTROLLER_LISTENER_NAMES: CONTROLLER
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT
      KAFKA_CONTROLLER_QUORUM_VOTERS: 1@localhost:9093
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      KAFKA_NUM_PARTITIONS: 3
```

проверка статуса
```
student:~/kafka$ docker ps
CONTAINER ID   IMAGE                 COMMAND                  CREATED         STATUS         PORTS      NAMES
cbdedfc54452   apache/kafka:latest   "/__cacert_entrypoin…"   5 minutes ago   Up 5 minutes   9092/tcp   broker
```

Создать топик и проверить что он создался 
```
student:~/kafka$ docker exec -it broker \
  /opt/kafka/bin/kafka-topics.sh \
  --create \
  --bootstrap-server localhost:9092 \
  --replication-factor 1 \
  --partitions 1 \
  --topic my-test-topic
Created topic my-test-topic.

student:~/kafka$ docker exec -it broker /opt/kafka/bin/kafka-topics.sh --list --bootstrap-server localhost:9092
my-test-topic

```



## (2) Отправьте несколько тем для сообщений через web UI ## 

```
student:~/kafka$ echo "testtt 123" | docker exec -i broker /opt/kafka/bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic my-test-topic 
student:~/kafka$ echo "testtt 321" | docker exec -i broker /opt/kafka/bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic my-test-topic
student:~/kafka$ echo "testtt 555521" | docker exec -i broker /opt/kafka/bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic my-test-topic
```

## (3) Прочитайте их, используя web UI в браузере ##

прочитал сообщения
```
student:~/kafka$ docker exec -i broker /opt/kafka/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic my-test-topic 
The consumer rebalance protocol (KIP-848) is production-ready! Set group.protocol=consumer to try it out. See https://kafka.apache.org/documentation/#consumer_rebalance_protocol
testtt 321
testtt 555521
```

## (4) Отправьте и прочитайте сообщения программно ##

Использую след питон скрипт
```
from kafka import KafkaConsumer

# Initialize the consumer and subscribe to the topic
consumer = KafkaConsumer(
    'my-kafka-topic',                       # The Kafka topic
    bootstrap_servers=['localhost:9092'],   # Your Kafka broker address
    group_id='my-test-topic',             # Consumer group ID
    auto_offset_reset='earliest',           # Start from the beginning if no offset exists
    value_deserializer=lambda x: x.decode('utf-8') # Automatically decode bytes to string
)

print("Listening for messages...")

try:
    # The consumer object acts as an infinite iterator
    for msg in consumer:
        print(f"Partition: {msg.value}")

except KeyboardInterrupt:
    print("\nStopping consumer...")
finally:
    # Ensure connections are closed
    consumer.close()
```

отправил сообщение
```
student:~/kafka$ echo "testtt123" | docker exec -i broker /opt/kafka/bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic my-test-topic
```

Прочитаем сообщения из топика
```
python test.py
Listening for messages..
testtt123
```
