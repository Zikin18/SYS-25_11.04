# Домашнее задание к занятию  «Очереди RabbitMQ» - Юрий Белобородов


### Задание 1. Установка RabbitMQ

Используя Vagrant или VirtualBox, создайте виртуальную машину и установите RabbitMQ.
Добавьте management plug-in и зайдите в веб-интерфейс.

*Итогом выполнения домашнего задания будет приложенный скриншот веб-интерфейса RabbitMQ.*

Ответ:

![1-1_setup.png](https://github.com/Zikin18/SYS-25_11.04/blob/master/1-1_setup.png)


### Задание 2. Отправка и получение сообщений

Используя приложенные скрипты, проведите тестовую отправку и получение сообщения.
Для отправки сообщений необходимо запустить скрипт producer.py.

Для работы скриптов вам необходимо установить Python версии 3 и библиотеку Pika.
Также в скриптах нужно указать IP-адрес машины, на которой запущен RabbitMQ, заменив localhost на нужный IP.

```shell script
$ pip install pika
```

Зайдите в веб-интерфейс, найдите очередь под названием hello и сделайте скриншот.
После чего запустите второй скрипт consumer.py и сделайте скриншот результата выполнения скрипта

*В качестве решения домашнего задания приложите оба скриншота, сделанных на этапе выполнения.*

Для закрепления материала можете попробовать модифицировать скрипты, чтобы поменять название очереди и отправляемое сообщение.

Ответ:

Отображение очереди 'Hello' в веб интерфейсе после однократного запуска producer.py

![2-1_produce_hello.png](https://github.com/Zikin18/SYS-25_11.04/blob/master/2-1_produce_hello.png)

Вывод комманды consumer.py

![2-2_consumer.png](https://github.com/Zikin18/SYS-25_11.04/blob/master/2-2_consumer.png)


### Задание 3. Подготовка HA кластера

Используя Vagrant или VirtualBox, создайте вторую виртуальную машину и установите RabbitMQ.
Добавьте в файл hosts название и IP-адрес каждой машины, чтобы машины могли видеть друг друга по имени.

Пример содержимого hosts файла:
```shell script
$ cat /etc/hosts
192.168.0.10 rmq01
192.168.0.11 rmq02
```
После этого ваши машины могут пинговаться по имени.

Затем объедините две машины в кластер и создайте политику ha-all на все очереди.

*В качестве решения домашнего задания приложите скриншоты из веб-интерфейса с информацией о доступных нодах в кластере и включённой политикой.*

Также приложите вывод команды с двух нод:

```shell script
$ rabbitmqctl cluster_status
```

Для закрепления материала снова запустите скрипт producer.py и приложите скриншот выполнения команды на каждой из нод:

```shell script
$ rabbitmqadmin get queue='hello'
```

После чего попробуйте отключить одну из нод, желательно ту, к которой подключались из скрипта, затем поправьте параметры подключения в скрипте consumer.py на вторую ноду и запустите его.

*Приложите скриншот результата работы второго скрипта.*

Ответ:

Перед обьединением в клаcтер в файлы hosts обоих серверов были добавлены строки

```
192.168.56.104 vm1server
192.168.56.102 vm2server
```

В файлы '.erlang.cookie' было установлено одинаковое значение на обоих серверах.
После чего второй сервер был добавлен в кластер к первому.

```
sudo rabbitmqctl stop_app
sudo rabbitmqctl join_cluster rabbit@vm1server
sudo rabbitmqctl start_app
```
![3-1_cluster.png](https://github.com/Zikin18/SYS-25_11.04/blob/master/3-1_cluster.png)

На первом сервере была добавлена политика 
```
sudo rabbitmqctl set_policy ha-all ".*" '{"ha-mode":"all"}'  
```

![3-2_polucy.png](https://github.com/Zikin18/SYS-25_11.04/blob/master/3-2_polucy.png)

Результат выполнения 
```
sudo rabbitmqctl cluster_status
```

![3-3_vm1_status.png](https://github.com/Zikin18/SYS-25_11.04/blob/master/3-3_vm1_status.png)
![3-4_vm2_status.png](https://github.com/Zikin18/SYS-25_11.04/blob/master/3-4_vm2_status.png)

Вывод команды 'rabbitmqadmin get queue='hello'' после нескольких запусков producer.py

![3-5_vm1_queue.png](https://github.com/Zikin18/SYS-25_11.04/blob/master/3-5_vm1_queue.png)
![3-6_vm2_queue.png](https://github.com/Zikin18/SYS-25_11.04/blob/master/3-6_vm2_queue.png)

Для чтения очереди с другого сервера в скрип consumer.py пришлось внести изменения чтобы добавить учетные данные пользователя

```
#!/usr/bin/env python
# coding=utf-8
import pika
credentials = pika.PlainCredentials('beloborodov', '123123')
connection = pika.BlockingConnection(pika.ConnectionParameters('vm2server',5672,'/',credentials))
channel = connection.channel()
channel.queue_declare(queue='hello')

def callback(ch, method, properties, body):
    print(" [x] Received %r" % body)


channel.basic_consume('hello', callback, auto_ack=True)
channel.start_consuming()
```

Результат выполнения

![3-7_consume.png](https://github.com/Zikin18/SYS-25_11.04/blob/master/3-7_consume.png)
