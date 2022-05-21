---
title: 重谈redis分布式锁
date: 2018-08-12 16:58:39
tag: java
---


### 前言

  Redis分布式锁其实网上已经一大把了，我写这篇博客的目的是想发表下自己的浅见。

  我们知道在单进程中，通过语言内置的锁可以保证数据的一致性，随着软件的不断扩大，很多应用已经是多进程的方式部署,那么为了保证数据的一致性,我们需要一个分布式锁。标题说的是Redis分布式锁，那么我下面主要讲的rendis锁，而且不会涉及到redis单点故障的问题。


### 常见的分布锁

  搜索一下常见的分布式锁说的都是通过redis和zookeeper来实现。

   - redis
     
     > setnx 通过设置一个共有的key，而且通过设置过期时间来避免死锁
  
   - zk

     > 通过设置一个共有的节点,通过zk的watch,观察节点的变化进行通知
     

### 锁的需求

  - 独占: 一个时间点只能一个线程访问

  - 可重入: 同一个线程只要当前持有锁，可以重复进入

  - 避免死锁: 多个线程访问不会出现死锁的情况

  - 锁释放: 进程故障能自动释放锁，只有持有锁的进程能释放锁


### 尝试自己实现一个

  > 实现 copy from importnew [Redis 分布式锁的正确实现方式（ Java 版 ）](http://www.importnew.com/27477.html)

```bash
public class RedisTool {
 
    private static final String LOCK_SUCCESS = "OK";
    private static final String SET_IF_NOT_EXIST = "NX";
    private static final String SET_WITH_EXPIRE_TIME = "PX";
 
    public static boolean tryGetDistributedLock(Jedis jedis, String lockKey, String requestId, int expireTime) {
 
        String result = jedis.set(lockKey, requestId, SET_IF_NOT_EXIST, SET_WITH_EXPIRE_TIME, expireTime);
 
        if (LOCK_SUCCESS.equals(result)) {
            return true;
        }
        return false;
 
    }

    private static final Long RELEASE_SUCCESS = 1L;
 
    public static boolean releaseDistributedLock(Jedis jedis, String lockKey, String requestId) {
 
        String script = "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end";
        Object result = jedis.eval(script, Collections.singletonList(lockKey), Collections.singletonList(requestId));
 
        if (RELEASE_SUCCESS.equals(result)) {
            return true;
        }
        return false;
 
    }
 
}
```

---

 > 上面的锁实现存在一个问题,假如程序员A不小心调用了releaseDistributedLock就会把某个已经持有锁的线程的锁释放掉。相信大家都能想到用ThreadLocal方式避免给未持有锁的线程释放了。 那我就试着将这代码改改。(伪代码)

 ```bash
 public class RedisToolv2 {
 
    private static final String LOCK_SUCCESS = "OK";
    private static final String SET_IF_NOT_EXIST = "NX";
    private static final String SET_WITH_EXPIRE_TIME = "PX";
    private final ThreadLocal<Boolean> hasLock = new ThreadLocal<>();

    public static boolean tryGetDistributedLock(Jedis jedis, String lockKey, String requestId, int expireTime) {
 
        String result = jedis.set(lockKey, requestId, SET_IF_NOT_EXIST, SET_WITH_EXPIRE_TIME, expireTime);
        
        boolean locked = false;

        if (LOCK_SUCCESS.equals(result)) {
            locked = true;
        }
        hasLock.set(locked);
        return locked;
 
    }

    private static final Long RELEASE_SUCCESS = 1L;
 
    public static boolean releaseDistributedLock(Jedis jedis, String lockKey, String requestId) {
        boolean locked = Optional.ofNull(hasLock.get()).orElse(false) ;
        boolean unloked = false;
        if(locked){
            String script = "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end";
            Object result = jedis.eval(script, Collections.singletonList(lockKey), Collections.singletonList(requestId));
            if (RELEASE_SUCCESS.equals(result)) {
                unloked = true;
            }
        }
        return false;
    }
 
}
 ```

---

 上面的方式虽然说解决了其他线程误删锁的情况，但是分布式锁其实还有很多细节都地方需要实现，当一个线程已经拥有此锁了，如果还有大量其他(本应用或其他应用)的线程在请求锁，无疑给redis造成很大的压力(当然这个得看应用的规模，小规模是不会出现这个情况)。

 > 假设我们同一个应用中同一个时间点只有线程在请求分布式锁，那么相对于Redis的压力是不是就会下降很多? 那么我们在代码中加入一个同步锁是不是就解决了问题呢？首先我想到的是synchronized可以重入，可以使用递归的方式调用。但是如果是递归，如果长时间拿不到锁，那么有可能会引发StackOverflowError。额， 那这不能实现，没招了咩？ 其实jdk里面带了一个 ReentrantLock,这个锁其实跟synchronized差不多，也是可以重入的，而且有一个 tryLock(long timeout, TimeUnit unit)的方法，在递归调用的时候可以避免Stack过大。 （思路有了，我相信代码也是很轻易就写出来了）

### 抛个砖  
  - 那么这个锁持有的时间该如何计算呢？ 

  - 还有就是这么实现这个锁是不是就完美了呢？？  

  -  Redis有个叫[键空间通知(keyspace notification)](http://redisdoc.com/topic/notification.html) 的功能，是不是可以用这种方式来实现一个类似zookeeper的锁呢 ？ 


### 最后

各位看官还有什么好的思路欢迎来交流呀。