# Redis缓存



## 企业缓存产品

### Memcached

优点：高性能读写，单一数据类型支持客户端式分布式集群，一致性hash

多核结构多线程读写性能高，缺点无持久化节点故障可能出现缓存穿透分布式需要客户端实现、跨机房数据同步困难架构扩容复杂度



### Redis

优点：高性能读写，多数据类型支持、数据持久化、高可用架构、支持自定义虚拟内存、支持分布式分片集群、单线程读写性能极高

缺点：多线程读写较Memcached慢



![image-20200530180901155](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200530180901155.png)

```
String是redis最基本的类型一个key对应一个value
String类型是二进制安全的意思是redis的string可以包含任何数据，比如jpd图片或者序列化对象
String类型是redis最基本的数据类哦行一个键最大能存储512MB


List是简单的字符串列表排序为插入的顺序列表最大长度为2^32-1
redis中的list是使用链表实现的这意味着及时列表中有上百万个元素增加一个元素在首部或者尾部都能在常量时间完成
可以使用likt获取最新的内容，还可以实现生产者消费者模式，生产者调用push添加到list中消费者调用pop从列表中取出

set是无序的字符串集合，集合中的值是唯一的无序的，可以对集合做出很多操作例如判断元素是否存在对多个及格执行交集并集和差集等等
我们通常可以用及格存储一些无关顺序的表达对象间关系的数据例如角色，这样就可以很轻松的判断某个用户是否拥有某个角色
set在一些用到随机值的场合是非常合适的我们可以用srandmember/spop去获取/弹出一个随机元素


Zset是有序的不重复的字符串集合，有序集合中的每个原申诉都关联了一个浮动值即为key的hash值，内部按照key的hash值排序好，并非请求时才排序而是结构已经设计成排序好了的

redis的hash值是字符串字段和字符串之间的映射是表示对象的完美数据类型，hash中的字段数量没有限制所以可以在你的应用程序以不同的方式来使用hash
```



## SpringBoot整合Redis缓存

### 配置

```
yml里面配置数据库配置和redis的host以及port
```

### 配置序列化工厂，使之在数据库中不为乱码

```
package com.springboot.redis.config;

import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.cache.CacheManager;
import org.springframework.cache.annotation.CachingConfigurerSupport;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheConfiguration;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.cache.RedisCacheWriter;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.*;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import java.time.Duration;

@Configuration
@EnableCaching
public class RedisConfig extends CachingConfigurerSupport {

    /**
     * 选择redis作为默认缓存工具
     * @param redisConnectionFactory
     * @return
     */
    /*@Bean
    //springboot 1.xx
    public CacheManager cacheManager(RedisTemplate redisTemplate) {
        RedisCacheManager rcm = new RedisCacheManager(redisTemplate);
        return rcm;
    }*/
    @Bean
    public CacheManager cacheManager(RedisConnectionFactory redisConnectionFactory) {
        RedisCacheConfiguration redisCacheConfiguration = RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofHours(1)); // 设置缓存有效期一小时
        return RedisCacheManager
                .builder(RedisCacheWriter.nonLockingRedisCacheWriter(redisConnectionFactory))
                .cacheDefaults(redisCacheConfiguration).build();
    }

    /**
     * retemplate相关配置
     * @param factory
     * @return
     */
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {

        RedisTemplate<String, Object> template = new RedisTemplate<>();
        // 配置连接工厂
        template.setConnectionFactory(factory);

        //使用Jackson2JsonRedisSerializer来序列化和反序列化redis的value值（默认使用JDK的序列化方式）
        Jackson2JsonRedisSerializer jacksonSeial = new Jackson2JsonRedisSerializer(Object.class);

        ObjectMapper om = new ObjectMapper();
        // 指定要序列化的域，field,get和set,以及修饰符范围，ANY是都有包括private和public
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        // 指定序列化输入的类型，类必须是非final修饰的，final修饰的类，比如String,Integer等会跑出异常
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jacksonSeial.setObjectMapper(om);

        // 值采用json序列化
        template.setValueSerializer(jacksonSeial);
        //使用StringRedisSerializer来序列化和反序列化redis的key值
        template.setKeySerializer(new StringRedisSerializer());

        // 设置hash key 和value序列化模式
        template.setHashKeySerializer(new StringRedisSerializer());
        template.setHashValueSerializer(jacksonSeial);
        template.afterPropertiesSet();

        return template;
    }

    /**
     * 对hash类型的数据操作
     *
     * @param redisTemplate
     * @return
     */
    @Bean
    public HashOperations<String, String, Object> hashOperations(RedisTemplate<String, Object> redisTemplate) {
        return redisTemplate.opsForHash();
    }

    /**
     * 对redis字符串类型数据操作
     *
     * @param redisTemplate
     * @return
     */
    @Bean
    public ValueOperations<String, Object> valueOperations(RedisTemplate<String, Object> redisTemplate) {
        return redisTemplate.opsForValue();
    }

    /**
     * 对链表类型的数据操作
     *
     * @param redisTemplate
     * @return
     */
    @Bean
    public ListOperations<String, Object> listOperations(RedisTemplate<String, Object> redisTemplate) {
        return redisTemplate.opsForList();
    }

    /**
     * 对无序集合类型的数据操作
     *
     * @param redisTemplate
     * @return
     */
    @Bean
    public SetOperations<String, Object> setOperations(RedisTemplate<String, Object> redisTemplate) {
        return redisTemplate.opsForSet();
    }

    /**
     * 对有序集合类型的数据操作
     *
     * @param redisTemplate
     * @return
     */
    @Bean
    public ZSetOperations<String, Object> zSetOperations(RedisTemplate<String, Object> redisTemplate) {
        return redisTemplate.opsForZSet();
    }
}
```

