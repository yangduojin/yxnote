# copy

```linux

docker run -d --name kafka \
-p 9092:9092 \
-e KAFKA_BROKER_ID=0 \
-e KAFKA_ZOOKEEPER_CONNECT=192.168.193.128:2181 \
-e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://192.168.193.128:9092 \
-e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 wurstmeister/kafka























```
