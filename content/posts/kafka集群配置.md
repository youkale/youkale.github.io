---
title: kafka集群配置
date: 2019-02-13 15:01:13
tag:
---

Kafka集群配置
---

## kafka集群需要使用到zookeeper,需要在配置kafka之前配置zookeeper

本集群里使用的主机ip

```
    节点1: 10.20.250.80
    节点2: 10.20.250.81
    节点3: 10.20.250.83
```  


相关路径

```
    主程序:        /data/kafka
    kafka配置:     /data/kafka/server.properties
    kafka数据:     /data/kafka/kafkadata
    zookeeper配置: /data/kafka/zookeeper.properties
    zookeeper数据: /data/kafka/zkdata
```


首先下载kafka,部署中使用的版本为 ``` kafka_2.11-0.10.2.1.tgz ```

```
    cd /data/
    tar -zxv -f  kafka_2.11-0.10.2.1.tgz
    ln -s kafka_2.11-0.10.2.1.tgz kafka
```


zookeeper 集群配置
---

每个节点对应开放的端口有 ``` 2181 ```` ``` 2888 ````  ``` 3888 ````

1. 首先确认zookeeper的数据日志目录,比如本例中使用的 ```/data/kafka/zkdata```
    
```
        #在3个节点上分别执行,创建目录
        mkdir -p /data/kafka/zkdata
```
    
2. 生成myid

```
        节点1~$ echo 1 > /data/kafka/zkdata/myid  #10.20.250.80
        节点2~$ echo 2 > /data/kafka/zkdata/myid  #10.20.250.81
        节点3~$ echo 3 > /data/kafka/zkdata/myid  #10.20.250.83
```

3. 生成配置文件 ```/data/kafka/zookeeper.properties```

```
        tickTime=2000
        initLimit=5
        syncLimit=2
        dataDir=/data/kafka/zkdata
        dataLogDir=/data/kafka/zkdata
        clientPort=2181
        server.1=10.20.250.80:2888:3888
        server.2=10.20.250.81:2888:3888
        server.3=10.20.250.83:2888:3888
```
4. 启动服务

```
        分别启动各个服务器
        bin/zookeeper-server-start.sh -daemon zookeeper.properties
        测试各个节点是否端口开放
        测试是否有错误日志
        
```

配置kafka,监听端口为 ```9092```

1. 节点1配置 ```/data/kafka/server.properties```

```
        broker.id=0 
        advertised.listeners=PLAINTEXT://10.20.250.80:9092   #节点ip
        num.network.threads=3
        num.io.threads=8
        socket.send.buffer.bytes=102400
        socket.receive.buffer.bytes=102400
        socket.request.max.bytes=104857600
        log.dirs=/data/kafka/kafkadata    #kafka数据目录
        num.partitions=1
        num.recovery.threads.per.data.dir=1
        offsets.topic.replication.factor=1
        transaction.state.log.replication.factor=1
        transaction.state.log.min.isr=1
        log.retention.hours=720
        log.segment.bytes=1073741824
        log.retention.check.interval.ms=300000
        zookeeper.connect=10.20.250.80:2181,10.20.250.81:2181,10.20.250.83:2181  #配置的zookeeper节点ip+主端口号,以逗号分隔
        zookeeper.connection.timeout.ms=6000
        group.initial.rebalance.delay.ms=0
```
    
2. 节点2配置 ```/data/kafka/server.properties```

```
        broker.id=1 
        advertised.listeners=PLAINTEXT://10.20.250.81:9092   #节点ip
        num.network.threads=3
        num.io.threads=8
        socket.send.buffer.bytes=102400
        socket.receive.buffer.bytes=102400
        socket.request.max.bytes=104857600
        log.dirs=/data/kafka/kafkadata    #kafka数据目录
        num.partitions=1
        num.recovery.threads.per.data.dir=1
        offsets.topic.replication.factor=1
        transaction.state.log.replication.factor=1
        transaction.state.log.min.isr=1
        log.retention.hours=720
        log.segment.bytes=1073741824
        log.retention.check.interval.ms=300000
        zookeeper.connect=10.20.250.80:2181,10.20.250.81:2181,10.20.250.83:2181  #配置的zookeeper节点ip+主端口号,以逗号分隔
        zookeeper.connection.timeout.ms=6000
        group.initial.rebalance.delay.ms=0
```

3. 节点3配置 ```/data/kafka/server.properties```

```
        broker.id=2 
        advertised.listeners=PLAINTEXT://10.20.250.83:9092   #节点ip
        num.network.threads=3
        num.io.threads=8
        socket.send.buffer.bytes=102400
        socket.receive.buffer.bytes=102400
        socket.request.max.bytes=104857600
        log.dirs=/data/kafka/kafkadata    #kafka数据目录
        num.partitions=1
        num.recovery.threads.per.data.dir=1
        offsets.topic.replication.factor=1
        transaction.state.log.replication.factor=1
        transaction.state.log.min.isr=1
        log.retention.hours=720
        log.segment.bytes=1073741824
        log.retention.check.interval.ms=300000
        zookeeper.connect=10.20.250.80:2181,10.20.250.81:2181,10.20.250.83:2181  #配置的zookeeper节点ip+主端口号,以逗号分隔
        zookeeper.connection.timeout.ms=6000
        group.initial.rebalance.delay.ms=0
```
    
4. 启动服务器,分别在各个节点启动服务
```
        bin/kafka-server-start.sh -daemon server.properties
```
    
5. 测试

```
    #测试端口是否正常开放,测试是否有异常输出
    #创建主题
    bin/kafka-topics.sh --create --zookeeper 10.20.250.80:2181,10.20.250.81:2181,10.20.250.83:2181  --replication-factor 1 --partitions 1 --topic test
    #查看主题
    bin/kafka-topics.sh --list --zookeeper 10.20.250.80:2181,10.20.250.81:2181,10.20.250.83:2181
```