### 使用RedisTemplate配合ValueOperations<Object,Object>进行对缓存的增删改查

```
首先反射一个RedisTemplate对象
@Autowired
RedisTemplate redisTemplate;

然后下面使用redisTemplate来构造ValueOperations对象
ValueOperations<String,User> operations=redisTemplate.opsForValue();

ValueOperations是对redis简单的K-V操作
SetOperations：set类型数据操作
ZSetOperations：zset类型数据操作
HashOperations：针对map类型的数据操作
ListOperations：针对list类型的数据操作

如果是获取方法的话先判断其在缓存中存在，如果存在那么就从缓存中获取数据如果不存在就去数据库中查找出来并存在缓存中
boolean redisTemplate.hasKey(key);//判断redis中是否有该key

Object operations.get(key);//通过operations去redis缓存中获取key对应的value

void operations.set(key,object,timeout,unit)//最后两个是指超时长度和时间单位例如：operations.set(key, user, 5, TimeUnit.HOURS);即有效期5小时

```

### Operations方法解析

```
void set(K key,V value);//如果redis原来有这个key那么会覆盖原来的value，此时默认是没有有效时间限制

void set(K key,V value,long timeout,TimeUnit unit);//相比于上方的set这里的set新增设置时间timeout即为单位数量后面的unit代表时间单位

V get(K key);//获取redis内key对应的value

V getAndSet(K key,V value)更新key对应的value并且返回原来的key对应的旧值

Integer append(K key,String value);//如果key已经存在并且是一个字符串则该命令将value加到原字符串末尾如果key不存在那么相当于set方法

del,set,get,getset会将key对应储存的值换成新的会重置过期时间
```

### ListOperations方法解析

```
long size(K key);//返回储存在key对应列表的长度

long leftPush(K key,V value);//将所有指定的值插入存储在键的对应list头部如果键不存在就会自动创建一个

long leftPushAll(K key,V[] values);//将对应数组全部插入到列表头中，比如数组为1 2 3 那么完成后顺序为3 2 1

long rightPush(K key,V value);

long rightPushAll(K key, V[] values);

void set(K key,long index,V value);在列表中的index的位置设置value值，如果原位置有值那么会替换原来的值

V index(K key,long index);//根据下标获取列表中的值，下标是从0开始的

V leftPop(K key);//弹出key对应的list最左边的元素，弹出之后该值在列表中会被删除

V rightPop(K key);

List<V> range(key,left,right);//获取一定返回内的值左右都是闭集合并非左闭右开
```

### SetOperations方法解析

```
long add(K key,V[] values);//无序集合中添加元素返回添加个数

long remove(K key,Object[] values);//移除集合中一个或多个成员

V pop(K key);//移除并返回集合中的一个随机元素

Boolean move(K key,V value,K key1);//将value元素从key集合移动到key1集合

long size(K key);//返回该key对应的set集合的长度

Set<V> members(K key);//返回key对应的set中的所有成员
```

## Redis缓存说明

### 为什么要使用redis

在项目中使用redi主要是从两个角度去考虑性能和并发，当然redis还具备可以做分布式锁等其他功能，但是如果只是为了分布式锁这些其他功能完全可以使用zookpeer代替并不是非要使用redis，所以该问题主要从性能和并发两个角度去答

