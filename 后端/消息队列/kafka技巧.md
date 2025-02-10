
# 查看Kafka是否可用
1. 查看所有topic
```
kafka-topics.sh --bootstrap-server localhost:9092 --list
```
2. 使用命令创建生产者
```
kafka-console-producer.sh --broker-list localhost:9092 --topic my_topic
```
3. 使用命令创建消费者，监控生产者
```
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic my_topic --from-beginning
```