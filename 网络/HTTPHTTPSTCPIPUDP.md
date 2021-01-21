# HTTP/HTTPS/TCP/IP/UDP

## TCP/IP族

![image-20210115163808429](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210115163808429.png)

## 一个HTTP请求的分层解析流程

1.解析域名，交互只认IP地址，先查看浏览器中是否有对应域名的DNS缓存，如果有就直接拿到IP，如果没有就去找本地ｈｏｓｔ配置里面是否有配置这个域名对应的IP，**如果也没有就会发起DNS请求**，获取对应的IP,**在应用层就已经获得了IP了**

2.**发起DNS请求之后应用层**会构造一个DNS请求报文然后应**用层会调用传输层UDP相关协议**，即调用传输层API之后会在DNS报文上加上一个UDP请求头，**然后传输层会交给网络层**，网络层会在UDP的基础上再加一个IP的请求头，**然后网络层会把其交给数据链路层**，数据链路层会进行一个二层的寻址，会把自己的Mark加上去**再通过物理层传到路由器**，路由器由三层构成分别是网络层、数据链路层、物理层，**传到路由器之后从物理层开始一层一层解析到网络层**之后会传到运营商的网络接口最后会找到对应域名的IP地址

**总的来说要获得一个域名的IP那么是从HTTP协议发起请求加上TCP协议参数再加上IP协议的参数和Mark投发送到路由器中然后获得对应的IP地址**

![image-20210115165327255](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210115165327255.png)

## HTTP 协议

**HTTP协议是超文本传输协议,其传输是由UDP(传输层)负责的**,**是一种无状态的,以请求/应答方式运行的协议,**它使用可扩展的语义和自描述消息格式,与基于网络的超文本信息系统灵活的互动

### HTTP报文格式

![image-20210115171250488](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210115171250488.png)

HTTP协议的请求报文和响应报文的结构就基本相同由三大部分组成:

- 起始行:描述请求或响应的基本信息
- 头部字段集合:使用key-value形式更详细地说明报文
- 消息正文:实际传输地数据,它不一定是纯文本,可以是图片/视频等二进制数据

### 请求行报文格式

![image-20210115171743622](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210115171743622.png)

- 请求方法:如GET/POST/PUT/HEAD
- 请求目标:通常是一个URL,标记了请i去 方法要操作的资源
- 版本号:表示报文使用的HTTP协议版本

### 响应行报文格式

![image-20210115171841339](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210115171841339.png)

- 版本号:表示报文使用的HTTP 协议版本
- 状态码:一个三位数,用代码的形式表示处理的结果比如200是成功500是服务器错误
- 原因:作为数字状态码补充是更详细的解释文字,帮助人理解原因

### HTTP头字段

头部字段是key-value的形式,key和value之间用:分割,比如前后分离式经常要与后端协商传输数据的类型Content-type:application/json.HTTP头字段非常灵活,不仅可以使用标准里的Host/Connection等已有头,也可以任意添加自定义头

头字段注意事项

- 字段名不区分大小写,字段名里面不允许出现空格,可以使用连字符"-",但不能使用下划线"_".字段后面必须紧接着:不饿能有空格,而:后的字段值前可以有多个空格
- 字段的顺序是没有意义的,可以任意排序不影响语义
- 字段原则上不允许重复除非这个字段本身 的语义允许

### HTTP请求完整的过程

![image-20210115173019802](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210115173019802.png)

- 解析IP,先从浏览器缓存中检查是否有该域名对应的IP没有就从本地host文件去找如果还没有就发起DNS请求获取ip
- 网络请求:TCP三次握手,发起HTTP请求,等待http相应,浏览器解析响应报文,渲染页面,tcp四次挥手

## TCP协议

TCP协议是面向链接的，可靠的，基于字节流的传输层通信协议

特点:

- 基于连接的：数据传输之前需要建立连接
- 全双工的:双向传输
- 字节流:不限制数据大小，打包成报文段，保证有序接收，重复报文自动丢弃
- 流量缓冲:解决双方处理能力的不匹配
- 可靠的传输服务:保证可达，丢包时通过重发机制实现可靠性
- 拥塞控制:防止网络出现恶行拥塞

### TCP连接管理