```
性能：在遇到需要执行耗时特别久并且结果不频繁变动得SQL就特别适合将运行结果放入缓存这样后面去缓存中读取就能使得请求快速响应

并发：在大并发得情况下所有得请求直接访问数据库，数据库会出现连接异常这个时候需要使用redis做一个缓冲操作让请求先访问到redis而不是直接访问数据库
```

### redis缺点

```
缓存和数据库双写一致性，即缓存中得数据和数据库中得数据要保持一致
 
缓存雪崩问题：当缓存服务器重启或者大量缓存集中在某一个时间段失效，这样在失效后也会给数据库带来很大的压力，解决方案（加锁或者队列的方式来保证不会有大量的线程对缓存数据库一次性进行读写，或者在设置失效时间的基础上增加一个随机值这样集体失效的时间就很难发生）

缓存穿透和缓存击穿问题：
穿透：
比如服务段发送一个请求在缓存中不存在在数据库中也不存在那么就会穿透两层即每次都要去访问数据库容易把数据库打崩，比如我请求一个主键为-1得请求此时redis中和数据库中都是不存在得很容易把数据库打崩掉，解决方案为在redis层和数据库层添加一个布隆过滤器，看请求得id数据库中是否有
//布隆过滤器,可以存储千万级
BloomFilter<Integer>filter=BloomFilter.create(Funnels.intergerFunnel(),1500,0.01);
filter.put(object);//将元素添加进布隆过滤器
boolean b=filter.mightContain(object);//判断该元素是否在布隆过滤器中

击穿：key快到过期时间了此时有大量请求到来再redis中查不到会全部去数据库中查找即击穿了redis层去数据库层请求容易造成数据库层的崩溃，解决方案（使用互斥锁）

缓存得并发竞争问题：
```

### 单线程得redis为什么这么快

```
纯内存操作
单线程操作避免了频繁得上下文切换
采用了非阻塞I/O多路复用机制：我们得redis-client在操作的时候会产生具有不同事件类型得socket,在服务段有一段I/O多路复用程序将其置入队列中然后文件事件分派其依次去队列中取，转发到不同得时间处理器中
```

### redis五种数据类型

```
string
list
set
zset
hash
```

### redis得过期策略以及内存淘汰机制

```
redis采用得是定期删除+惰性删除策略
为什么不用定时删除策略？定时删除用一个定时器来负责监视key过期则自动删除虽然内存及时释放但是十分消耗CPU资源在大并发请求下CPU要将时间应用在处理请求而不是删除key,因此没有采用这一策略

那么定期删除+惰性删除是若何工作得呢？
定期删除：redis默认每隔100ms检查是否有过期得key,有过期得key则删除，需要说明得是redis不是每隔100ms将所有得key检查一次而是随机抽取进行检查，因此如果只采用定期删除策略会导致很多Key到时间没有删除
于是惰性删除派上用场也就是说你再获取某个key得时候redis会检查一下这个key如果设置了过期时间那么是否过期了如果过期了此时就会删除

如果定期删除没删除key然后你也没即时去请求key,也就是说惰性删除也没生效这样redis得内存会越来越高那么就应该采取内存淘汰机制（即内存不足以容纳写入数据时采取得操作）
在redis.conf中有一行配置#maxmemory-policy volatile-lru
内存淘汰策略有：
noeviction:当内存不足以容纳新写入数据时,新写入操作会报错(基本不用)
allkeys-lru:当内存不足以容纳新写入数据时，在键空间中移除最近少使用得key(目前推荐使用这种模式)
allkeys-random:当内存不足以容纳新写入数据时,在键空间中随机移除某个key(基本没有人用)
volatitle-lru：当内存不足以容纳新写入数据时在设置了过期时间得键空间中移除最近少使用得key这种是把redis即当缓存又当持久化存储得时候采用（不推荐）
volatile-random:当内存不足以容纳新写入数据时,在设置了过期时间得键空间中随机移除某个key(不推荐)
volatile-ttl:当内存不足以容纳新写入数据时在设置了过期时间得键空间中,过期时间最早的优先移除(不推荐)
```

## SpringBoot整合Jedis缓存 

Jedis相比于redisTemplate来说快很多，推荐使用Jedis这种方式

Jedis的构造底层是用Socket技术给Redis打开一个通道，但是平凡的Jedis创建和关闭会消耗大量的时间（Socket的创建很花时间），所以采用连接池的方式，每次使用完Jedis将连接归还给连接池即可

### 依赖

```
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>3.2.0</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>3.8.1</version>
            <scope>test</scope>
        </dependency>
```

### Jedis设置连接池并从中拿Jedis对象

