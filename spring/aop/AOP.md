AOP(Aspect Oriented Programming)面向切面编程：基于面向对象编程基础之上新的编程思想；<b>指在程序运行期间将某段代码动态的切入到指定方法的 指定位置</b>进行运行的这种编程方式

我们希望的是业务逻辑；核心功能；日志模块；在核心功能运行期间自己动态的加上，运行的时候日志功能可以加上，可以使用动态代理来将日志代码在目标方法执行前后执行

动态代理问题：

```
1.写起来太难
2.jdk默认的动态代理：如果目标对象没有实现任何接口无法为其创建代理对象
代理对象和被代理对象唯一能产生联系就是实现了同一个接口
3.实现所有都通用的动态代理太过复杂
```

由于Spring动态代理 所以Spring实现了AOP 功能，底层就是动态代理（可以自动的在目标方法执行前后执行），可以利用Spring一句代码都不写的去创建动态代理，实现简单而且没有强制要求目标对象必须实现接口

AOP专业名词 ：

```
切面类：横切关注点，通知方法，横切关注点表示为目标方法需要切入通知方法得地方

连接点：一个方法的每一个关注点都是一个连接点比如：方法开始，方法返回，方法异常，方法结束

切入点：即在连接点选择的需要加入通知方法的连接点
 
切入点表达式：在众多连接点中选出切入点啊
```
如何讲切面类中的这些通知方法动态的在目标方法运行的各个位置切入，即使目标对象没有实现任何接口也能实现动态代理

AOP使用步骤：

```
1.导包
2.写配置
  将目标类和切面类（封装了在目标方法执行前后执行的方法）加入到ioc容器中,@Service
  告诉Spring到底哪个是切面类，加上一个注解@Aspect
  告诉Spring,切面类里面的每一个方法都是何时何地运行,aspect里面有很多注解帮助解决这个问题
  @Before:在目标方法之前运行，加在方法上面
  @After：在目标方法结束之后
  @AfterReturning:在目标方法正常返回之后
  @AfterThrowing在目标方法抛出异常之后运行
  @Around：环绕
  加了之后需要指定是在哪个方法上运行所以需要增添一个注解的参数：execution(访问权限符 返回值类型 方法全名)
  比如：@Before("execution(public int com.atguigu.impl.MYMath.add(int,int))")
  如果想要所有方法都执行：execution(public int com.atguigu.impl.MYMath.*（int,int）)即代表在MYMath类下的所有方法都适用
   
   开启基于注解的AOP功能@EnableAspectJAutoProxy
3.测试
从ioc容器中拿到目标对象才能起作用，new不起作用
```


细节：

```
从ioc容器中去获取一个对象，如果是通过类型来找而不是通过id找一定使用实现类的接口类型作为参数，AOP底层是动态代理，ioc保存的组件是他的代理对象而不是本类类型
当然可以使用id来查找但是一样使用接口类型来接收这个查出来的对象

当然没有接口就是本类型
```


```
切入点表达式：
固定格式：execution(访问权限 返回值类星星 方法全类名（参数表）)

切入点表达式通配符：
*：匹配一个或者多个字符比如execution(public int com.impl.Mymath*r(int,int))
execution(public int com.impl.*.*(int,int)),
或者匹配任意一个参数比如(int,*),这个情况只匹配两个参数
或者匹配一层路径比如com.atguigu.*.Math就代表匹配atguigu和Math之间的一层路劲，*不能表示任意权限其他都可代表

权限只能写public所以权限的位置可以写也可以不写

虽然说*只能代表一层路径，但是如果*写在开头那么也是可以代表多层路径的比如:*(..)代表所有都可，或者*.*(..)代表所有包下所有类都可

..:匹配任意多个参数或者任意多个类型，比如我想匹配任意方法任意类型都可那么，execution(public int com.atguigu.impl.*(..))即代表去impl下的任意参数任意类型都可匹配
匹配任意多层路径execution（public int com.atguigu..Math*.*(..)）表示atguigu和Math之间可以有任意多层路径

..不能写在开头

还有通配符&& || !都可
```


```
通知方法的执行顺序：
正常执行：@Before @After @AfterReturning

异常执行:
@Before @After @AfterThrowing
```