TCP连接:四元组(源地址，源端口，目的地址，目的端口)

TCP报文：

![image-20210117004437103](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210117004437103.png)

TCP三次握手:

![image-20210117005220070](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210117005220070.png)

左边是客户端右边是服务器

首先客户端想要建立连接就发送一个连接报文和随机序列号(x)到服务器SYN,seq=x然后客户端进入SYN-SENT发送状态，服务器收到客户端的连接报文之后应答回客户端发送一个ACK,ack=x+1(表示应答了seq=x的连接请求)和一个连接请求SYN,seq=y并进入等待应带状态,客户端收到来自服务器的ACK,ack=x+1之后进入ESTABLISHED(连接状态)，然后客户端应答服务器的连接请求返回一个ACK,ack=y+1回服务器，服务器收到之后进入到ESTABLISHED(连接状态)

TCP三次握手时操作系统内核情况:

![image-20210117010356930](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210117010356930.png)

在接收到SYN报文时会将其放入SYN队列当中然后立刻返回一个SYN/ACK然后进入等待状态，等待接收到对应的ACK后进入ACCEPT队列中Socket Api就能成功拿到连接即代表进入了ESTABLISHED(建立连接)状态

TCP四次挥手:

![image-20210117235836565](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210117235836565.png)

客户端发送一个FIN报文然后客户端进入等待关闭状态，服务器端收到来自客户端的FIN报文之后会立刻返回一个ACK告诉客户端已经收到了FIN报文，然是需要稍作等候（慢慢处理成能够关闭不带来数据丢失的状态）然后服务器端再发送一个FIN报文给客户端告诉客户端服务器已经准备好释放连接了，客户端接收到来自服务器端的FIN报文之后立刻返回一个ACK给服务器端告诉服务器端已经收到服务器端的FIN，然后需要等待一个报文来回的时间2MSL之后客户端最终进入CLOSE状态

为什么最后要等待2MSL后释放连接:

- 防止报文丢失，导致服务端重复发送FIN
- 防止滞留在网络中的报文对新建立的连接造成数据扰乱

### TCP是一个字节流的协议

**TCP把应用交付的数据仅仅看成是一连串无结构的字节流，TCP并不知道字节流的含义，TCP并不关心应用程序一次将多大的报文发送到TCP的缓存中，而是根据对方给出的窗口值和当前网络拥堵的成都来决定一个报文段应该包含多少个字节，并且拥有报文去重的能力**

### 数据可靠性传输

![image-20210118001647879](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210118001647879.png)

即一个报文发送等待一个ack的接收

重传机制:如果传输过程中数据有丢失

#### 1.ack报文丢失

![image-20210118001748364](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210118001748364.png)

客户端在一定时间的等待之后没有收到服务端的ack就会任务ack丢失了就会进行一个报文的重传直到收到ack

#### 2.请求报文丢失

![image-20210118001849959](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210118001849959.png)

和上面的一样客户端在一定时间收不到ack就会重新发送报文，直到收到ack

#### 滑动窗口协议与累计确认(延时ack)

![image-20210118002220628](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210118002220628.png)

滑动窗口大小通过tcp三次握手和对端协商且受网络状况影响，

它是一次发送一批报文，如上图加入一次性一批发送了5个报文，但是我们只收到了1245的ack其中3的ack丢失了，那么我们只能算从开头连续的最长报文即12，剩下的345下次都要重传

## HTTPS协议

HTTPS协议相对于HTTP的"裸奔"似的协议不同，HTTP随时都能被截取被篡改或者被拦截，但是HTTPS协议是一种安全的协议，他并没有改变HTTP协议本身只是在HTTP和TCP层之间增加了一个SSL层，SSL一般就是加密算法

加密算法 有对称密钥加密算法和非对称密钥加密算法

### 对称密钥加密算法

编码和解码使用相同的密钥的算法比如AES，SHA256等等

### 非对称密钥加密算法

他拥有两个密钥一个叫公钥一个叫私钥，两个密钥是不同的，公钥可以公开给任何人使用，而私钥必须严格保密，服务器秘密保管私钥，在网上任意分发公钥，想要登录网站只需要使用公钥加密就行，但是通常 需要大量的数学运算会比较慢

![image-20210120153726564](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210120153726564.png)