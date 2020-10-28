# Redis实现订阅发布

Redis通过PUBLISH`SUBSCRIBE等命令实现了订阅与发布模式，分为i订阅/发布到频道和订阅/发布到模式

## 订阅/发布到频道

![image-20200510195859402](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200510195859402.png)

![image-20200510195908937](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200510195908937.png)

### 订阅频道

**每个Redis服务器进程**都维持着一个表示服务器状态的redis.h/redisServer结构，

**结构内有一个pubsub_channels属性，其为一个字典保存订阅频道的信息**

![image-20200510200030651](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200510200030651.png)

其中字典的键为正在被订阅的频道，字典的值是一个链表保存了所有订阅了该频道的客户端![image-20200510200115195](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200510200115195.png)

当**客户端**调用SUBSCRIBE命令时，程序九江客户端和要订阅的频道在pubsub_channels字典中关联起来

![image-20200510200232479](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200510200232479.png)

通过pubsub_channels字典程序只需要检查某个频道是否为字典的键，就可以通过这个值来取出所有订阅该频道的客户端的信息即可检查某个频道是否为字典的键

### 发送信息到频道

了解了pubsub_channels字典的结构之后解释publish命令的实验就和能简单了

当调用publish channel message命令时 程序根据channel定位到字典的键，然后将信息发送给字典值链表中的所有客户端

![image-20200510200753900](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200510200753900.png)

### 退订频道

使用UNSUBSCRIBE命令可以退订指定的频道，他从pubsub_channels字典中定位到具体频道然后删除关于当前客户端的信息 ，这样退订之后信息就不会从这个频道i发送到客户端

## 订阅到模式和信息发送

当使用PUBLISH命令发送信息到某个频道时不仅所有订阅该 频道的客户端会受到消息，如果有某个/某些模式和这个频道匹配的话，那么所有订阅了这个/这些模式的刻画u的那也同样会收到信息

![image-20200510201833279](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200510201833279.png)

![image-20200510201854372](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200510201854372.png)

### 订阅模式

结构redisServer中有一个属性pubsub_patterns属性是一个链表，链表中保存着所有和模式相关的信息

![image-20200510202051384](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200510202051384.png)

链表中每个节点都包含一个redis.h/pubsubPattern结构

![image-20200510202357198](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200510202357198.png)

![image-20200510203407450](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200510203407450.png)