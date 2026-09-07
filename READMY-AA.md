## Описание задачи

В данной работе рассмотривается вариант межкластерного взаимодействия Active-Active с помощью брокера сообщений Kafka.

Ключевой ньюанс данного взаимодействия - Разрешение конфликтов. Apache Ignite из коробки дает 2 стратегии разрешения конфликтов, ниже описана краткая выжимка с различиями двух подходов и алгоритмы из документации. Также есть возможность разработать свой механизм для разрешения конфликтов.
Для включения разрешения конфликта необходимо добавить в конфигурацию бин CacheVersionConflictResolverPluginProvider. Если задавать параметр conflictResolveField для добавленного бина, то применится стратегия "Разрешение конфликта на основе поля value(conflictResolveField) записи", в противном случае стратегия "Разрешение конфликтов на основе версии записи". В нашем примере мы используем стратегию "Разрешение конфликтов на основе версии записи". При реализации собственной стратегии разрешения конфликтов добавляют в бин параметр conflictResolver, в нашем случае он не нужен.

#### Различие стратегий "Разрешение конфликтов на основе версии записи" и "Разрешение конфликта на основе поля value(conflictResolveField) записи"

У каждой записи в БД есть версия основанная на временной метке. Первая стратегия разрешает конфликты на основе самой свежей версии данной метки и реализует алгоритм LWW. Второй алгоритм тоже смотрит на данную метку, но если по ним он не смог разрешить конфликт, он еще смотрит на поле conflictResolveField. Поле conflictResolveField представляет собой, к примеру, какое-то монотонное значение, которое увеличивается при модификации. У какой записи это conflictResolveField больше, тот и побеждает в решении конфликта.  

* Conflict resolution based on the entry’s version (Разрешение конфликтов на основе версии записи)

This approach provides the eventual consistency guarantee when each entry is updatable only from a single cluster.

This approach does not replicate any updates or removals from the destination cluster to the source cluster.

Algorithm:

a. Changes from the "local" cluster are always win. Any replicated data can be overridden locally.

b. If both old and new entry are from the same cluster then entry versions comparison is used to determine the order.

c. Conflict resolution failed. Update will be ignored. Failure will be logged.

* Conflict resolution based on the entry’s value field (Разрешение конфликта на основе поля value(conflictResolveField) записи)

This approach provides the eventual consistency guarantee even when entry is updatable from any cluster.

Conflict resolution field, specified by conflictResolveField, should contain a user provided monotonically increasing value such as query id or timestamp.

This approach does not replicate the removals from the destination cluster to the source cluster, because removes can’t be versioned by the field.

Algorithm:

a. Changes from the "local" cluster are always win. Any replicated data can be overridden locally.

b. If both old and new entry are from the same cluster then entry versions comparison is used to determine the order.

c. If conflictResolveField is provided then field values comparison is used to determine the order.

d. Conflict resolution failed. Update will be ignored. Failure will be logged.

### Настройка окружения

1. Перейти в директорию kafka

> cd C:\Users\Ishkhan\IdeaProjects\ignite-cdc-a\kafka

2. Поднять брокер сообщения Kafka

> docker-compose up -d

3. Перейти в корень проекта Ignite a кластера

> cd C:\Users\Ishkhan\IdeaProjects\ignite-cdc-a

4. Установить ignite-cdc-ext.***.jar в папку libs Ignite a кластера. В папке shelve хранятся различные версии расширения. Для версий ignite 2.15 и ниже - ignite-cdc-ext-1.2.1-8.jar (скомпилирован и адаптированный на 8 java). Для версии выше ignite 2.15 - ignite-cdc-ext-1.0.0.jar (скомпилирован на 11 java).


5. Поднять Ignite a кластер

> docker-compose up -d

6. Необходимо активировать Ignite a кластер после запуска всех нод.

> docker-compose exec -it ignite-node-a-1 ./apache-ignite/bin/control.sh --set-state ACTIVE

