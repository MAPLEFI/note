# SpringIOC/AOP，bean生命周期

## Spring自动装配原理

打开@SpringBootApplication->@EnableAutoConfiguration->@Import(AutoConfigurationImportSelector.class),其中AutoConfigurationImportSelector类中有一个方法名为getCandidateConfigurations里有一个getSpringFactoriesLaderFactoryClass()返回了一个EnableAutoConfiguration.class把对应包中的这个类读进来完成了自动装配功能

但是为什么此处的@Import就导入了，是因为在ConfigurationClassPostProcessor里面会解析@Import

## SpringIOC

IOC:控制反转，传统的JAVASE是通过new来创建一个对象，是程序主动的创建依赖对象，而**IOC是利用反射的原理**将对象创建的权力交给**spring容器**，spring在运行的时候**根据配置文件**来动态的创建对象和维护对象之间的关系，实现了松耦合的思想->**实现方式:配置文件，注解**

### 什么是反射?

**反射就是在运行时才知道要操作的类是什么，并且可以在运行时获取类的完整构造，并调用对应得方法**

### IOC原理

创建类得xml文件

```
<bean id="userService" class="...." />
```

然后创建一个工厂类使用dom4j解析配置文件+反射

实际上IOC就是将类得所有信息放到了一个容器里面，容器是由map构成得，有需要得情况在运行期间会去容器中拿对应类得对象通过反射进行赋值

## SpringAOP

AOP:面向切面编程，Filter(过滤器)也是一种AOP.AOP是一种新的方法论，AOP的主要编程对象是切面，而切面模块化横切关注点，可以举例通过事物说明:**通过配置可以实现业务逻辑和系统服务分离，业务逻辑只关心业务的处理不用去处理其他的事情**

正常的一个方法的切面通常是横切面有前置切面(@Before),后置切面(@AfterReturning),返回切面(@After),异常切面(@AfterThrowing)

通常来说环绕通知的顺序更符合逻辑

```
@Around
try{
前置切面
method
返回切面
}catch(){
   异常切面
}finally{
   后置切面
}
```

例子:

```
参数一般有一个ProceedinngJoinPoint pjp
try{
  Sys("环绕前置");//@Before
  proceed=pjp.proceed(args);//利用反射执行目标方法
  Sys("环绕返回");//@AfterReturning
}catch(Exception e){
Sys("环绕异常");//@AfterThrowing
}finally{
Sys("环绕后置")//@After
}
```

### AOP声明式事务管理有两种常用的方式

- 基于tx和aop名字空间的xml配置文件
- 基于@Transactional注解(实际上是ThreadLoacl的实现原子性)

### AOP思想实现技术

SpringAOP:动态代理，在运行期对业务方法进行增强，不会生成新类

### AOP的底层原理是什么？拦截器的优势有哪些?

SpringAOP的底层都是通过代理来实现的

- jdk动态代理:需要实现某个接口(AOP默认使用jdk动态代理)
- cglib动态代理:目标类的子类(不需要类继承任何接口,CGlib包在Spring core)
- 拦截器是基于Java反射机制实现的，使用代理模式 

## Spring中bean得生命周期

创建一个普通类，生成对应得beanDefinitionMap,然后使用BEan工厂进行一个统一得管理，如果需要实例化就推断构造方法根据beanDefinition生成Object，执行Bean得后置处理器