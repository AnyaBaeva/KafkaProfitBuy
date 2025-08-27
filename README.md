# KafkaProfitBuy
в powershell
перейти в каталог файла docker-compose.yml 
выполнить команду docker-compose up -d
дождаться запуска сервисов
### результат: 
Добавлен Kafka UI с возможностями:
📊 Единый интерфейс для обоих кластеров
Порт: 8080

Доступ: http://localhost:8080

🔧 Подключенные сервисы:
Cluster1 (брокеры 1-3) + Schema Registry + Connect

Cluster2 (брокеры 4-6)

🌟 Возможности Kafka UI:
📝 Управление топиками - создание, удаление, настройка

👀 Просмотр сообщений в реальном времени

📈 Мониторинг производительности брокеров

🔍 Анализ consumer groups и lag

⚙️ Настройка репликации и partitions

📋 Schema management для Avro данных

🔄 Connect management - мониторинг коннекторов

📊 Визуализация метрик и здоровья кластера

🎯 Преимущества:
Единая точка управления двумя кластерами

Web-интерфейс вместо командной строки

Real-time мониторинг

Удобное управление топиками и потребителями

Визуализация данных и метрик

# MirrorSourceConnector (репликация топиков):
PUT запрос
http://localhost:8083/connectors/mirror-source-connector/config
"Content-Type: application/json"

```
{
"name": "mirror-source-connector",
"connector.class": "org.apache.kafka.connect.mirror.MirrorSourceConnector",
"source.cluster.alias": "cluster1",
"target.cluster.alias": "cluster2",
"source.cluster.bootstrap.servers": "broker1:29092,broker2:29093,broker3:29094",
"target.cluster.bootstrap.servers": "broker4:29095,broker5:29096,broker6:29097",
"topics": "replicate_me",
"replication.factor": 3,
"sync.topic.acls.enabled": "true"
}

```

Результат:
![img.png](img.png)

# MirrorCheckpointConnector (репликация оффсетов):
PUT запрос
http://localhost:8083/connectors/mirror-checkpoint-connector/config
"Content-Type: application/json"

```
{
"name": "mirror-checkpoint-connector",
"connector.class": "org.apache.kafka.connect.mirror.MirrorCheckpointConnector",
"source.cluster.alias": "cluster1",
"target.cluster.alias": "cluster2",
"source.cluster.bootstrap.servers": "broker1:29092,broker2:29093,broker3:29094",
"target.cluster.bootstrap.servers": "broker4:29095,broker5:29096,broker6:29097",
"groups": ".*",
"groups.exclude": "console-consumer-.*,connect-.*,__.*",
"sync.group.offsets.enabled": "true",
"emit.checkpoints.interval.seconds": 60,
"refresh.groups.interval.seconds": 60
}

```

# MirrorHeartbeatConnector (мониторинг репликации):
PUT запрос
http://localhost:8083/connectors/mirror-heartbeat-connector/config
"Content-Type: application/json"

```
{
"name": "mirror-heartbeat-connector",
"connector.class": "org.apache.kafka.connect.mirror.MirrorHeartbeatConnector",
"source.cluster.alias": "cluster1",
"target.cluster.alias": "cluster2",
"source.cluster.bootstrap.servers": "broker1:29092,broker2:29093,broker3:29094",
"target.cluster.bootstrap.servers": "broker4:29095,broker5:29096,broker6:29097",
"emit.heartbeats.interval.seconds": 10
}


```

# Schema Registry
загрузить схему через kafka ui
схема в файле ./product.avsc
создать схему:

docker run --rm -it --network=host `
confluentinc/cp-schema-registry:7.4.0 `
kafka-avro-console-producer `
--broker-list localhost:29092,localhost:29093,localhost:29094 `
--topic products `
--property schema.registry.url=http://localhost:8081 `
--property value.schema='{\"type\":\"record\",\"name\":\"User\",\"namespace\":\"com.example.avro\",\"fields\":[{\"name\":\"id\",\"type\":\"int\"},{\"name\":\"name\",\"type\":\"string\"},{\"name\":\"email\",\"type\":\"string\"},{\"name\":\"age\",\"type\":\"int\"}]}'

