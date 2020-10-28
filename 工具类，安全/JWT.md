# JWT

## 什么是JWT

```
是为了在网络应用环境间传递声明而执行的一种基于JSON的开放标准（(RFC 7519).该token被设计为紧凑且安全的，**特别适用于分布式站点的单点登录（SSO）场景**。JWT的声明一般被用来在身份提供者和服务提供者间传递**被认证的用户身份信息**，以便于从资源服务器获取资源，也可以增加一些额外的其它业务逻辑所必须的声明信息，**该token也可直接被用于认证，也可被加密**
```

## JWT使用的token验证和传统的session验证的区别

### 传统的session验证

```
我们知道，http协议本身是一种无状态的协议，而这就意味着如果用户向我们的应用提供了用户名和密码来进行用户认证，那么下一次请求时，用户还要再一次进行用户认证才行，因为根据http协议，我们并不能知道是哪个用户发出的请求，所以为了让我们的应用能识别是哪个用户发出的请求，我们只能在服务器存储一份用户登录的信息，这份登录信息会在响应时传递给浏览器，告诉其保存为cookie,以便下次请求时发送给我们的应用，这样我们的应用就能识别请求来自哪个用户了,这就是传统的基于session认证
```

### 基于session验证所暴露的问题

```
 每个用户经过我们的应用认证之后，我们的应用都要在服务端做一次记录，以方便用户下次请求的鉴别，通常而言session都是保存在内存中，而随着认证用户的增多，服务端的开销会明显增大
 
 扩展性: 用户认证之后，服务端做认证记录，如果认证的记录被保存在内存中的话，这意味着用户下次请求还必须要请求在这台服务器上,这样才能拿到授权的资源，这样在分布式的应用上，相应的限制了负载均衡器的能力。这也意味着限制了应用的扩展能力。
```



## 总结传统seesion验证的缺点和JWT的优点

```
session验证缺点：每次需要把信息保存到cookie中如果用户量一大就会导致内存消耗非常大，而且session验证只能保存在一台服务器上换句话说如果换一台服务器之后将不能通过验证

jwt优点：不用存在cookie中也不寄托于某一台服务器非常适合于分布式的项目可以减轻开销和负载均衡的压力，jwt安全紧凑从服务器中获取也可以指定多样化的声明满足各类业务并且关闭crsf攻击相对更加安全
```



### 基于token的鉴权机制

```
基于token的鉴权机制类似于http协议也是无状态的，**它不需要在服务端去保留用户的认证信息或者会话信息**。这就意味着基于token认证机制的应用不需要去考虑用户在哪一台服务器登录了，这就为应用的扩展提供了便利
流程为：
用户使用用户名密码来请求服务器
服务器进行验证用户的信息
服务器通过验证发送给用户一个token
客户端存储token，并在每次请求时附送上这个token值
服务端验证token值，并返回数据
```

WT是由三段信息构成的，将这三段信息文本用`.`链接一起就构成了Jwt字符串。就像这样:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ

第一部分我们称它为头部（header),第二部分我们称其为载荷（payload, 类似于飞机上承载的物品)，第三部分是签证（signature).

header中声明加密的算法通常使用HMAC SHA256
{"alg":"HS512"}然后将头部进行base64加密

payloada中用于存放有效信息的地方
iss: jwt签发者
sub: jwt所面向的用户
aud: 接收jwt的一方
exp: jwt的过期时间，这个过期时间必须要大于签发时间
nbf: 定义在什么时间之前，该jwt都是不可用的.
iat: jwt的签发时间
jti: jwt的唯一身份标识，主要用来作为一次性token,从而回避重放攻击
payload用来承载要传递的数据，它的json结构实际上是对JWT要传递的数据的一组声明，这些声明被JWT标准称为claims，
它的一个“属性值对”其实就是一个claim(要求)，每一个claim的都代表特定的含义和作用
{"sub":"admin","created":1489079981393,"exp":1489684781}
然后将其base64加密得到Jwt的第二部分
signature为以header和payload生成的签名，一旦header和payload被篡改验证失败，需要base64机密后的header和payload,然后再通过header中声明的加密方式进行加盐secret组合加密然后就构成了jwt的第三部分
将 这三部分连成一个完整的字符串构成了最终的jwt
```

![image-20200518115251952](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200518115251952.png)

## 整合SpringSecurity及JWT

### 依赖

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<!--Hutool Java工具包-->
<dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-all</artifactId>
    <version>4.5.7</version>
</dependency>
<!--JWT(Json Web Token)登录支持-->
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.9.0</version>
</dependency>
```

### 添加JWT token的工具类

根据payload的claims生成jwt的token

```
    private String generateToken(Map<String, Object> claims) {
        return Jwts.builder()
                .setClaims(claims)
                .setExpiration(generateExpirationDate())//设置过期时间
                .signWith(SignatureAlgorithm.HS512, secret)//设置加密算法和密匙
                .compact();
    }
```

从token中获取jwt中的payload的claims信息

```
    private Claims getClaimsFromToken(String token) {
        Claims claims = null;
        try {
            claims = Jwts.parser()
                    .setSigningKey(secret)//设置解密的密匙
                    .parseClaimsJws(token)//需要解密的token
                    .getBody();
        } catch (Exception e) {
            LOGGER.info("JWT格式验证失败:{}",token);
        }
        return claims;
    }
```

