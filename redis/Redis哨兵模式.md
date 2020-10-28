# Redis哨兵模式

## 应用场景：

当主服务器宕机后需要手动把一台从服务器切换为主服务器，这就需要人工干预费时费力还会造成一段时间内服务不可用，更多时候我们优先考虑哨兵模式(Sentinel)

## 模式概述

Sentinel是一种特殊的模式，首先Redis提供了哨兵的命令，哨兵是一个独立的进程，作为进程他会独立运行，原理为哨兵通过发送命令等待Redis服务器响应从而监视运行多个Redis实例

![image-20200603080040049](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200603080040049.png)

在这里哨兵有两个作用

```
通过发送命令让Redis服务器返回监控其 运行状态，包括主服务器和从服务器
当哨兵监视到主服务器宕机会自动将主服务器 切换成从服务器然后通过发布订阅模式通知其他的从服务器修改配置文件让他们切换主机
```

然而一个哨兵进程对Redis服务器进行监控可能会出现问题为此我们可以使用多个哨兵进行监控各个哨兵之间还会进行监控这样就形成了多哨兵模式

## 故障切换(failover)

假设主服务器宕机哨兵1先检测到这个结果系统并不会马上进行failover过程仅仅是哨兵1主观的认为主服务器不可用这个现象称为主观下线。当后面的哨兵也检测到主服务器不可用并且数量达到一定值时，哨兵之间就会进行一次投票投票结果由一个哨兵发起进行failover操作，切换成功后就会通过发布订阅模式，让各个哨兵把自己监控搞得从服务器实现切换主机这个过程称为客观下线，这样对于客户端而言一切都是透明的 

## 修改服务器redis配置设置主从服务器以及哨兵

修改redis.conf文件

```
# 使得Redis服务器可以跨网络访问
bind 0.0.0.0
# 设置密码(可以不设置)
requirepass "123456"
# 指定主服务器，注意：有关slaveof的配置只是配置从服务器，主服务器不需要配置
slaveof 192.168.11.128 6379
# 主服务器密码，注意：有关slaveof的配置只是配置从服务器，主服务器不需要配置
masterauth 123456
```

上述内容主要是配置Redis服务器，从服务器比主服务器多一个slaveof的配置和密码。





放置哨兵修改redis安装目录下的sentinel.conf文件

```
# 禁止保护模式
protected-mode no
# 配置监听的主服务器，这里sentinel monitor代表监控，mymaster代表服务器的名称，可以自定义，192.168.11.128代表监控的主服务器，6379代表端口，2代表只有两个或两个以上的哨兵认为主服务器不可用的时候，才会进行failover操作。
sentinel monitor mymaster 192.168.11.128 6379 2
# sentinel author-pass定义服务的密码，mymaster是服务名称，123456是Redis服务器密码
# sentinel auth-pass <master-name> <password>
sentinel auth-pass mymaster 123456
```



进入Redis的安装目录的src目录通过下方的命令来启动哨兵

```
# 启动Redis服务器进程
./redis-server ../redis.conf
# 启动哨兵进程
./redis-sentinel ../sentinel.conf
```



## Java中使用哨兵模式（Jedis连接池）

```
   JedisPoolConfig jedisPoolConfig=new JedisPoolConfig();
      jedisPoolConfig.setMaxTotal(10);//设置最大链接数
      jedisPoolConfig.setMaxIdle(10);//设置最大空闲数
      jedisPoolConfig.setMaxWaitMillis(10);//设置最大等待时间/s
      jedisPoolConfig.setJmxEnabled(true);
      jedisPoolConfig.setTestWhileIdle(true);//开启在空闲时检查redis
      Set<String>set=new HashSet<>();
      set.add("192.168.138.128:26380");
      set.add("192.168.138.128:26381");
      set.add("192.168.138.128:26382");
      JedisSentinelPool jedisSentinelPool=new JedisSentinelPool("mymaster",set,jedisPoolConfig);//创建哨兵模式的Jedis连接池
      Jedis jedis1=jedisSentinelPool.getResource();
      jedis1.close();
      jedisSentinelPool.close();
```

需要开启端口并且需要改变服务器的redis.conf里的protected-model变为no即关闭保护模式，没有投票决定之前set里面的第一个即为主集群