---
title: es集群配置
date: 2019-02-13 15:04:44
tag:
---

服务器部署说明
------

### Linux
1. 安装wget
``` yum install wget -y```
2. 修改内核最大map数量并生效
``` echo  'vm.max_map_count = 262144' >> /etc/sysctl.conf && sysctl -p ``` 
3. 新增用户es
``` useradd es -d /opt/es ```
4. 设置文件打开数量
编辑 ``` vi /etc/security/limits.conf ```
增加 ```es               -       nofile          65536 ```


### elasticsearch 集群部署
1. 查看各个节点ip,确保能相互ping通:
    如:
    ```
    10.20.250.80
    10.20.250.81
    10.20.250.82
    ```

2. 各个节点切换用户,下载elasticsearch,解压
    ```
    su - es
    wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.3.1.tar.gz
    tar -zxv -f elasticsearch-6.3.1.tar.gz
    ln -s elasticsearch-6.3.1 elasticsearch
    ```
3. 编辑节点1配置文件 ```elasticsearch/config/elasticsearch.yml```
```
    cluster.name: erp-dfs
    node.name: ${HOSTNAME}
    #path.data: /esdata/data
    #path.logs: /esdata/logs
    network.host: 10.20.250.80
    http.port: 9200
    transport.tcp.port: 9300
    bootstrap.system_call_filter: false
    xpack.monitoring.collection.enabled: true
    discovery.zen.ping.unicast.hosts: ["10.20.250.81", "10.20.250.82"]
```
4. 编辑节点2配置文件 ```elasticsearch/config/elasticsearch.yml```
```
    cluster.name: erp-dfs
    node.name: ${HOSTNAME}
    #path.data: /esdata/data
    #path.logs: /esdata/logs
    network.host: 10.20.250.81
    http.port: 9200
    transport.tcp.port: 9300
    bootstrap.system_call_filter: false
    xpack.monitoring.collection.enabled: true
    discovery.zen.ping.unicast.hosts: ["10.20.250.80", "10.20.250.82"]
```

5. 编辑节点3配置文件 ```elasticsearch/config/elasticsearch.yml```
```
    cluster.name: erp-dfs
    node.name: ${HOSTNAME}
    #path.data: /esdata/data
    #path.logs: /esdata/logs
    network.host: 10.20.250.82
    http.port: 9200
    transport.tcp.port: 9300
    bootstrap.system_call_filter: false
    xpack.monitoring.collection.enabled: true
    discovery.zen.ping.unicast.hosts: ["10.20.250.80", "10.20.250.81"]
```

6. 启动各个节点
```
    bin/elasticsearch
```

7. 验证各个节点是否正常运行
查看启动信息中是否包含以下信息:

```
    [2018-07-31T12:39:18,667][INFO ][o.e.c.s.ClusterApplierService] [sysopt-node3.novalocal] added {{sysopt-node1.novalocal}{N8IhoG11Tfia8Y52klYrqg}{bqqmJB2iScaV1f2rD9qCyw}{10.20.250.80}{10.20.250.80:9300}{ml.machine_memory=8202297344, ml.max_open_jobs=20, xpack.installed=true, ml.enabled=true},}, reason: apply cluster state (from master [master {sysopt-node3.novalocal}{adTO7mKPT32UAq9aZxY_gg}{0C_DuqnPReSxEZ6i2hO5Hw}{10.20.250.82}{10.20.250.82:9300}{ml.machine_memory=8202297344, xpack.installed=true, ml.max_open_jobs=20, ml.enabled=true} committed version [15] source [zen-disco-node-join]])
    [2018-07-31T12:40:59,787][INFO ][o.e.c.s.ClusterApplierService] [sysopt-node3.novalocal] added {{sysopt-node2.novalocal}{-CHWSrLjQTOZqT3q75GVIA}{r93bo_kQTwOZxNZCekDsWA}{10.20.250.81}{10.20.250.81:9300}{ml.machine_memory=8202297344, ml.max_open_jobs=20, xpack.installed=true, ml.enabled=true},}, reason: apply cluster state (from master [master {sysopt-node3.novalocal}{adTO7mKPT32UAq9aZxY_gg}{0C_DuqnPReSxEZ6i2hO5Hw}{10.20.250.82}{10.20.250.82:9300}{ml.machine_memory=8202297344, xpack.installed=true, ml.max_open_jobs=20, ml.enabled=true} committed version [16] source [zen-disco-node-join]])
```

```
    netstat -tunlp |grep 9200
    netstat -tunlp |grep 9300
```

    结果:
```
    tcp6       0      0 10.20.250.82:9200       :::*                    LISTEN      10373/java
    tcp6       0      0 10.20.250.82:9300       :::*                    LISTEN      10373/java
```

    ```
    是否有输出
    curl http://10.20.250.82:9200 
    
    {
    "name" : "sysopt-node1.novalocal",
    "cluster_name" : "erp-dfs",
    "cluster_uuid" : "NclsYFudSHOgBxGHoap2yQ",
    "version" : {
        "number" : "6.3.1",
        "build_flavor" : "default",
        "build_type" : "tar",
        "build_hash" : "eb782d0",
        "build_date" : "2018-06-29T21:59:26.107521Z",
        "build_snapshot" : false,
        "lucene_version" : "7.3.1",
        "minimum_wire_compatibility_version" : "5.6.0",
        "minimum_index_compatibility_version" : "5.0.0"
    },
    "tagline" : "You Know, for Search"
    }
```

8. 启用守护进程

> 停止各个节点服务,使用下面的命令进行守护

    bin/elasticsearch -d -p pid
