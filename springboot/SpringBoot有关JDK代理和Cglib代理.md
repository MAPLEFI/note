# SpringBoot有关JDK代理和Cglib代理

Spring AOP使用JDK动态代理或CGLIB为给定目标对象创建代理。（默认使用JDK动态代理）。

       如果被代理的目标对象已经实现了接口，则使用JDK动态代理。如果目标对象未实现任何接口，则会创建CGLIB代理。

如果要强制使用CGLIB代理（例如，代理目标对象的方法不仅仅是其接口实现的方法），您注意以下问题：

final 方法不建议使用，因为它们无法被覆盖。

java动态代理是利用反射机制生成一个实现代理接口的匿名类，在调用具体方法前调用InvokeHandler来处理。

而cglib动态代理是利用asm开源包，对代理对象类的class文件加载进来，通过修改其字节码生成子类来处理。

1、如果目标对象实现了接口，默认情况下会采用JDK的动态代理实现AOP 
2、如果目标对象实现了接口，可以强制使用CGLIB实现AOP 

3、如果目标对象没有实现了接口，必须采用CGLIB库，spring会自动在JDK动态代理和CGLIB之间转换





例如在实现shiro配置的时候需要强势开启cglib代理，因为很多时候我们并不会特意为某个bean去写一个接口，再去实现，这种情况下如果用默认的jdk动态代理，则一旦有多个aop代理，比如既有shiro注解代理，又有事务切片或注解代理，这种时候去注入这个bean便会报错，原因就是这个bean被多次代理的时候，jdk代理是基于接口的，所以最后这个bean的类型变成了代理接口proxy的类型

shiro中配置强制使用cglib代理：

```
    @Bean
    public static DefaultAdvisorAutoProxyCreator getDefaultAdvisorAutoProxyCreator() {
        DefaultAdvisorAutoProxyCreator defaultAdvisorAutoProxyCreator = new DefaultAdvisorAutoProxyCreator();
        // 强制使用cglib，防止重复代理和可能引起代理出错的问题
        // https://zhuanlan.zhihu.com/p/29161098
        defaultAdvisorAutoProxyCreator.setProxyTargetClass(true);
        return defaultAdvisorAutoProxyCreator;
    }
```