```
      JedisPoolConfig jedisPoolConfig=new JedisPoolConfig();
      jedisPoolConfig.setMaxTotal(10);//设置最大链接数
      JedisPool pool=new JedisPool(jedisPoolConfig,"47.94.84.23");//创建连接池
      System.out.println("Jedis连接初始化成功！");
      Jedis jedis=pool.getResource();//jedis从连接池中获取
      jedis.close();//jedis归还连接池，2.9版本后采用这种方式原来的方法被淘汰了
      System.out.println("Jedis已经返回连接池");
      pool.close();
     log.info("Jedis连接池已经关闭");
```



### Jedis基本使用

```
Jedis jedis=new Jedis("地址");
//设置redis String字符串数据
jedis.set("runoobkey","www.baidu.com");
//在jedis key后面追加新的值
jedis.append("runoobkey","new");
//将对象用jedis存储进缓存中,将对象转换成JSON进行储存
jedis.set("user",JSON.toJSONString(user));
取出后是一个String的JSON串想要转换成对象的话使用JSONObject.parseObject(String,对象,class)


//jedis设置key的时效
jedis.setex(key,seconed,value);
//jedis删除一个键
jedis.del(key);


//设置redis List数据，lpush即为leftpushAll
jedis.lpush("s-list","Runoob");
//设置redis List数据,rpush即为rightpushAll
jedis.rpush("s-list","Runner","Faaster")
//取出key对应的长度
jedis.llen(key);
//将redis里的list数据按一定范围输出,如果范围最后一个值为-1表示全部取出，最后一个值的长度可以超出实际长度但是只会得到原有长度的数据
List<String>list=jedis.lrange("s-list",0,2);
//获取redis里的所有key
Set<String>keys=jedis.keys("*");
//取出redis List数据最左边的一个数据并将其在List中删除
jedis.lpop(key);
//取出redis List数据最右边的一个数据并将其在List中删除
jedis.rpop(key);


//设置redis Map数据,这里的Map只能为Map<String,String>
jedis.hmset(key,map);
//设置redis Map数据，这里的Map只能为Map<String,Double>
jedis.zadd(key,map);
//按条件统计jedis里value在double1和double2之间的个数
jedis.zcount(key,double1,double2);

//判断redis中是否含有这个key
jedis.exists(key);
//返回redis Map类型里面键值对个数
jedis.hlen(key);
//返回redis Map类型里面的所有键值对的key
jedis.hkeys(key);
//返回redis Map类型里面所有键值对的value
jedis.hvals(key);
//返回redis Map类型里面所有键值对
jedis.hgetAll(key);
```

#### JSON依赖

```
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.46</version>
        </dependency>
```



## Redis总结

### redis单线程为什么执行速度那么快

```
纯内存操作，避免大量访问数据库，redis将数据存储再内存里面读写数据的时候都不会收到硬盘I/O速度的限制

单线程操作，避免了不必要的上下文切换和竞争，也不存在多线程切换而消耗CPU也不用考虑锁的问题

采用了非阻塞I/O多路复用机制，多路加入一个队列中，再从队列中取出
```

### redis数据结构

```
String:优点不会出现字符串更变造成的内存溢出问题，获取字符串长度时间复杂度为O(1),空间预分配，惰性空间释放free字段会默认留够一定的空间防止多次重分配内存
List:实现为一个双向链表可以支持双向查找和遍历
Hash：在数据+链表的基础上进行了一些rehash优化 Redis的Hash采用链地址法来处理冲突没有使用红黑树优化
ZSet:内部使用HashMap和SkpiList来保证数据的存储和有序
Set：内部实现是一个value为null的HashMap,实际就是通过计算hash的方式来快速排重，这也是set能提供判断一个成员是否在集合内的原因
```

### redis事务

```
Multi开启事务
Exec执行事务块内命令
Discard取消事务
Watch监视一个或多个key,如果事务执行前key改变则事务打断
```

### redis事务的实现特征

```
所有命令都将会被串行化的顺序执行，事务执行期间Redis不会再为其他客户端的请求提供任何服务，从而保证了事务中所有命令的原子执行

Redis事务中如果有某一条命令执行失败其后的命令仍然会被继续执行

如果在写入的过程中出现系统崩溃如断电导致的宕机那么此时也许只有部分数据被写入到子盘中而另外一部分数据却已经丢失
```

### Redis的同步机制

```
全量拷贝，slave第一次启动时连接Master发送PSYNC命令

master会执行bgsave命令来在生成rdb文件，期间所有命令将被写入缓冲区

master bgsave执行完毕,向slave发送rdb文件，slave收到rdb文件丢弃所有旧数据开始载入rdb文件，同步完成之后slave执行从master缓冲区发送过来的所有写命令
```

