# Войти в контейнер Kafka
>docker exec -it kafka bash

> cd /opt/kafka/bin

# Создаем основной топик для данных (несколько партиций для параллельной обработки)
>./kafka-topics.sh --create \
--topic ignite_cdc_data \
--bootstrap-server localhost:9092 \
--partitions 3 \
--replication-factor 1 \
--config retention.ms=604800000 \
--config segment.bytes=1073741824

# Создаем мета-топик (строго 1 партиция!)
>./kafka-topics.sh --create \
--topic ignite_cdc_meta \
--bootstrap-server localhost:9092 \
--partitions 1 \
--replication-factor 1 \
--config retention.ms=2592000000 \
--config cleanup.policy=compact

# Проверить созданные топики
>./kafka-topics.sh --list --bootstrap-server localhost:9092

или

http://localhost:8088/

# Получить детальную информацию о топиках
>./kafka-topics.sh --describe --topic ignite_cdc_data --bootstrap-server localhost:9092

>./kafka-topics.sh --describe --topic ignite_cdc_meta --bootstrap-server localhost:9092