7. Создать таблицу CDC_TEST1 в кластере Ignite a. Можно подключиться к кластеру через DBeaver. Важный ньюанс, VALUE_TYPE очень важно явно указать, иначе Ignite сгенерирует случайное значение и при передаче данных реплицирующая и реплицируемые таблицы не смапятся.

```
CREATE TABLE CDC_TEST1 (
id INT,
name VARCHAR(100),
value DECIMAL(10, 2),
status VARCHAR(20),
created_date TIMESTAMP,
updated_date TIMESTAMP,
PRIMARY KEY (id)
) WITH "CACHE_NAME=CDC_TEST1, TEMPLATE=PARTITIONED, VALUE_TYPE=SQL_CDC_TEST1_TYPE";
```
8. Запустить ./ignite-cdc.sh на каждой ноде Ignite a кластера.

> docker-compose exec -it ignite-node-a-1 ./apache-ignite/bin/ignite-cdc.sh config/cdc-config.xml

> docker-compose exec -it ignite-node-a-2 ./apache-ignite/bin/ignite-cdc.sh config/cdc-config.xml

> docker-compose exec -it ignite-node-a-3 ./apache-ignite/bin/ignite-cdc.sh config/cdc-config.xml

9. Запустить один раз ./kafka-to-ignite.sh на любой ноде Ignite a кластера

> docker-compose exec -it ignite-node-a-1 ./apache-ignite/bin/kafka-to-ignite.sh -v /opt/ignite/apache-ignite/config/kafka-config.xml

10. Перейти в корень проекта Ignite b кластера

> cd C:\Users\Ishkhan\IdeaProjects\ignite-cdc-b

11. Установить ignite-cdc-ext.***.jar в папку libs Ignite b кластера. В папке shelve хранятся различные версии расширения. Для версий ignite 2.15 и ниже - ignite-cdc-ext-1.2.1-8.jar (скомпилирован и адаптированный на 8 java). Для версии выше ignite 2.15 - ignite-cdc-ext-1.0.0.jar (скомпилирован на 11 java).


12. Поднять Ignite b кластер

> docker-compose up -d

13. Необходимо активировать Ignite b кластер после запуска всех нод.
> docker-compose exec -it ignite-node-b-1 ./apache-ignite/bin/control.sh --set-state ACTIVE

14. Создать таблицу CDC_TEST1 в кластере Ignite b. Важный ньюанс, VALUE_TYPE очень важно явно указать, иначе Ignite сгенерирует случайное значение и при передаче данных реплицирующая и реплицируемые таблицы не смапятся.

```
CREATE TABLE CDC_TEST1 (
id INT,
name VARCHAR(100),
value DECIMAL(10, 2),
status VARCHAR(20),
created_date TIMESTAMP,
updated_date TIMESTAMP,
PRIMARY KEY (id)
) WITH "CACHE_NAME=CDC_TEST1, TEMPLATE=PARTITIONED, VALUE_TYPE=SQL_CDC_TEST1_TYPE";
```
15. Запустить ./ignite-cdc.sh на каждой ноде Ignite b кластера.

> docker-compose exec -it ignite-node-b-1 ./apache-ignite/bin/ignite-cdc.sh config/cdc-config.xml

> docker-compose exec -it ignite-node-b-2 ./apache-ignite/bin/ignite-cdc.sh config/cdc-config.xml

> docker-compose exec -it ignite-node-b-3 ./apache-ignite/bin/ignite-cdc.sh config/cdc-config.xml

16. Запустить один раз ./kafka-to-ignite.sh на любой ноде Ignite b кластера

>  docker-compose exec -it ignite-node-b-1 ./apache-ignite/bin/kafka-to-ignite.sh -v /opt/ignite/apache-ignite/config/kafka-config.xml

17. Вставить тестовую запись в Ignite b кластер. Можно подключиться к кластеру через DBeaver.

> INSERT INTO CDC_TEST1 (id, name, value, status, created_date, updated_date)
VALUES (1, 'Товар А', 1500.50, 'ACTIVE', CURRENT_TIMESTAMP, CURRENT_TIMESTAMP);

