## Описание задачи

В данной работе пока рассмотривается вариант межкластерного взаимодействия Active-Passive с помощью брокера сообщений Kafka. При реализации Active-Active необходимо прописать политику решения конфликтов в конфигурационный файл.

### Настройка окружения
1. Перейти в корень проекта первого кластера Active

> cd C:\Users\Ishkhan\IdeaProjects\ignite-cdc-a

2. Поднять кластер Ignite a
 
> docker-compose up -d

3. Перейти в корень проекта второго кластера Passive

> cd C:\Users\Ishkhan\IdeaProjects\ignite-cdc-b

3.1. Активировать кластер
> docker-compose exec -it ignite-node-a-1 ./apache-ignite/bin/control.sh --set-state ACTIVE

3.2. Создать таблицу CDC_TEST1

> CREATE TABLE CDC_TEST1 (
id INT,
name VARCHAR(100),
value DECIMAL(10, 2),
status VARCHAR(20),
created_date TIMESTAMP,
updated_date TIMESTAMP,
PRIMARY KEY (id)
) WITH "CACHE_NAME=CDC_TEST1, TEMPLATE=PARTITIONED, VALUE_TYPE=SQL_CDC_TEST1_TYPE";

4. Поднять кластер Ignite b

> docker-compose up -d

4.1. Активировать кластер
> docker-compose exec -it ignite-node-b-1 ./apache-ignite/bin/control.sh --set-state ACTIVE

4.2. Создать таблицу CDC_TEST1

> CREATE TABLE CDC_TEST1 (
id INT,
name VARCHAR(100),
value DECIMAL(10, 2),
status VARCHAR(20),
created_date TIMESTAMP,
updated_date TIMESTAMP,
PRIMARY KEY (id)
) WITH "CACHE_NAME=CDC_TEST1, TEMPLATE=PARTITIONED";

5. Перейти в директорию kafka 

> C:\Users\Ishkhan\IdeaProjects\ignite-cdc-a\kafka

6. Поднять брокер сообщения Kafka 

> docker-compose up -d

7. Запустить ./ignite-cdc.sh на каждой ноде Active кластера

> docker-compose exec -it ignite-node-a-1 ./apache-ignite/bin/ignite-cdc.sh config/cdc-config.xml

> docker-compose exec -it ignite-node-a-2 ./apache-ignite/bin/ignite-cdc.sh config/cdc-config.xml

> docker-compose exec -it ignite-node-a-3 ./apache-ignite/bin/ignite-cdc.sh config/cdc-config.xml

8. Запустить один раз ./kafka-to-ignite.sh на любой ноде Passive кластера

>  docker-compose exec -it ignite-node-b-1 ./apache-ignite/bin/kafka-to-ignite.sh -v /opt/ignite/apache-ignite/config/kafka-config.xml

9. Вставляем тестовую запись в Ignite a

> INSERT INTO CDC_TEST1 (id, name, value, status, created_date, updated_date)
VALUES (1, 'Товар А', 1500.50, 'ACTIVE', CURRENT_TIMESTAMP, CURRENT_TIMESTAMP);