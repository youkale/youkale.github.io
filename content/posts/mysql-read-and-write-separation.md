---
title: 通过Connector/J实现数据库读写分离
date: 2019-03-01 10:40:00
tag: database
---

## 前言

>在公司的一些业务中，不管什么分布式应用还是单机应用，不可否认CURD是日常工作中最常接触的，那么数据库的性能问题，可能就关系到你整个应用的性能。随着公司的成长，数据两的增大，很多公司为了保证数据的安全、可用，一般都会通过binlog加入slave服务器对master进行一个备份, 但是在实际应用中，很多人都把slave仅仅当作一个备份，最多的就是dba同学可能在slave上面查点数据, 而我们的应用还只是对主库的一个读写。那么我们怎么样才能把slave利用起来呢？ 很多同学可能回想到在应用代码中配置两个数据源,针对select语句走从库，修改操作走主库。用过sharding-jdbc的也许就会想到用他的master-slave模式，这些都是可以实现读写分离。

## Connector/J

>前面卖了个关子，那么下面我就来补充一个使用原生Mysql驱动如何简单实现读写分离:

    jdbc:mysql:replication://[master host][:port],[slave host 1][:port][,[slave host 2][:port]]...[/[database]][?propertyName1=propertyValue1[&propertyName2=propertyValue2]...]

是不是看了这个URI立刻就明白了？ mysql驱动可以直接通过配置replication来实现读写分离.
当用多个slave,会自动在slaves中根据你的策略进行LB,内置的策略有random(随机),bestResponseTime(最快响应),sequential(顺序),如果这些都不能满足你的需求,你还可以自己实现BalanceStrategy接口自定义一个。不用改一行业务代码，通过修改一个jdbc连接字符串就能实现读写分离，还有什么比这个更爽的吗？
