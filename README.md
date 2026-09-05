# Домашнее задание Сартисона Евгения N13 #

Описание/Пошаговая инструкция выполнения домашнего задания:


(1) Запустите RabbitMQ (можно в docker)

(2) Отправьте несколько тем для сообщений через web UI

(3) Прочитайте их, используя web UI в браузере

(4) Отправьте и прочитайте сообщения программно - выберите знакомый язык программирования (C#, Java, Go, Python или любой другой, для которого есть библиотека для работы с RabbitMQ), отправьте и прочитайте несколько сообщений


Для пунктов 2 и 3 сделайте скриншоты отправки и получения сообщений.
Для пункта 4 приложите ссылку на репозитарий на гитхабе с исходным кодом.




## (1) Запустите RabbitMQ (можно в docker) ## 

Использую пример из [How to Run RabbitMQ in Docker Compose](https://medium.com/@kaloyanmanev/how-to-run-rabbitmq-in-docker-compose-e5baccc3e644) 

Поднял docker контейнер
```
services:
  rabbitmq:
    image: rabbitmq:latest
    container_name: rabbitmq
    restart: always
    ports:
      - 5672:5672
      - 15672:15672
    environment:
      RABBITMQ_DEFAULT_USER: kalo
      RABBITMQ_DEFAULT_PASS: kalo
    configs:
      - source: rabbitmq-plugins
        target: /etc/rabbitmq/enabled_plugins
    volumes:
      - rabbitmq-lib:/var/lib/rabbitmq/
      - rabbitmq-log:/var/log/rabbitmq

configs:
  rabbitmq-plugins:
    content: "[rabbitmq_management]."  

volumes:
  rabbitmq-lib:
    driver: local
  rabbitmq-log:
    driver: local
```

проверка статуса
```
student:~$ docker ps
CONTAINER ID   IMAGE             COMMAND                  CREATED          STATUS          PORTS                                                                                                                                          NAMES
8a5da38db5b5   rabbitmq:latest   "docker-entrypoint.s…"   37 seconds ago   Up 34 seconds   4369/tcp, 0.0.0.0:5672->5672/tcp, [::]:5672->5672/tcp, 5671/tcp, 15691-15692/tcp, 25672/tcp, 0.0.0.0:15672->15672/tcp, [::]:15672->15672/tcp   rabbitmq

```

UI доступен по http://localhost:15672/
<img width="1185" height="741" alt="image" src="https://github.com/user-attachments/assets/5acdb347-9752-4172-aae8-aee40d7a66c8" />




## (2) Отправьте несколько тем для сообщений через web UI ## 

создал несколько очередей
<img width="615" height="460" alt="image" src="https://github.com/user-attachments/assets/28afdef2-9cb2-4c01-9137-5da938efc600" />

отправить сообщение со вкладки Exchanges(AMQP default)
<img width="844" height="826" alt="image" src="https://github.com/user-attachments/assets/89fc24eb-1d4a-4ae1-bdfb-ca0dfac9cf56" />


## (3) Прочитайте их, используя web UI в браузере ##
<img width="680" height="789" alt="image" src="https://github.com/user-attachments/assets/7e4fd1d6-c8e1-4bfa-bc0a-c038b16acd7c" />
прочитал сообщения в очереди queue-any


## (4) Отправьте и прочитайте сообщения программно ##

прочитал сообщения через CURL
```
student:~$ curl -u kalo:kalo \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{"count":2, "ackmode":"ack_requeue_true", "encoding":"auto"}' \
  http://localhost:15672/api/queues/%2F/queue-any/get
[{"payload_bytes":19,"redelivered":true,"exchange":"","routing_key":"queue-any","message_count":1,"properties":{"delivery_mode":2,"headers":{}},"payload":"testing 1st message","payload_encoding":"string"},{"payload_bytes":19,"redelivered":true,"exchange":"","routing_key":"queue-any","message_count":0,"properties":{"delivery_mode":2,"headers":{}},"payload":"testing 2nd message","payload_encoding":"string"}]
```
