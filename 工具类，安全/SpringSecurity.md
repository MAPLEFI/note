# SpringSecurity

## 依赖

```
<!-- spring security依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
导入依赖之后什么都不需要做，此时所有的请求都会弹出一个需要授权的验证框，Spring security默认用户名为user 密码在启动的时候会在控制台生成
```

## 如果什么都不做全部用默认的配置那么不管什么都会进行限制，如果想要关闭全局验证功能：

```
在启动类上的@SpringBootApplication后面写上(exclude=SecurityAutoConfiguration.class)即可关闭全局验证功能
```

## 由于一直使用默认的用户名user和随机生成的密码实在是不方便，所以我们可以自己配置：

```
在application里面配置：
sprin.security.user.name=
spring.security.user.password=
这种也不推荐，推荐和shiro一样使用数据库的方式来进行管理
```

##  创建一个配置类

```
创建一个类继承WebSecurityConfigurerAdapter并覆写里面的configure方法并创建用户，并且在类头添加一个@EnableSecurity和@Configuration注解
```

### config方法的覆写：

```
利用参数的AuthenticationManagerBuilder auth来增添对象和密码以及角色：
protected void configure(uthenticationManagerBuilder auth) throws Exception{
  /*
  基于内存的方式构建两个用户账号
  */
  auth.inMemoryAuthentication()
  .passwordEncoder(密码的加密方式在Security里面一般使用new BCryptPasswordEncoder())
  .withUser()
  .password()
  .roles();//其中roles里面可以不写角色名，user password,roles里面的参数均为String,password里面要写按加密方式弄好的结果，加密方式不可为空
}






密码加密的另一种方式：
通过@Bean注入指定PasswordEncoding
@Configuration
@EnableWebSecurity//启动SrpingSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
/**
 * 通过覆写configure方法进行创建用户
 */

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        /**
         * 基于内存的方式构建两个用户账号
         * admin 123456
         * user 123465
         */
        auth.inMemoryAuthentication().
                withUser("admin").
                password(passwordEncoder().encode("123456")).
                roles();
        auth.inMemoryAuthentication().
                withUser("user").
                password(passwordEncoder().encode("123456")).
                roles();
    }
    @Bean//注入PasswordEncoder,注入后就不需要再configure里面的加用户是再写一次加密方式了，可以直接对password加密
    public PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }
}
```

### 角色授权

```
给不同的用户设置不同的角色，不同的角色允许的方法是不同的

开启方法级别安全控制
再@Configuration注解类上再添加
@EnableGlobalMethodSecurity(prePostEnable=true)//当他为true时会拦截注解了@PreAuthorize的方法

配置方法级别的权限控制：
使用注解@PreAuthorize(“hasAnyRole('admin','normal')”)即当拥有admin或者normal角色时可以访问该方法
```