如何在同时方法运行的时候把详细信息拿到：
```
只需要为通知方法的参数列表上写一个参数 JoinPoint,封装了当前目标方法的详细信息 
joinPoint.getArgs();获取到目标方法啊运行时使用的参数以一个数组的方式返回

joinPoint.getSignature()获取到方法签名，可以通过这个方法签名获取到方法名方法修饰符方法返回值类型等等
比如signature.getName();就可以获取到方法名

获取返回值可以在参数中新增一个Object result参数，但是要告诉Spring这个result用来接收方法返回值，
需要在@AfterReturning注解里面新增一个参数returning="result"即可，即@AfterReturning(value="execution()",returning="result")

获取异常信息：在@AfterThrowing里新增参数throwing=""指定哪个参数是接收异常信息的，即@AfterThrowing(value="execution",throwing="exception")
通知方法参数里面的 exception也得是Exception exception才行不然无法接收所有情况的异常，如果范围写小了就是指定Spring可以接收哪些异常，超出这个范围就不会调用对应的通知方法
同理接受返回值一般来说是写Object这样可以接收所有类型的返回值，如果范围些小了那么相当于指定SPring接收哪些返回值，超出这些范围就不会调用对应的通知方法
```
Spring对通知方法一点都不严格就算把通知方法的返回值写成了private而不是public也会照样执行，唯一要求就是通知方法的参数列表一定不能能乱写

通知方法是Spring利用反射调用的，每次调用方法得确认这个方法的参数表的值

参数表上的每一个参数，Spring都得知道是什么

如果需要改动切入点表达式，改动量较大的话我们可以抽取可重用的切入点表达式：

```
1.随便声明一个没有实现的返回类型为void的空方法
2.给方法上标注@Pointcut注解
3.在@Pointcut注解里面写对应的切入点表达式比如：
@Pointcut("execution(public int com.atguigu.*.Math*R。*(..))")public void haha(){};那么下面如果想使用这局切入点表达式就可以直接引用,比如:@After("haha()")即可
```


环绕通知：

```
@Around
是前置通知返回通知异常通知后置通知四合一，
运行顺序和使用环绕通知以外得不同，其顺序为：
try{
    前置通知
    method
    返回通知
}
catch(){
    异常通知
}
finally{
    后置通知
}
其注解上依然需要使用切入点表达式
不同于其他注解他的获得方法参数和其他信息得不是JoinPoint而是ProceedingJoinPoint,他是JoinPoint得一个实现类

步骤为：
pjp.getArgs()获得方法参数用一个Object数组接收
然后pjp.procced(args)将获得得参数放进去即可利用反射调用目标方法，需要抛出异常，他就是method得运行
用Object proceed=pjp.procced(args)来接收方法反射后得返回值
```

```
所以环绕通知得步骤就是
1.加注解@Around
2.参数为ProceedinngJoinPoint
3.通过参数和上面四个通知一样拿到参数列表和创建一个Signature
4.写一个
try{
//@Before所在位置
    Sys("环绕前置");
    proceed=pjp.proceed(args);//利用反射执行目标方法
    //@AfterReturning所在位置
    Sys("环绕返回")
}
catch(Exception e)
{
     //@AfterThrowing所在位置
    Sys("环绕异常")
}
finally{
   //@After所在位置
    Sys("环绕后置")
}
```
环绕通知时优先于普通通知执行，执行顺序：

```
普通前置

{

try{
环绕前置
环绕执行目标方法：目标方法执行
      环绕返回
}
catch{
环绕异常
  }finally{
    环绕后置}
}
普通后置
普通方法返回/普通方法异常通知

如果方法运行时出现异常由于环绕异常已经把异常接收了所以普通方法异常就不会出现了，为了让外界也能看到异常我们可以将异常在环绕异常抛出去

新的顺序：环绕前置--普通前置--目标方法执行--环绕返回/出现异常--环绕后置--普通后置--普通返回/出现异常
```

如果是多切面顺序为：

```
1.如果没有命名Order得话那么多切面是按照阿斯卡码来定义顺序得，按顺序一个一个执行，除了前置优先执行完一个切面再执行下一个切面，环绕优先只在对应切面起着作用详细看图AOP多切面图解

2.可以添加注解Order(int)里面得int指定顺序，int越小越在多切面外层
```