生成token的过期时间

```
private Date generateExpirationDate() {
    return new Date(System.currentTimeMillis() + expiration * 1000);
}//还是推荐使用Clander来设置Date
```

从token中获取登录用户名

```
    public String getUserNameFromToken(String token) {
        String username;
        try {
            Claims claims = getClaimsFromToken(token);
            username =  claims.getSubject();
        } catch (Exception e) {
            username = null;
        }
        return username;
    }
```

验证token是否还有效

```
    public boolean validateToken(String token, UserDetails userDetails) {
        String username = getUserNameFromToken(token);
        return username.equals(userDetails.getUsername()) && !isTokenExpired(token);//判断用户名和时间是否过期
    }
```

验证token是否已经过期

```
private boolean isTokenExpired(String token) {
    Date expiredDate = getExpiredDateFromToken(token);
    return expiredDate.before(new Date());
}
```

从token中获取过期时间

```
    private Date getExpiredDateFromToken(String token) {
        Claims claims = getClaimsFromToken(token);
        return claims.getExpiration();
    }
```

根据用户信息生成token

```
public String generateToken(UserDetails userDetails) {
    Map<String, Object> claims = new HashMap<>();
    claims.put(CLAIM_KEY_USERNAME, userDetails.getUsername());
    claims.put(CLAIM_KEY_CREATED, new Date());
    return generateToken(claims);//利用用户名和当前时间组成claims，再调用利用claims生成token的方法
}
```

验证token是否可以被刷新

```
    public boolean canRefresh(String token) {
        return !isTokenExpired(token);//token没有过期就可刷新
    }
```

刷新token(刷新token的有效时间)

```
    public String refreshToken(String token) {
        Claims claims = getClaimsFromToken(token);
        claims.put(CLAIM_KEY_CREATED, new Date());
        return generateToken(claims);
    }
```

完整的JWTTokenUtil：

```
@Component
public class JwtTokenUtil {
    private static final Logger LOGGER = LoggerFactory.getLogger(JwtTokenUtil.class);
    private static final String CLAIM_KEY_USERNAME = "sub";
    private static final String CLAIM_KEY_CREATED = "created";
    @Value("${jwt.secret}")
    private String secret;
    @Value("${jwt.expiration}")
    private Long expiration;

    /**
     * 根据负责生成JWT的token
     */
    private String generateToken(Map<String, Object> claims) {
        return Jwts.builder()
                .setClaims(claims)
                .setExpiration(generateExpirationDate())
                .signWith(SignatureAlgorithm.HS512, secret)
                .compact();
    }

    /**
     * 从token中获取JWT中的负载
     */
    private Claims getClaimsFromToken(String token) {
        Claims claims = null;
        try {
            claims = Jwts.parser()
                    .setSigningKey(secret)
                    .parseClaimsJws(token)
                    .getBody();
        } catch (Exception e) {
            LOGGER.info("JWT格式验证失败:{}",token);
        }
        return claims;
    }

    /**
     * 生成token的过期时间
     */
    private Date generateExpirationDate() {
        return new Date(System.currentTimeMillis() + expiration * 1000);
    }

    /**
     * 从token中获取登录用户名
     */
    public String getUserNameFromToken(String token) {
        String username;
        try {
            Claims claims = getClaimsFromToken(token);
            username =  claims.getSubject();
        } catch (Exception e) {
            username = null;
        }
        return username;
    }

    /**
     * 验证token是否还有效
     *
     * @param token       客户端传入的token
     * @param userDetails 从数据库中查询出来的用户信息
     */
    public boolean validateToken(String token, UserDetails userDetails) {
        String username = getUserNameFromToken(token);
        return username.equals(userDetails.getUsername()) && !isTokenExpired(token);
    }

    /**
     * 判断token是否已经失效
     */
    private boolean isTokenExpired(String token) {
        Date expiredDate = getExpiredDateFromToken(token);
        return expiredDate.before(new Date());
    }

    /**
     * 从token中获取过期时间
     */
    private Date getExpiredDateFromToken(String token) {
        Claims claims = getClaimsFromToken(token);
        return claims.getExpiration();
    }

    /**
     * 根据用户信息生成token
     */
    public String generateToken(UserDetails userDetails) {
        Map<String, Object> claims = new HashMap<>();
        claims.put(CLAIM_KEY_USERNAME, userDetails.getUsername());
        claims.put(CLAIM_KEY_CREATED, new Date());
        return generateToken(claims);
    }

    /**
     * 判断token是否可以被刷新
     */
    public boolean canRefresh(String token) {
        return !isTokenExpired(token);
    }

    /**
     * 刷新token
     */
    public String refreshToken(String token) {
        Claims claims = getClaimsFromToken(token);
        claims.put(CLAIM_KEY_CREATED, new Date());
        return generateToken(claims);
    }
}
```

## 总结整合JWT

这里采用了jwt里面的claim关键信息只加了sub即用户名，**核心通过实现UserDetailsService的方法来获取其UserDetails对象的用户名密码以及角色**，在每次的请求都访问一次JwtAuthenticationTokenFilter去解开token的claim里面的sub获取和用户名然后通过UserDetailsService来判定角色，再使用UsernamePasswordAuthenticationToken将这个userDetail的信息和其角色信息加到request缓存中去实现角色验证