### Redis集群模式性能优化

```
Master最好不要做任何从持久化工作如RDB内存快照和AOF日志文件

如果数据比较重要某个Slave开启AOF备份数据，策略设置为每秒同步一次

为了主从复制的速度和连接的稳定性Master和Slave最好在同一个局域网内

```

### Redis最适合的场景

```
会话缓存

排行榜/计数器

发布/订阅
```

### 缓存淘汰机制

```
先进先出算法(FIFO)
最近使用最少(LFU)
最长时间未被使用(LRU)
当存在热点数据的时候LRU的效率很好，但偶发性的周期性的批量操作会导致LRU的命中率急剧下降
```

### Redis删除过期的key的策略

```
定期删除+惰性删除
定期删除和定时删除的不同在于定期删除每隔一段时间随机抽几个检查而定时删除隔一段时间全部检查一遍太过于浪费CPU
惰性删除:当你使用一个key时首先检查它是否过期如果过期就删除
如果一个key定期删除一直没有检查它也没有使用它进行惰性删除但是已经过期了就会占用内存所以当内存满了的时候采取缓存淘汰机制
```

### 缓存雪崩

```
同一时间大量缓存失效

处理方法：
在设置时间时增加一个随机值使得同一时间出现大量缓存失效的情况少发生
双层缓存策略
定时更新策略
```

### 缓存击穿

```
频繁请求查询系统中不存在的数据导致的

处理方法：
查询结果为null的时候仍然缓存这个null的结果设置一个较短的过期时间
布隆过滤器
```

#### 布隆过滤器

本质上布隆过滤器是一种数据结构比较巧妙得概率型数据结构，特别是高效得插入和查询可以来告诉你某样东西一定不存在或者可能存在，但是他的结果是概率型得而不是确切得

布隆过滤器为什么不用HashMap,HashMap确实可以做到利用将值映射到HashMap得key然后在O(1)得时间内返回结果但是由于负载因子得存在导致内存空间通常不能被用满，一旦数据量上去之后内存消耗会非常大

**布隆过滤器得数据结构**

布隆过滤器是一个bit向量或者说bit数组：

![image-20200616215306983](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200616215306983.png)

如果我们要映射一个值到布隆过滤器中我们需要使用多个不同得哈希函数生成多个哈希值并对相应位置得bit变为1，例如对值"baidu"取得三个不同得hash值1 4 7那么上图转变为:

![image-20200616215521123](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200616215521123.png)

如果别的值得hash得三个值对应得位置已经变成1了那么就覆盖原来得位置变为1

![image-20200616215706479](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200616215706479.png)

此时tencent得4原来已经是1了那么将其覆盖并返回该值可能存在，

那么如果有一个值得三个hash都存在比如上图来说，此hash得三个值为1 3 7

那么也不能说这个值一定存在，因为随着增加的值越来越多被置为1得地方越来越多，就算某个值没有被存储过但是万一三个哈希值都已经被置为1了还是会判断这个值可能存在



另外hash得个数也需要权衡，个数越多效率越低，但是如果太少得话那我们得误报率会变高，这里贴一个公式

![image-20200616222749478](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200616222749478.png)

k为hash的个数，m为布隆过滤器的长度,n为插入元素的个数p为 误报率

### redis阻塞原因

```
数据结构使用不合理
CPU饱和
持久化阻塞
```

### redis热点数据出现造成集群访问量倾斜解决办法

```
使用本地缓存
利用分片算法特性将hot key加上前缀或者后缀把一个hotkey的数量变成redis实例个数N的倍数M
```

### redis分布式锁

选择使用zookeeper

### Redis如何做持久化

```
bgsave做镜像全量持久化，耗时较长在停机时会导致大量丢失数据，所以配合aof使用，重启之后使用bgsave重新创建内存在使用aof重放近期的操作来完成恢复
aof做增量持久化
```

### 如何使用Redis做异步队列

```
使用list结构做队列rpush生产消息，lpop消费消息，如果lpop没有消息的时候要适当sleep一会儿再重试，除了sleep以外list还有个指令交blpop，在没有消息的时候他会阻塞直到有消息进来
```

### 能不能生产一次消费多次呢

```
可以的，只要使用redis的订阅发布模式即pub/sub主题订阅者模式可以实现1:N的消息队列，缺点是在消费者下线的情况下生产的消息会丢失最好使用rabbitmq来解决
```

### 为什么redis zset结构使用跳跃链表(SkipList)而不用红黑树呢

```
SkpiList实现更加简单复杂度和红黑树相同，在并发环境下红黑树在插入和删除的时候需要rebalance,性能不如SkipList
```

