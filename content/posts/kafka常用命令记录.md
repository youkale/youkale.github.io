---
title: kafka常用命令记录
date: 2019-02-13 14:28:42
tag:
---

# kafka常用命令记录

#### 消费测试
    kafka-console-consumer.bat --bootstrap-server 172.20.2.95:9092 --topic xxx --from-beginning

#### 查看消费者
    kafka-consumer-groups.sh --bootstrap-server 172.20.2.95:9092 --list
    kafka-consumer-groups.sh --zookeeper 172.20.2.95:2181 --list

####  查看新加入的消费者
    kafka-consumer-groups.sh --new-consumer --bootstrap-server 172.20.2.95:9092 --list

#### 修改分区
    kafka-topics.sh --zookeeper 10.20.247.104:2181,10.20.247.105:2181,10.20.247.106:2181 --alter --topic yugong-dfs_db-ta_documents  --partitions 20

#### 查看分区
    kafka-topics.sh --zookeeper 10.20.247.104:2181,10.20.247.105:2181,10.20.247.106:2181 --topic yugong-dfs --describe

#### 生产者压测
    kafka-producer-perf-test.sh --topic test_produce --num-records 100000000 --record-size 1000 --throughput 200000 --producer-props bootstrap.servers=10.20.247.104:9092,10.20.247.105:9092,10.20.247.106:9092

#### 消费者压测
    kafka-consumer-perf-test.sh --zookeeper localhost:2181 --topic test_perf --fetch-size 1048576 --messages 1000000 --threads 1

####  增加副本
    kafka-reassign-partitions.sh --zookeeper 10.20.247.104:2181,10.20.247.105:2181,10.20.247.106:2181 --reassignment-json-file replicas.json --execute

####  验证副本
    kafka-reassign-partitions.sh --zookeeper 10.20.247.104:2181,10.20.247.105:2181,10.20.247.106:2181 --reassignment-json-file replicas.json --verify

#### 查看消费offset
    kafka-consumer-groups.sh --bootstrap-server 10.20.247.104:9092,10.20.247.105:9092,10.20.247.106:9092 --describe --group erp-dfs-group-id-prod-1
